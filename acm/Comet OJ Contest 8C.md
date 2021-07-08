---
title: Comet OJ Contest 8C 符文能量
date: 2019-08-27 09:00:00
enableMathJax: true
katex: true
tags:
- dp
- 线性dp
categories:
- ACM
---
### 题目大意：
给定一个长度为$n$的二元组序列$(a_i,b_i)$ 对相邻的$i-1$和$i$合并两个二元组的代价为$b_{i-1}\cdot a_i$ 你还可以选择一个区间 使得这个区间的所有二元组变为$(k\cdot a_i,k\cdot b_i)$ 你也可以不选 求将$n$个区间合并成一个区间的最小代价
<!--more-->
### 题解：
首先假如没有操作的话 合并成一个区间的贡献就是$\sum_{i=2}^{n}b_{i-1}\cdot a[i]$ 不会由于合并顺序的改变而变化 ~~我一开始以为是合并石子ex 也没高兴推~~

所以我们只要考虑对一段区间加上操作就好了 这就可以进行线性dp 

设$f[i][0]$为到第$i$个数还没有进行操作的最小代价

设$f[i][1]$为到第$i$个数正在操作序列中的最小代价

设$f[i][2]$为到第$i$个数已经操作过且第$i$个数不在操作范围里

那么转移方程就可以写成：

$f[i][0]=f[i-1][0]+b_{i-1}\cdot a_i$

$f[i][1]=min(f[i-1][0]+k\cdot b_{i-1}\cdot a_i,f[i-1][1]+k^2\cdot b_{i-1}\cdot a_i)$

$f[i][2]=min(f[i-1][2]+b_{i-1}\cdot a_i,f[i-1][1]+k\cdot b_{i-1}\cdot a_i)$

最后答案即为$min(f[n][0],f[n][1],f[n][2])$

### AC代码：
```cpp
#include<cstdio>
#include<iostream>
#include<cstring>
#include<algorithm>
#define ll long long
using namespace std;
const int maxn=100050;
const int inf=0x3f3f3f3f;
ll n,k,a[maxn],b[maxn];
ll f[maxn][3];
int main()
{
	cin>>n>>k;
	for(int i=1;i<=n;i++) cin>>a[i]>>b[i];
	for(int i=2;i<=n;i++)
	{
		f[i][0]=f[i-1][0]+b[i-1]*a[i];
		f[i][1]=min(f[i-1][0]+k*b[i-1]*a[i],f[i-1][1]+k*k*b[i-1]*a[i]);
		f[i][2]=min(f[i-1][2]+b[i-1]*a[i],f[i-1][1]+k*b[i-1]*a[i]);
	}
	cout<<min(f[n][0],f[n][2])<<endl;
	return 0;
}
```

