

一个**生成随机树（此树非彼数）的算法**，树的结点编号从1开始，这个算法生成了树的结点个数、树的结点的权值、树的每条边的结点。 
如下面是一棵10结点的二叉树的生成结果： 
```
10
-23 -44 -51 -9 13 51 62 11 -63 19 
6 9
6 4
9 2
9 3
4 7
4 1
2 5
2 8
3 10 
```

---
# 第一次更新

想到一种O(n)复杂度的随机树生成算法。

设想有树结点1、2、3、4、5，生成它们的一个随机排列，如4、1、3、5、2；
那么，如果我们设定每个结点的子节点数量为2，或者设置其他区间（如[1, 3]）。
那么根节点就是4，它的子节点为1、3，以BFS的方式遍历生成子节点，1的子节点为5、2，就可以生成随机树了。

生成随机排列的算法复杂度为O(n)。
对于a[0], a[1], a[2], a[3], a[4]，如何生成随机排列？
获得x = random(0, 3)，（区间[0, 3]的下标），然后交换a[x], a[3]，就生成了一个随机值
接下来，x = random(0, 2),然后交换a[x], a[2]
不断地依次生成，就可以得到一个随机排列，且时间复杂度为O(n)。

所以整个算法的时间复杂度为O(n)。

下面的代码将生成随机排列和BFS遍历的过程融合在了一起，建议分开实现，更加清晰、模块化。 

```cpp
#include <iostream>
#include <fstream>
#include <cstdlib>
#include <queue>
#define random(a, b) rand()%(b-a+1) + a
using namespace std;

void creatData(int n, string filename) {
	fstream file(filename.c_str(), ios::out);
	int *tree = new int [n];
	for (int i = 0; i < n; ++i) {
		tree[i] = i + 1;
	}
	int root = random(0, n - 1);
	swap(tree[root], tree[n - 1]);
	int nxt_idx = n - 2;
	queue<int> Que;
	file << n << endl;
	for (int i = 0; i < n; ++i) {
		file << random(-1024, 1024) << ' ';
	}
	file << endl;
	Que.push(tree[n - 1]);
	while (!Que.empty()) {
		int now = Que.front();
		Que.pop();
		int cnt = random(1, 3);
		for (int i = 0; i < cnt; ++i) {
			int tmp_idx = random(0, nxt_idx); 
			swap(tree[tmp_idx], tree[nxt_idx]);
			file << now << ' ' << tree[nxt_idx] << endl;
			Que.push(tree[nxt_idx]);
			nxt_idx--;
			if (nxt_idx == -1) break;
		}
		if (nxt_idx == -1) break;
	}
}

int main()
{
	creatData(10, "creatTree10.txt");
	creatData(1000, "creatTree1000.txt");
	creatData(10000, "creatTree10000.txt");
	creatData(100000, "creatTree100000.txt");
	creatData(1000000, "creatTree1000000.txt");
}
```

---
---
# 原文

思路是将结点
编号1-n push进vector中，然后随机选一个点为root，并从vector中删除这个点。然后使用基于BFS的方式，从root扩展，随机选在（m, n）区间的子节点个数，同时使用随机方法获取在剩余的vector中获取子节点编号，然后从vector中删除。就这样不断地扩散，当vector的size为0时，说明无子节点可选，从而可以结束算法。

可以设置子节点的随机范围，结点权值的随机范围。

需要注意的是，该算法时间复杂度大概高达O(n^2)，生成100000个数据要40+s，所以要生成更大规模的数据需要较长的时间。

---

代码

```cpp
#include <iostream>
#include <vector>
#include <cstdlib>
#include <queue> 
#include <fstream>
#define random(a, b) rand()%(b-a+1) + a
using namespace std;

vector<int> que;
queue<int> seq;

void deleteOne(int one) {
	vector<int>::iterator it = que.begin();
	while (it != que.end()) {
		if (*it == one) {
			que.erase(it);
			break;
		}
		it++;
	}
}

void creat(int n, string filename) {
	que.clear();
	fstream outfile(filename.c_str(), ios::out);
	outfile << n << endl;
	for (int i = 1; i <= n; ++i) {
		int x = random(-64, 64);
		outfile << x << ' ';
		que.push_back(i);
	}
	outfile << endl;
	int root = random(0, n - 1);
	root = que[root];
//	outfile << "root = " << root << endl;
	deleteOne(root);
	seq.push(root);
	while (que.size()) {
		int now = seq.front();
		seq.pop();
		int ns = random(2, 2); // 将子节点范围取做（2，2）就生成了二叉树 
		int len = que.size();
		for (int i = 0; i < ns; ++i) {
			int x = random(0, len - 1);
			outfile << now << ' ' << que[x] << endl;
			seq.push(que[x]);
			deleteOne(que[x]);
			len--;
			if (len == 0) break;
		}
	}
}

int main()
{
	creat(10, "creatTree10.txt");
	creat(1000, "creatTree1000.txt");
	creat(10000, "creatTree10000.txt");
	creat(100000, "creatTree100000.txt");
}
```