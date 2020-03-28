---
title: "倍增法求Lca-最近公共祖先"
date: 2018-07-16
draft: false
tags: ["算法"]
categories: ["算法"]
---

## 一. 明确问题

看标题便知道了, 这篇博客力求解决的问题是**求出一棵树的两个结点的最近公共祖先(LCA), 方法是倍增法.**

那么什么是Lca呢? 
> **它是一棵树上两个结点向上移动, 最后交汇的第一个结点, 也就是说这两个结点祖先里离树根最远也是离他们最近的结点.**

什么是倍增法呢? 
> **此问题说的是用倍增法求解lca问题, 那么我们可以推测这种方法还可以解决其他的一些问题(不在当下讨论范围). 在学习的过程中, 我是这么理解的: 它是一种类似于二分的方法, 不过这里不是二分, 而是倍增, 以2倍, 4倍, 等等倍数增长**

一下没有理解倍增法没关系, 看后面的做法, 再结合前面, 前后贯通大概可以理解的七七八八.

## 二. 思路引导

**下面的思路试图把过程模块化, 如果你不知道一个地方如何实现, 还请不要纠结**(比如不要纠结于树的深度怎么求, 假设我们求好了树的深度)

我们找的是任意两个结点的最近公共祖先, 那么我们可以考虑这么两种种情况:

1. 两结点的深度相同.
2.  两结点深度不同.

算法实现来说, **第一种情况是第二种情况的特殊情况, 第二种情况是要转化成第一种情况的**

先不考虑其他, 我们思考这么一个问题: **对于两个深度不同的结点, 把深度更深的那个向其父节点迭代, 直到这个迭代结点和另一个结点深度相同, 那么这两个深度相同的结点的Lca也就是原两个结点的Lca. ** 因此第二种情况转化成第一种情况来求解Lca是可行的.

现在还不知道如何把两个结点迭代到相同深度, 别急, 这里用到的是上面提到的**倍增法**.

那么剩下的问题事就解决第一种情况了, 两个结点深度相同了. 怎么求他们的Lca呢?

这里也是用了倍增法. 思路和上一步的倍增法是一样的, 不同之处有两点
1. 这次是两个结点一起迭代向前的.
2.  这次迭代停止的条件和上次不一样.

OK, 现在无法理解上面的几个过程没关系,**只要知道我们用递增法解决了上述的两个问题.** 具体细节看下面分析.

## 三. 整体框架.

那么过了一遍上面的思路引导之后, 我们可以大概想一想怎么实现整个的问题了.

其实用求Lca这个问题可以分为两块: **预处理 + 查询**, 其中预处理是O(VlogV), 而一次查询是O(logV), V代表结点数量. 所以总**时间复杂度为O(VlogV +ＱlogV)**. Q为查询次数

其步骤是这样的: 
1. 存储一棵树(邻接表法)
2.  获取树各结点的上的深度(dfs或bfs)
3.  获取2次幂祖先的结点, 用parents[maxn][20]数组存储, **倍增法关键**
4.  用倍增法查询Lca

### 步骤一

**用邻接表存储一棵树, 并用from[]数组记录各结点的父节点, 其中没有父节点的就是root.**

parents[u][]数组存储的是u结点的祖先结点. 
如parents[u][0], 是u结点的2⁰祖先结点, 即1祖先, 也即父节点. 在输入过程中可以直接得到.
parents[u][1], 是u结点的2¹祖先结点,即2祖先, 也即父亲的父亲
parents[u][2], 是u结点的2²祖先结点, 即4祖先, 也即(父亲的父亲)的(父亲的父亲), 也就是爷爷的爷爷. 

**理解这个关系很重要, 这也是通过父亲结点获取整个祖先结点的关键.** 现在可以先跳过.
```cpp
void getData()
{
	cin >> n;
	int u, v;
	for (int i = 1; i < n; ++i) {
		cin >> u >> v;
		G[u].push_back(v);
		parents[v][0] = u;
		from[v] = 1;
	}
	for (int i = 1; i <= n; ++i) {
		if (from[i] == -1) root = i;
	}
}
```

### 步骤二

**获取各结点的深度, 可以用DFS或这BFS方法**

```
void getDepth_dfs(int u) // DFS求深度
{
	int len = G[u].size();
	for (int i = 0; i < len; ++i) {
		int v = G[u][i];
		depth[v] = depth[u] + 1;
		getDepth_dfs(v);
	}
}

void getDepth_bfs(int u) // BFS求深度
{
	queue<int> Q;
	Q.push(u);
	while (!Q.empty()) {
		int v = Q.front();
		Q.pop();
		for (int i = 0; i < G[v].size(); ++i) {
			depth[G[v][i]] = depth[v] + 1;
			Q.push(G[v][i]);
		}
	}
}
```
### 步骤三

**求祖先**

在步骤一里面我们讨论了parents数组的意义, 它存的是结点u的2次幂祖先, 从父亲结点开始. 为什么要存2次幂? 这就是**倍增法**的思想了, 我们进行范围缩小不是一步一步的, 那样太暴力了, **所以我们需要某个跨度, 让我们能够先跨越大步, 接近的时候在小步小步跨越, 这样可以大大节省时间.**

**读者可能会疑惑, 先大步, 后小步, 可是我怎么知道什么时候该大步, 什么时候该小步呢? 难道不会不小心跨过头吗? **

其实不会的, 在代码实现上, **这样的跨越有条件约束, 是非常讲究的. 读者不必为此纠结, ** 不过要讲解也是十分费力不讨好的事情, 所以请读者认证推敲后面Lca函数的代码, 认真琢磨为什么是那样跨越, 其中真味自会品出. **最好是自己写几个例子, 模拟跨越的过程, 在结合现实意义去理解**

那么我们回到当前问题. 请看下面这个公式:

> **parents[i][j] = parents[parents[i][j-1]][j-1]**

这是构造此数组的公式. 不难理解, **父亲的父亲就是爷爷, 爷爷的爷爷就是4倍祖先. ** 请读者结合现实意义去理解. 

```
void getParents()
{
	for (int up = 1; (1 << up) <= n; ++up) {
		for (int i = 1; i <= n; ++i) {
			parents[i][up] = parents[parents[i][up - 1]][up - 1];
		}
	}
}
```

### 步骤四

做完了前面**O(VlogV)**的预处理操作, 剩下的就是查询了, 一次查询**O(logV)**

因此, 我们可以敏锐的想到: **Lca算法适合查询次数比较多的情况, 不然, 光是预处理就花了那么多时间了**. 所以说, 查询是我们享受成果的时候了.

```
int Lca(int u, int v)
{
	if (depth[u] < depth[v]) swap(u, v); // 使满足u深度更大, 便于后面操作 
	int i = -1, j;
	// i求的是最大二分跨度 
	while ((1 << (i + 1)) <= depth[u]) ++i;
	
	// 下面这个循环是为了让u和v到同一深度 
	for (j = i; j >= 0; --j) {
		if (depth[u] - (1 << j) >= depth[v]) { // 是>=, 因为如果<,代表跳过头了,跳到了上面. 
			u = parents[u][j];
		}
	}
	
	if (u == v) return u; // 刚好是祖宗 
	
	// u和v一起二分找祖宗
	for (j = i; j >= 0; --j) {
		if (parents[u][j] != parents[v][j]) {
			u = parents[u][j];
			v = parents[v][j];
		}
	}
	return parents[u][0]; // 说明上个循环迭代到了Lca的子结点 
}
```

 - 首先把u调整到深度更大(或相同)的结点, 便于后面操作.
   
 -  然后获取最大跨度i, 所有的跨越都是从i开始的.
   
 - 再然后把u上升到和v一样的深度. 也就是我们前面讨论过的情况二转情况一.
   
 - 最后, 两个结点同时迭代, 直到找到Lca

至此, 我们的问题就解决了.

## 完整代码

```
#include <iostream>
#include <algorithm>
#include <cstring>
#include <queue>
#include <vector>
using namespace std;

const int maxn = 10005;
int parents[maxn][20], depth[maxn];
int n, from[maxn], root = -1;
vector<int> G[maxn];

void init()
{
	memset(parents, -1, sizeof(parents));
	memset(from, -1, sizeof(from));
	memset(depth, -1, sizeof(depth));
}

void getData()
{
	cin >> n;
	int u, v;
	for (int i = 1; i < n; ++i) {
		cin >> u >> v;
		G[u].push_back(v);
		parents[v][0] = u;
		from[v] = 1;
	}
	for (int i = 1; i <= n; ++i) {
		if (from[i] == -1) root = i;
	}
}

void getDepth_dfs(int u)
{
	int len = G[u].size();
	for (int i = 0; i < len; ++i) {
		int v = G[u][i];
		depth[v] = depth[u] + 1;
		getDepth_dfs(v);
	}
}

void getDepth_bfs(int u)
{
	queue<int> Q;
	Q.push(u);
	while (!Q.empty()) {
		int v = Q.front();
		Q.pop();
		for (int i = 0; i < G[v].size(); ++i) {
			depth[G[v][i]] = depth[v] + 1;
			Q.push(G[v][i]);
		}
	}
}

void getParents()
{
	for (int up = 1; (1 << up) <= n; ++up) {
		for (int i = 1; i <= n; ++i) {
			parents[i][up] = parents[parents[i][up - 1]][up - 1];
		}
	}
}

int Lca(int u, int v)
{
	if (depth[u] < depth[v]) swap(u, v);
	int i = -1, j;
	while ((1 << (i + 1)) <= depth[u]) ++i;
	for (j = i; j >= 0; --j) {
		if (depth[u] - (1 << j) >= depth[v]) {
			u = parents[u][j];
		}
	}
	if (u == v) return u;
	for (j = i; j >= 0; --j) {
		if (parents[u][j] != parents[v][j]) {
			u = parents[u][j];
			v = parents[v][j];
		}
	}
	return parents[u][0];
}

void questions()
{
	int q, u, v;
	cin >> q;
	for (int i = 0; i < q; ++i) {
		cin >> u >> v;
		int ans = Lca(u, v);
		cout << ans << endl;
		//cout << u << " 和 " << v << " 的最近公共祖先(LCA)是: " << ans << endl; 
	}
}

int main()
{
	init();
	getData();
	depth[root] = 1;
	getDepth_dfs(root);
	//getDepth_bfs(root);
	getParents();
	questions();
}
/*
9
1 2
1 3
1 4
2 5
2 6
3 7
6 8
7 9
5
1 3
5 6
8 9
8 4
5 8
*/
```