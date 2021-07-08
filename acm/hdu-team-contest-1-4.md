---
title: HDU ACM练习赛1 硬币游戏
date: 2020-03-03 11:00:00
tags:
- dp
- 区间dp
categories:
- ACM
description: 一道环形区间dp的题目
excerpt: 一道环形区间dp的题目
summary: 一道环形区间dp的题目
enableMathJax: true
katex: true
aliases:
- /ACM/hdu-team-contest-1-4
---
### 题意简述
$n$个硬币围成一个环，每次可以选一个代替他顺时针方向的下一个，获得两个硬币面值差的绝对值的价值，问最大能获得多少价值
### 题解
因为每次都是顺时针，所以肯定是在一块取一个区间，再另一块取一个区间，最后再合并，对于环我们可以复制两倍破环诚链，之后就是区间dp的模板，我们定义$f[l][r]$表示从$l$到$r$合并起来的最大价值
对于断点$k$，一定是把$[l,k]$和$[k+1,r]$的区间合并，由于都是顺时针所以前一个剩下$a[l]$后一个剩下$a[k+1]$，然后统计更新即可
注：由于题目已经关闭所以无法测试代码正确性，大概是对的
### 代码
```cpp
const int inf=0x3f3f3f3f;
const int maxn=1000;
int n,a[maxn],f[maxn][maxn];
int main()
{
	cin>>n;
	for(int i=1;i<=n;i++) cin>>a[i],a[i+n]=a[i];
	for(int len=2;len<=n;len++)
	{
		for(int l=1;l<=2*n-len+1;l++)
		{
			int r=l+len-1;
			for(int k=l;k<=r-1;k++)
			{
				f[l][r]=max(f[l][r],f[l][k]+f[k+1][r]+abs(a[l]-a[k+1]));
			}
		}
	}
	int ans=0;
	for(int i=1;i<=n+1;i++) ans=max(ans,f[i][i+n-1]);
	cout<<ans<<endl;
	return 0;
}
```