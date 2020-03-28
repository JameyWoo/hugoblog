---
title: "DP_最长回文子串"
date: 2018-7-11
draft: false

categories: ["算法"]
---


####DP问题, 最长回文子串

最长回文子串问题指的是在一个字符串中, 是回文子串的长度的最大值. 这里的回文子串是**连续的**. 

*如字符串"PATZJUJZTACCBCC", 他的最长回文子串是"ATZJUJZTA", 长度为9, 当然它还有其他回文子串如"CCBCC", 但是长度不够长.*

这类问题似乎有多种解法, 复杂度从O(n^3)到O(n)不等.

下面介绍一种**时间复杂度为O(n^2)**的.

思路是典型的DP思路, 我们可以考量这样一个数组, dp[i][j], bool类型, 值为1代表字符串从S[i]到S[j]是回文子串, 值为0代表不是.

那么对于任意的i, j, 如何判断dp[i][j]的值呢? 讨论下面两种情况:

**1. s[i] = s[j]时, 如果dp[i+1][j-1] = 0, 即从s[i+1]到s[j-1]是回文子串, 那么从s[i][j]自然是回文子串, 所以dp[i][j] = dp[i+1][j-1]**
**2.  s[i] != s[j]时, 如论如何dp[i][j] = 0**

那么这样看思路就非常清晰了, 看上去非常简单, 按照这个规则判断一下就好了.

不过其实这道题是非常有技巧的, 我们可以看到要获取dp[i][j]值, 那么就需要知道dp[i+1][j-1]的值, 那么就不能从前往后遍历i了, 怎么办呢? 从后往前遍历吗? 当然也不行. 这就是这种方法的精彩所在, 两套循环,**外围遍历长度, 内围遍历字符串起始点i.**

因为dp[i][j]的长度是j - i + 1, 而dp[i+1][j-1]的长度是j - i - 1, 长度差了两个, 如果我把长度小的结果都求解出来了, 那么长度更长的用长度更小的, 无论i是否会更大, 都是可以的. 不得不说很是精彩.

代码实现上, **注意前面初始化长度 2时对ans赋恰当的值.**

```
#include <iostream>
using namespace std;

int main()
{
	string str; 
	cin >> str;
	int dp[str.size() + 1][str.size() + 1] = {}, ans = 1;
	for (int i = 0; i < int(str.size()); ++i) {
		dp[i][i] = 1;
		if (str[i] == str[i + 1]) {
			dp[i][i + 1] = 1;
			ans = 2;
		}
	}
	for (int len = 3; len <= (int)str.size(); ++len) {
		for (int i = 0; i + len - 1 < (int)str.size(); ++i) {
			int k = i + len - 1;
			if (str[i] == str[k]) {
				dp[i][k] = dp[i + 1][k - 1];
				if (dp[i][k]) ans = len;
			}
		}
	}
	cout << ans;
}
/*
PATZJUJZTACCBCC    ans = 9

34567536487326483254   ans = 1
*/ 
```