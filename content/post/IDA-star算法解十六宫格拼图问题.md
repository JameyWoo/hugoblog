---
title: "IDA Star算法解十六宫格拼图问题"
date: 2018-10-01
draft: false

categories: ["算法"]
---

IDA*算法, ID(Iterative Deepening)指的是迭代加深. 它的思想是重复进行限制最大深度的深度优先搜索(此限制从某个最小值遍历到最大值), 也称为深度受限搜索.

一般情况下, 为了提高搜索速度, 迭代加深不会记录已搜索过的状态, **但同时, 需要做一些调整, 以避免出现马上回溯到上一状态的情况.**

----------

IDA*算法的步骤

1) 首先对初始状态进行评估, 评估值作为最小限度, 而最大限度为自己的设置.
这个评估值在这个问题中可以用此状态到正确状态的每个位置的曼哈顿距离来表示.

2) 从最小限度到最大限度进行遍历, 此值作为当前dfs的限度值, 这个限度不断在有效范围内递增的过程就称作迭代加深

3) 进行dfs, 调整状态, 将新状态加入到新的dfs中, 直到找到了一个解(由于迭代加深, 此解为最优解). 进行回溯, 加入路径, 算法结束.

PS. 如果在限度内都没有找到解, 就输出`unsolved`. 

从上面的分析中可见, 即使是IDA*算法, 其局限性依然很大, 比如它需要设置一个最大限制, 而超出这个限制的状态将无法求解出. 

---
一些解释:

1. **曼哈顿距离预处理, 每个点在另一个位置的曼哈顿距离16*16**
x坐标距离 `abs(i / N - j / N)`
y坐标距离 `abs(i % N - j % N)`
曼哈顿距离可以将x坐标和y坐标相互独立开来, 且曼哈顿距离是相对的. 而在上面的表达式中, 可以理解为他们以(0, 0)为参照点;
即`abs((i / N - 0 - (j / N - 0))`

2. **状态的定义**
在这个IDA*算法中, 每个状态包含了以下信息.
	1) 16个数的位置
	2) 空格所在位置
	3) 当前状态距离正确状态的曼哈顿距离
	简单的结构体可实现

3. **dfs难道要遍历所有可能的情况? 不, 别忘了我们是迭代加深(Iterative Deepeni)!**
所谓迭代, 就是一代一代更迭, 所以, 既然我们确定了最大范围(LIMIT), 那么我们就可以在这个范围里再设置范围限制, 然后搜索(dfs).
每当找到了一个解, 这个解就是最优解, 因为更优解在我们之前的搜索中没有出现.

4. **不会绕圈吗?**
答: 会绕圈, 但是不会有很大影响, 因为我们设置了搜索次数, 所以绕圈多消耗步骤的自然会淘汰掉.

5. **如何理解`sum += MDT[i][pz.f[i] - 1];`**
`MDT[i][pz.f[i] - 1]`这个状态可以理解为, 在第i格位置, 当它的值为`pz.f[i]`时, 他们的曼哈顿距离之差. 为什么要减一? 因为输入的值为`1...15`, 而代码中都位置下标都是从0开始的.

---

IDA*完整代码
```cpp
#include <iostream>
#include <cmath>

using namespace std;
#define N 4
#define N2 16
#define LIMIT 57

static const int dx[4] = {0, -1, 0, 1};
static const int dy[4] = {1, 0, -1, 0};
static const char dir[4] = {'r', 'u', 'l', 'd'};
int MDT[N2][N2];

struct Puzzle {
	int f[N2], space, MD; // 位置, 空格, 曼哈顿距离
};

Puzzle state;
int limit;
int path[LIMIT];

int getALLMD(Puzzle pz) {
	int sum = 0;
	for (int i = 0; i < N2; ++i) {
		if (pz.f[i] == N2) continue;
		sum += MDT[i][pz.f[i] - 1];
	}
	return sum;
}

bool isSolved() {
	for (int i = 0; i < N2; ++i) {
		if (state.f[i] != i + 1) return false;
	}
	return true;
}

bool dfs(int depth, int prev) {
	if (state.MD == 0) return true; // 搜索到了答案.
	if (depth + state.MD > limit) return false; // 超过当前迭代限制
	
	int sx = state.space / N;
	int sy = state.space % N;
	Puzzle tmp;
	
	for (int r = 0; r < 4; ++r) {
		int tx = sx + dx[r];
		int ty = sy + dy[r];
		if (tx < 0 || ty < 0 || tx >= N || ty >= N) continue;
		if (max(prev, r) - min(prev, r) == 2) continue; // 妙! 避免迂回. 减少了很多不必要搜索
		tmp = state;
		
		state.MD -= MDT[tx * N + ty][state.f[tx * N + ty] - 1]; // 消除原位置的曼哈顿距离
		state.MD += MDT[sx * N + sy][state.f[tx * N + ty] - 1]; // 添加新位置的曼哈顿距离, 注意, MDT由非0/16产生
		swap(state.f[tx * N + ty], state.f[sx * N + sy]);
		state.space = tx * N + ty;
		if (dfs(depth + 1, r)) { // 先搜索, 搜索成功后再添加路径. 巧妙, 值得学习
			path[depth] = r;
			return true;
		}
		state = tmp; // 回溯复原
	}
	return false;
}

string iterative_deepening(Puzzle in) {
	in.MD = getALLMD(in);
	
	for (limit = in.MD; limit <= LIMIT; limit++) { // 绝了, 原来是这样加一个常数
		state = in;
		if (dfs(0, -100)) {
			string ans = "";
			for (int i = 0; i < limit; ++i) ans += dir[path[i]];
			return ans;
		}
	}
	return "unsolvable";
}

int main()
{
	for (int i = 0; i < N2; ++i) {
		for (int j = 0; j < N2; ++j) {
			MDT[i][j] = abs(i / N - j / N) + abs(i % N - j % N);
		}
	}
	
	Puzzle in;
	
	for (int i = 0; i < N2; ++i) {
		cin >> in.f[i];
		if (in.f[i] == 0) {
			in.f[i] = N2;
			in.space = i;
		}
	}
	string ans = iterative_deepening(in);
	if (ans != "unsolvable") cout << ans.size() << endl;
	cout << ans << endl;
}
```
参考数据:

```cpp
6 13 5 2
8 1 10 12
3 7 15 9
14 4 0 11 // 53

1 2 3 4
6 7 8 0
5 10 11 12
9 13 14 15  // 8

5 8 9 14
10 13 1 6
12 2 7 15
4 0 3 11  // 56

12 7 2 4
5 1 0 9
14 13 6 8
3 15 10 11 // 47

5 11 10 7
13 0 9 3
14 2 4 8
1 15 6 12  // 38

5 1 4 7
2 0 11 3
9 6 10 8
13 14 15 12  // 14

9 14 13 15
5 3 11 6
8 12 2 1
10 7 4 0  // unsolvable
```
十六宫格随机数据: **排列置乱算法**

```cpp
#include <iostream>
#include <cstdlib>
#include <ctime>
#define RAND(l, r) l+(int)(r-l+1)*rand()/(RAND_MAX+1)
using namespace std;

int main()
{
	srand(time(NULL));
	int data[16];
	for (int i = 0; i < 16; ++i) {
		data[i] = i;
	}
	for (int i = 15; i >= 0; --i) {
		int ind = RAND(0, i);
		swap(data[i], data[ind]);
	}
	for (int i = 0; i < 4; ++i) {
		for (int j = 0; j < 4; ++j) {
			cout << data[i*4 + j] << ' ';
		}
		cout << endl;
	}
}
```
