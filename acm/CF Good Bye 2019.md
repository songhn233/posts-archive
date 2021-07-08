---
title: CF Good Bye 2019 题解
categories:
- ACM
tags:
- 构造
- 思维
- codeforces
enableMathJax: true
katex: true
date: 2020-01-10 09:00:00
---
时隔许久的题解，A到D题，感觉全是构造题啊）
<!--more-->
### A. Card Game

#### 题意简述：

有两个人玩游戏，第一个人有$k1$张牌，第二个人有$k2$张牌，一共有$n$张牌且牌值从$1$到$n$，每一次两人分别抽出一张牌，谁的牌大就拿走这两张牌，最后拿光对方的牌赢，问先手赢还是后手赢
#### 题解：
有最大的牌即值为$n$的牌的人每次出这张牌就能赢，读入时候判断一下即可
#### AC代码：
```cpp
#include<cstdio>
#include<algorithm>
#include<cstring>
#include<iostream>
#include<vector>
#include<queue>
#include<cmath>
#include<map>
#include<set>
#define ll long long
#define F(i,a,b) for(int i=(a);i<=(b);i++)
#define mst(a,b) memset((a),(b),sizeof(a))
#define PII pair<int,int>
using namespace std;
template<class T>inline void read(T &x) {
    x=0; int ch=getchar(),f=0;
    while(ch<'0'||ch>'9'){if (ch=='-') f=1;ch=getchar();}
    while (ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch^48);ch=getchar();}
    if(f)x=-x;
}
const int inf=0x3f3f3f3f;
const int maxn=150;
int T;
int n,k1,k2;
int a[maxn],b[maxn]; 
int main()
{
	cin>>T;
	while(T--)
	{
		cin>>n>>k1>>k2;
		int pan=0;
		for(int i=1;i<=k1;i++) 
		{
			read(a[i]);
			if(a[i]==n) pan=1;
		}
		for(int i=1;i<=k2;i++)
		{
			read(b[i]);
			if(b[i]==n) pan=2;
		}
		if(pan==1) puts("YES");
		else puts("NO");
	}
	
	return 0;
}
```
### B. Interesting Subarray
#### 题意简述：
如果一个序列的最大值减去最小值大于等于这个序列的长度我们就认为这个序列是好的，给定一个$a$序列，假如能找到其中的一个子序列是好的，输出YES，反之输出NO，子序列是指把原序列开头或者结尾删除一段连续数后得到的非空序列
#### 题解：
个人认为是一道挺有趣的题，以为一开始很容易往扫描然后各种分析性质的算法上去靠，其实这应该算是一个结论题或者说证明题
首先给出结论：
> 若不存在相邻的两个数相差大于1就为NO，否则为YES

接下来考虑证明一下，YES很显然，因为相邻两数的长度为2，相差最少也是2所以肯定满足。
下面考虑一下不存在相差大于1，那么最大也就是1，考虑一下相差全是1，那么$k$个数之间就差$k-1$但总长度是$k$，不满足，加上$-1$和$0$的情况不会变得更优，所以永远不可能大于等于序列长度，即为NO，证毕
#### AC代码：
```cpp
#include<cstdio>
#include<algorithm>
#include<cstring>
#include<iostream>
#include<vector>
#include<queue>
#include<cmath>
#include<map>
#include<set>
#define ll long long
#define F(i,a,b) for(int i=(a);i<=(b);i++)
#define mst(a,b) memset((a),(b),sizeof(a))
#define PII pair<int,int>
using namespace std;
template<class T>inline void read(T &x) {
    x=0; int ch=getchar(),f=0;
    while(ch<'0'||ch>'9'){if (ch=='-') f=1;ch=getchar();}
    while (ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch^48);ch=getchar();}
    if(f)x=-x;
}
const int inf=0x3f3f3f3f;
const int maxn=200050;
int T;
ll n,a[maxn];
int main()
{
	cin>>T;
	while(T--)
	{
		read(n);
		for(int i=1;i<=n;i++) read(a[i]);
		int l,r;
		l=r=-1;
		for(int i=2;i<=n;i++)
		{
			if(abs(a[i]-a[i-1])>=2)
			{
				l=i-1,r=i;
				break;
			}
		}
		if(l==r&&l==-1) puts("NO");
		else 
		{
			puts("YES");
			cout<<l<<" "<<r<<endl;
		}
	}
	return 0;
}
```
### C. Make Good
#### 题意简述：
我们认为一个序列$a_1,a_2,\cdots,a_m$是好的如果$a_1+a_2+\cdots+a_m=2\cdot(a_1\oplus a_2\oplus\cdots\oplus a_m)$
现在给你一个$a$序列，让你添加不超过$3$个数使得$a$序列是好的
#### 题解：
又是一个构造题，这个题可以这么想，前面一堆杂乱的$a$很碍事麻烦我们构造，我们先要把它们消掉，由于一个性质$a\oplus a=0$所以我们计算出$a_1\oplus a_2\oplus\cdots\oplus a_m$记为$xor_a$然后先异或掉这样左边变为$sum_a+xor_a$，右边变为$0$,$0$异或任何数都等于那个数本身，现在已经很显然了，两边再加$sum_a+xor_a$即可满足
#### AC代码：
```cpp
#include<cstdio>
#include<algorithm>
#include<cstring>
#include<iostream>
#include<vector>
#include<queue>
#include<cmath>
#include<map>
#include<set>
#define ll long long
#define F(i,a,b) for(int i=(a);i<=(b);i++)
#define mst(a,b) memset((a),(b),sizeof(a))
#define PII pair<int,int>
using namespace std;
template<class T>inline void read(T &x) {
    x=0; int ch=getchar(),f=0;
    while(ch<'0'||ch>'9'){if (ch=='-') f=1;ch=getchar();}
    while (ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch^48);ch=getchar();}
    if(f)x=-x;
}
const int inf=0x3f3f3f3f;
const int maxn=100050;
int T,n;
int main()
{
	cin>>T;
	while(T--)
	{
		ll sum=0,temp=0;
		cin>>n;
		for(int i=1;i<=n;i++)
		{
			ll x;read(x);
			sum+=x;
			temp^=x;
		}
		cout<<2<<endl;
		cout<<temp<<" "<<sum+temp<<endl;
	}
	return 0;
}
```
### D. Strange Device
#### 题意简述：
这是一个交互题，给定$n$和$k$，一个长度为$n$的序列，每次询问询问$k$个下标，返回这$k$个数中递增排序为第$m$的数的下标和值，最多询问$n$次，求$m$的值

#### 题解：
这道题也是构造，其实可以从样例中摸到规律，我们对前$k+1$个数查询$k+1$次即可，即第$i$次查询跳过第$i$个数，我们会发现，对于其中排名为$m+1$的数，由于前$m$个都会被删掉一次，所以一共会出现$m$次，排名为$m$的数一共会出现$k+1-m$次，而且只有可能出现这两个值，那么我们在查询时候存一下更大的那个值，用一个map维护值出现的次数，最后输出排名为$m+1$的数的map值即可
#### AC代码：
```cpp
#include<cstdio>
#include<algorithm>
#include<cstring>
#include<iostream>
#include<vector>
#include<queue>
#include<cmath>
#include<map>
#include<set>
#define ll long long
#define F(i,a,b) for(int i=(a);i<=(b);i++)
#define mst(a,b) memset((a),(b),sizeof(a))
#define PII pair<int,int>
using namespace std;
template<class T>inline void read(T &x) {
    x=0; int ch=getchar(),f=0;
    while(ch<'0'||ch>'9'){if (ch=='-') f=1;ch=getchar();}
    while (ch>='0'&&ch<='9'){x=(x<<1)+(x<<3)+(ch^48);ch=getchar();}
    if(f)x=-x;
}
const int inf=0x3f3f3f3f;
int n,k;
map<int,int> mp;
int main()
{
	cin>>n>>k;
	int pan=0;
	for(int i=1;i<=k+1;i++)
	{
		printf("?");
		for(int j=1;j<=k+1;j++)
		{
			if(j!=i) printf(" %d",j);
		}
		printf("\n");
		fflush(stdout);
		int pos,val;
		cin>>pos>>val;
		mp[val]++;
		pan=max(pan,val); 
	}
	cout<<"! "<<mp[pan]<<endl;
	return 0;
}
```
