

今天参加了一场洛谷网的比赛, 深受打击. 寒假过了这么多天, 一直没有认真学习算法, 以至于现在的水平比两个月前还要低. 

本来就没有多少底子, 又退步了许多, 感慨万分.

在洛谷上看到这么一道题
[最大子段和](https://www.luogu.org/problemnew/show/P1115)

如果数据小的话, **用暴力枚举**很简单就可以做出来了, 时间按复杂度位O(n^3)
可是一道算法题怎么会这么简单呢?

样例的数据是非常大的, 所以用n^3的办法一个样例都过不了

```
#include <iostream>
#include <algorithm> 

using namespace std;

int main()
{
	//std::ios::sync_with_stdio(false);
	long long int n, num[22000], Max = -1e8, sum;
	cin >> n;
	for( int i = 1; i <= n; ++i ) {
		scanf("%lld", &num[i]);
	}
	for( int i = 1; i <= n; ++ i ) {
		for( int j = 1; j <= n - i + 1; ++j ) {
			sum = 0;
			for(int k = j; k < j + i; ++k) {
				sum += num[k];
			}
			//cout << "sum = " << sum << endl;
			Max = max(Max, sum);
			//cout << Max << endl;
		}
	}
	cout << Max;
}
```

稍微优化一下, 使用**前缀和以及差分**, 可以有效地将复杂度降为O(n^2), 可是, 对于这道题来说, 这个时间还是太多了. 只能通过40%的样例

```
#include <iostream>
#include <cstdio>
#include <algorithm>

long long int n, num[220000], qz[220000] = {}, ans = -1e9;

using namespace std;

int main()
{
	//std::ios::sync_with_stdio(false);
	cin >> n;
	for( int i = 1; i <= n; ++ i ) {
		//cin >> num[i];
		scanf("%lld", &num[i]);
	}
	for( int i = 1; i <= n; ++i ) {
		qz[i] = qz[i-1] + num[i];
	}
	for( int i = 1; i <= n; ++i ) {
		for( int j = 1; j <= n-i+1; ++j ) {
			ans = max(ans, qz[j+i-1] - qz[j-1]);
		}
	}
	cout << ans << endl;
}
```

还有一种**分治**的方法做这道题, 时间复杂度为O(nlogn), 有点复杂,我还不会.

最简单的呢, **使用DP的方法, 时间复杂度为O(n)**

这种方法凭我自己现在的水平是想不出来的, 尤其是现在状态如此之差. 看别人解释后写了代码觉得如此巧妙, 于是又惊叹算法之神奇

可怜的是, 即便别人告诉我这道题用DP解最简单, 我也解不出来, 谁叫我DP还没入门呢?

思路描述是这样的:

> a数组是储存输入的数值；c[i]表示的是a数组从头加到i的和；b[i]表示的是从头到i包括i的的最大子段和~（必须包括i！！！）；minn储存最小的前缀和（因为要减去所以要尽量小，详情见下一行） 动态转移方程式：b[i]=c[i]-minn（总和减去前缀和） 最后输出b数组最大的值（题目求最大的子段和）

第一阶段的代码是这样的

```
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

long long int qz[220000] = {}, num[220000], mm[220000] = {}, an[220000], ans = -1e9;

int main()
{
	int n;
	cin >> n;
	for(int i = 1; i <= n; ++i) {
		scanf("%lld", &num[i]);
		qz[i] = num[i] + qz[i - 1];
		mm[i] = min(mm[i - 1], qz[i - 1]);
		ans = max(ans, qz[i] - mm[i]);
	}
	cout << ans;
}
```

时间复杂度确实是小, 但是有一个问题就是空间复杂度太大了, 没必要这样开数组.

看了别人的题解, dalao们用的是滚动数组, 一个上面220000的数组只用两个元素循环滚动即可.

叹服!

妙哉!

```
#include <iostream>
#include <cstdio>
#include <algorithm>

using namespace std;

long long int qz[2] = {}, mm[2] = {}, ans = -1e9, m, n;

int main()
{
	cin >> n;
	for( int i = 1; i <= n; ++i ) {
		scanf("%lld", &m);
		qz[i % 2] = qz[(i + 1) % 2] + m;
		mm[i % 2] = min(mm[(i + 1) % 2], qz[(i + 1) % 2]);
		ans = max(ans, qz[i % 2] - mm[i % 2]);
	}
	cout << ans;
}
```

写这道题还发现了或者说学到了或者说巩固了几个小知识

1) 取消cin的同步用这样的代码

```
std::ios::sync_with_stdio(false);
```
此为巩固

2) 使用scanf输入C++ 中long long 型的数
是这样的

```
long long int n;
scanf("%lld", &n);
```
如果n的类型写为long long Dev会发出警告, 还不知道为什么

百度一下, 又网友说long long 就是long long int , 只是int省略掉了

3) 使用scanf要包含头文件cstdio

以前我是不包含的, 不过这次在洛谷上提交出现了编译错误, 这也算是发现了自己的一个小bug吧

________________2018-7-9补充更新________________

今天程序设计训练测试出了这道题, 时隔多月再次写这道入门DP, 发现实现方法略有不同.

```
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;
const int INF = 0x3f3f3f3f;

int main()
{
	int x;
	vector<int> v;
	while (cin >> x) {
		v.push_back(x);
	}
	int qz = 0, MAX = -INF;
	for (int i = 0; i < v.size(); ++i) {
		if (qz + v[i] <= 0) qz = 0;
		else {
			qz += v[i];
			MAX = max(MAX, qz);
		}
	}
	cout << MAX;
}
```