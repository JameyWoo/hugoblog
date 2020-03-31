
用头脑风暴学算法，对于一个问题，我们不只是要解决它，还要去思考有什么好的方法，差的方法去解决，甚至是一些错误的但可以提供思想借鉴的方法。

此问题“数组中出现次数超过一半的数字”是一道非常经典的算法题，我把它放在算法风暴系列第一篇来解析，探讨学习一个算法的过程，从慢到快，从最直观的方法到脑洞大开的方法，由表面深入本质。

----------

[下一篇：算法风暴之二—最小的k个数](https://blog.csdn.net/wjh2622075127/article/details/82830683)

## 问题描述
**给定一个数组，且已知数组中有一个数出现次数超过一半（严格），请求出这个数。**

问题很简单，方法也多样，但什么方法是最好的呢？为什么它最好？各种方法之间有什么优缺点？下面我们一一展开。

----------

## 方法一：给数组排序

这大概是最直观的方法了，最容易想到，也是最多人能够想出来的。如果我们使用快排的话，只需要`O(nlogn)`的时间就可以找到这个数。

那么思考这样一个问题：给数组排序了，然后怎么找这个数呢？有两种方法

1、从小到大遍历已排序数组，同时统计每个数出现的次数（某个数和上一个数不同则计数置为1），如果出现某个计数超过一半，那么正在计数的数就是所求数。

PS：这种方法可行，相比于快排的时间复杂度是可以忽略的，但是我们还有更好的方法，直击本质。

2、对一个已排好序的序列，出现次数超过一半的数必定是中位数。因此，我们只要输出中位数即可。

复杂度分析：
| **时间复杂度** |  **O(nlogn)** |
|--|--|
| **空间复杂度** | **O(n)** |


手写快排代码：

```cpp
#include <algorithm>
#include <iostream>
#include <cstdlib>
#include <ctime>
#define RAND(start, end) start+(int)(end-start+1)*rand()/(RAND_MAX+1);
using namespace std;

const int maxn = 10005;

int Partition(int *data, int length, int start, int end)
{
	if (start == end) return start;
	srand((unsigned)time(NULL));
	int index = RAND(start, end);
	swap(data[index], data[end]);
	int one = start - 1;
	for (index = start; index < end; ++index) {
		if (data[index] < data[end]) {
			++one;
			if (one != index) swap(data[one], data[index]);
		}
	}
	++one;
	swap(data[one], data[end]);
	return one;
}

void QuickSort(int *data, int length, int start, int end)
{
	if (length <= 1) return;
	int mid = Partition(data, length, start, end);
	if (mid > start)
		QuickSort(data, length, start, mid - 1);
	if (mid < end)
		QuickSort(data, length, mid + 1, end);
}

int main()
{
	int n, data[maxn];
	cin >> n;
	for (int i = 0; i < n; ++i) {
		cin >> data[i];
	}
	QuickSort(data, n, 0, n - 1);
	cout << data[n >> 1];
}
```

## 方法二：桶排序计数

如果我们需要统计的数组元素都是正整数呢？那么我们就可以使用桶排序，给他们计数，然后超过数组大小一半的就是结果了。

然而桶排序看上去很简单，“复杂度也不高”，却有很多的限制。

1、首先，数组统计的数需得是可hash的，不然无法将他们在hash数组上计数。但是某些情况，如元素有负值，可进行灵活转化，使其可hash。
2、其次，桶排序方法空间换时间，需要消耗额外的空间，取决于数据的范围。
3、桶排序并非真的那么快。桶排序的时间复杂度并非是普通的`O(n)`, 它的n指的是最大数据范围，如果有这样一组数据`1 100 10000 1000000`，那么桶排序将会有至少1000000次循环，且开出1e6的空间，大大浪费资源。

桶排序方法适合数据范围不大，且数据密度较大的数据。非也，则在此问题上算不上好方法。

代码

```cpp
#include <iostream>
using namespace std;

int main()
{
	int n, max_size = 0, ans = 0;
	cin >> n;
	int *data = new int[n];
	for (int i = 0; i < n; ++i) {
		cin >> data[i];
		max_size = max(max_size, data[i]);
	}
	int *hash = new int[max_size + 1];
	for (int i = 0; i <= max_size; ++i)
		hash[i] = 0;
	for (int i = 0; i < n; ++i)
		hash[data[i]]++;
	for (int i = 0; i <= max_size; ++i)
		if (hash[i] > n >> 1) ans = i;
	cout << ans;
	delete [] data;
	delete [] hash;
}
```

## 方法三：巧用栈

其实我们可以发现，上面的方法一和方法二，固然是这道题的解法之一，但**不是非常具有针对性**。也就是说，那两种方法是**功能过剩的**，而这所谓功能过剩，也正是导致它性能不是最佳的原因。

**那么，我们就应该思考某种算法，只针对这个问题，完全的利用好效率。那么就要从题目出发，找蕴含在问题中的本质规律了。**

其实这个问题的核心就是：**出现次数超过一半**。

我们做这样的思考：

假设k就是我们要求的那个数，那么对这个数组，删掉其中任意两个数所剩下的数组，其对应的k值会改变吗？答案是会的。**但是，如果删掉任意两个不相同的数呢？答案是不会！** 为什么不会？相信聪明的读者瞬间就明白原因，只需进行简单的推导就可以了。

具体的实现过程就是：**每遍历一个数，就将其入栈，同时查询它和栈内前一个元素的大小，如果不同，就同时出栈，否则不变。**

以上，就是用栈的方法解决这个问题的核心。

|时间复杂度 | O(n)|
|--|--|
| **空间复杂度**|**O(n)** |

栈实现代码：

```cpp
#include <iostream>
using namespace std;

int main()
{
	int n;
	cin >> n;
	int *data = new int[n];
	int *stack = new int[n];
	int top = 0;
	cin >> data[0];
	stack[++top] = data[0];
	for (int i = 1; i < n; ++i) {
		cin >> data[i];
		stack[++top] = data[i];
		if (top > 1 && stack[top] != stack[top - 1]) top -= 2;
	}
	cout << stack[top];
	delete [] data;
	delete [] stack;
}
```

## 方法四：找中位数（第n/2大数）

从方法一的分析中我们知道，这个数组的中位数就是答案。方法一是通过给所有的数进行排序找出这个中位数，而我们思考，排序是否有些大材小用？找这个中位数的方法是否可以更简单些？

答案是有的，而且这类问题被称为**找第k个数**。

**思想是快排的思想。时间复杂度为`O(n)`**

利用快排思想，我们可以找出第n/2大的数，同时在第n/2th数左边的数都小于它，右边的数都大于它。这个数就是数组的中位数。

快速排序简称快排，利用分治的思想，在数组中随机选择一个数，然后以这个数为基准，把大于它的数划分到它的右侧，小于它的数划分到它的左侧，并且递归的分别对左右两侧数据进行处理，直到所有的区间都按照这样的规律划分好。

那么在这个问题中，如何利用快排的方法呢？**快排是对每一个区间进行分治处理，而此问题不必，我们只要找到第n/2小的数。每次随机划分得的第m个数，如果m < n/2, 那么对[m + 1, n - 1]这个区间继续递归；如果m > n/2，那么对[0, m - 1]这个区间进行递归；如果刚好有m = n/2，那么函数结束，区间[0, n/2 - 1]的数就是最小的n/2个数。**

此算法的平均时间复杂度为O(n), 快速排序的详细证明可参考“算法导论”。


```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>
#define RAND(start, end) start + (int)(end - start + 1)*(rand()/(RAND_MAX + 1))
using namespace std;

int Partition(int *data, int length, int start, int end)
{
	if (start == end) return start;
	srand((unsigned)time(NULL));
	int index = RAND(start, end);
	swap(data[index], data[end]);
	int one = start - 1;
	for (index = start; index < end; ++index) {
		if (data[index] < data[end]) {
			++one;
			if (one != index) swap(data[one], data[index]);
		}
	}
	++one;
	swap(data[one], data[end]);
	return one;
}

void FindIt(int *data, int length, int start, int end)
{
	int mid = Partition(data, length, start, end);
	if (mid == length >> 1) return;
	else if (mid > length >> 1) FindIt(data, length, start, mid - 1);
	else FindIt(data, length, mid + 1, end);
}

int main()
{
	int n;
	cin >> n;
	int *data = new int[n];
	for (int i = 0; i < n; ++i) {
		cin >> data[i];
	}
	FindIt(data, n, 0, n - 1);
	cout << data[n >> 1];
}
```
[下一篇：算法风暴之二—最小的k个数](https://blog.csdn.net/wjh2622075127/article/details/82830683)