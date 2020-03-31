

**学过矩阵学了矩阵再看斐波那契数列, 秒懂, 结合矩阵快速幂, 加深了一个概念的理解: 矩阵也就是一个基本的计算单位.**

矩阵快速幂解法其实就是**快速幂+矩阵.** 

和普通的快速幂有什么不同? 不同的是**基数的类型**,快速幂的过程还是一样的. 同样的,快速幂结果一般取模, 因为数据实在是太大了. 那么矩阵快速幂是否也应该取模?

那么推导一下似乎可以发现,**矩阵的每个数都取模p,因为结果其实就是矩阵的某个元素** 

以[[1, 1], [1, 0]]的幂为n, n = 0时, 结果0, 如果我们取f0 = 0, f1 = 1, 以此类推,那么**n次幂的结果就是右下角的那个数.** 

同时**时间复杂度为O(n log n).** 

我们来回顾一下快速幂, ***是从一次不断进行倍增, 如果数n 的二进制形式在某一位上为1,那么就乘以这一位,否则不乘*** 

先写一下快速幂代码. 
```c++
#include <iostream>
using namespace std;

int main()
{
	int n, p, ans = 1, val = 2;
	cin >> n >> p;
	while (n) {
		if (n & 1) ans = ans * val % p;
		val *= val;
		n /= 2;
	}
	cout << ans;
}
// OK, 就是这么简单.
```
----------
那么矩阵快速幂也呼之欲出了. 

然而写了代码发现计算过程有误.(划掉, 应该是推导有误,睡个午觉再来看看). 
OK, 找出bug来了, 推导没有错误, **是做矩阵乘法的时候几个下标写错了. 需要特别小心**

但是发现答案稍微有点问题**,看出其实now矩阵不用设置为二维单位矩阵,直接是[1, 0], 他就是答案. **

但是非常悲剧的是洛谷上没有这么简单的模板题, 都**不给我辛苦学习AC一下的快感. **

```
#include <iostream>
using namespace std;

int main()
{
	int n, p = (1 << 31);
	cin >> n;
	int now[2][1] = {{1}, {0}}, matrix[2][2] = {{1, 1}, {1, 0}};
	while (n) {
		int a, b, c, d;
		if (n & 1) { // 矩阵相乘. 
			a = matrix[0][0] * now[0][0] + matrix[0][1] * now[1][0];
			b = matrix[1][0] * now[0][0] + matrix[1][1] * now[1][0];
			now[0][0] = a % p, now[1][0] = b % p;
		}
		// 下面是矩阵平方, 有点麻烦. 
		a = matrix[0][0] * matrix[0][0] + matrix[0][1] * matrix[1][0];
		b = matrix[0][0] * matrix[0][1] + matrix[0][1] * matrix[1][1];
		c = matrix[1][0] * matrix[0][0] + matrix[1][1] * matrix[1][0];
		d = matrix[1][0] * matrix[0][1] + matrix[1][1] * matrix[1][1];
		//cout << "c = " << c << endl; 
		matrix[0][0] = a % p, matrix[0][1] = b % p, matrix[1][0] = c % p, matrix[1][1] = d % p;
		n /= 2;
		//cout << "----------" << endl;
		//cout << matrix[0][0] << ' ' << matrix[0][1] << endl << matrix[1][0] << ' ' << matrix[1][1] << endl;
	}
	cout << now[1][0];
}
```

再来串矩阵快速幂模板(不只是斐波那契矩阵)

也就是**矩阵计算 + 快速幂**的组合. **O(n**3 + log k)**
```
#include <iostream>
#include <cstring>
using namespace std;
typedef long long LL;

const LL mode = 1000000007;

void calc(LL ans[105][105], LL matrix[105][105], LL n)
{
	LL tmp[105][105];
	memset(tmp, 0, sizeof(tmp));
	for (int i = 1; i <= n; ++i) {
		for (int j = 1; j <= n; ++j) {
			for (int k = 1; k <= n; ++k) {
				tmp[i][j] += ans[i][k] * matrix[k][j];
				tmp[i][j] %= mode;
			}
		}
	}
	for (int i = 1; i <= n; ++i) {
		for (int j = 1; j <= n; ++j) {
			ans[i][j] = tmp[i][j];
		}
	}
}

int main()
{
	LL n, k, matrix[105][105], ans[105][105];
	cin >> n >> k;
	for (int i = 1; i <= n; ++i) {
		for (int j = 1; j <= n; ++j) {
			cin >> matrix[i][j];
		}
	}
	// 初始化为单位矩阵. 
	memset(ans, 0, sizeof(ans));
	for (int i = 1; i <= n; ++i) {
		ans[i][i] = 1;
	}
	while (k) {
		if (k & 1) {
			calc(ans, matrix, n);
		}
		calc(matrix, matrix, n);
		k /= 2;
	}
	for (int i = 1; i <= n; ++i) {
		for (int j = 1; j <= n; ++j) {
			cout << ans[i][j] << ' ';
		}
		cout << endl;
	}
}
/*
矩阵计算 + 快速幂
*/ 
```