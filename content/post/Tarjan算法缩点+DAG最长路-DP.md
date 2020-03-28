---
title: "Tarjan算法缩点+DAG最长路 DP"
date: 2018-08-13
draft: false

categories: ["算法"]
---


我们按照复杂程度来讨论不同的Tarjan算法变形的差异.

## 第一个问题: Tarjan算法找出一个图里面的全部强连通分量(包括单独的点).

但此时只是有所区分的将所有的点划分为一个个的强连通分量, 尚且没有缩点. 上面这个功能实现起来最简单. 

它的Tarjan函数内部是这样的.

```c++
void tarjan(int u)
{
	dfn[u] = low[u] = ++index;
	stack[++top] = u;
	in_stack[u] = true;
	for (int i = 0; i < G[u].size(); ++i) {
		int v = G[u][i];
		if (!dfn[v]) { // 更新新的点.
			tarjan(v);
			low[u] = min(low[u], low[v]);
		} else if (in_stack[v]) {
			low[u] = min(low[u], low[v]);
		} // 还剩下一种不在栈中但是已经访问过的情况,是其他连通分量的
	}
	if (dfn[u] == low[u]) {
		do {
			cout << stack[top] << ' ';
			in_stack[stack[top]] = false; // 漏写了这一条.
		} while (stack[--top + 1] != u);
		cout << endl;
	}
}
```

## 第二个问题: 对每个强连通分量进行缩点, 使得此图变成一张DAG. 

在Tarjan函数内部他们的主要区别是当`dfn[u] == low[u]`的这一段

```
if (dfn[u] == low[u]) {
		cnt++;
		int now;
		do {
			now = sta.top();
			sta.pop();
			in_stack[now] = false;
			to[now] = cnt;
		} while (now != u);
	}
```
- **增加了一个全局变量`cnt`, 表示当前缩点的编号.**

-  **增加了一个`to`数组, 用来表示原来的点在缩点之后是哪个点.**

所以我们可以有下面这段代码, **`set<int> Now`代表缩点后的新图.**

```
for (int i = 1; i <= n; ++i) {
		for (int j = 0; j < G[i].size(); ++j) {
			int u = to[i], v = to[G[i][j]];
			if (u != v) Now[u].insert(v);
		}
	}
```
**通过`to`数组关联起原图和缩点后图的点, 从而建立新图.**

这样, 通过`dfn[u] == low[u]`处的修改, 以及结合`to`数组建立新图的过程, 就实现了Tarjan算法的缩点.

## 第三个问题: 如何快速获得新图各个结点的入度出度.

上面的`to`数组保留, 看下面这段代码

```
for (int i = 1; i <= n; ++i) {
		for (int j = 0; j < G[i].size(); ++j) {
			if (to[i] != to[G[i][j]]) {
				out[to[i]]++;
				in[to[G[i][j]]]++;
			}
		}
	}
```
一二层循环遍历之前所有的边, 里面一个条件语句, **判断边的两端点是否指向同一个缩点, 如果不是, 那么他们在`to`数组中所指向的新的结点也将作为一条边**. **利用`to`数组可以快速方便的获取新图中的入度和出度, 这样的话要知道入度和出度就无需建立新图.**

##第四个问题: 缩点之后求解DAG最长路

看洛谷上的这道题 [传送门](https://www.luogu.org/problemnew/show/P3387)

此题是以点为权值而非边, 但做法是基本差不多的. 都是DP算法.

DP函数是这样

```
int DP(int u)
{
	if (dp[u]) return dp[u];
	set<int>::iterator is = Now[u].begin();
	while (is != Now[u].end()) {
		dp[u] = max(dp[u], DP(*is));
		is++;
	}
	dp[u] += val_now[u];
	return dp[u];
}
```
**状态转移方程: `dp[i] = max{dp[j] | (i, j) ∈E} + val_now[i]`**
**区别于边为权值的方程:`dp[i] = max{dp[j]+length[i→j] | (i,j)∈E}`**

AC代码

```
#include <algorithm>
#include <iostream>
#include <vector>
#include <stack>
#include <set>
using namespace std;

const int maxn = 10005;
stack<int> sta;
vector<int> G[maxn];
set<int> Now[maxn];
int n, m, index = 0, cnt = 0, ans = 0;
int to[maxn], dfn[maxn] = {}, low[maxn];
int dp[maxn] = {}, val[maxn], val_now[maxn] = {};
bool in_stack[maxn] = {};

void tarjan(int u)
{
	dfn[u] = low[u] = ++index;
	in_stack[u] = true;
	sta.push(u);
	for (int i = 0; i < G[u].size(); ++i) {
		int v = G[u][i];
		if (dfn[v] == 0) {
			tarjan(v);
			low[u] = min(low[u], low[v]);
		} else if (in_stack[v]) low[u] = min(low[u], low[v]);
	}
	if (dfn[u] == low[u]) {
		cnt++;
		int now;
		do {
			now = sta.top();
			sta.pop();
			in_stack[now] = false;
			to[now] = cnt;
		} while (now != u);
	}
}

int DP(int u)
{
	if (dp[u]) return dp[u];
	set<int>::iterator is = Now[u].begin();
	while (is != Now[u].end()) {
		dp[u] = max(dp[u], DP(*is));
		is++;
	}
	dp[u] += val_now[u];
	return dp[u];
}

int main()
{
	cin >> n >> m;
	for (int i = 1; i <= n; ++i)
		cin >> val[i];
	for (int i = 0; i < m; ++i) {
		int u, v;
		cin >> u >> v;
		G[u].push_back(v);
	}
	for (int i = 1; i <= n; ++i)
		if (dfn[i] == 0) tarjan(i);
	for (int i = 1; i <= n; ++i) {
		for (int j = 0; j < G[i].size(); ++j) {
			int u = to[i], v = to[G[i][j]];
			if (u != v) Now[u].insert(v);
		}
	}
	for (int i = 1; i <= n; ++i) {
		int t = to[i];
		val_now[t] += val[i];
	}
	for (int i = 1; i <= cnt; ++i)
		ans = max(ans, dp[i] = DP(i));
	cout << ans;
}
```

**我认为Tarjan算法缩点的核心就是`to`数组. **