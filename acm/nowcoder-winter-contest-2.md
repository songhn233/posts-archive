---
title: 2020牛客寒假算法基础集训营2 题解
date: 2020-02-09 11:00:00
aliases:
- /ACM/nowcoder-winter-contest-2
categories:
- ACM
description: 牛客寒假第二场的部分题目题解与记录
excerpt: 牛客寒假第二场的部分题目题解与记录
summary: 牛客寒假第二场的部分题目题解与记录
enableMathJax: true
katex: true
tags:
- 动态规划
- 贪心
- 数学
- 数据结构
- 排序
- 思维
- 枚举
---
~~这场打着打着就睡着了~~，所以基本都是赛后补题，感觉这场有点数学场
### C. 算概率
#### 题意简述
有$n$道题目，每道题目有一个正确概率$p_i$，问这$n$道题里恰好做出$0,1,2,\cdots ,n$道的概率分别是多少，对$10^9+7$取模
注意这里对于分数$\frac{a}{b}$取模的意思是找到一个$q$使得$b\cdot q\mod(10^9+7)=a$
#### 题解
这个其实算是比较裸的二维dp吧）
设$f[i][j]$表示前$i$个数做对$j$个数的概率，可以以第$i$个数是否作对为切口，可得状态转移方程$f[i][j]=f[i-1][j]\cdot(1-p_i)+f[i-1][j-1]\cdot p_i$ 
注意在模的意义下$(1-p_i)$还是保持原来的性质的，可以设$\frac{b-a}{b}=q$和$\frac{a}{b}=q$不难推导出$p=q$，之后就可以dp了，稍微注意一下初始化就行

#### AC代码
```cpp
const int maxn=2050;
const ll mod=1e9+7;
ll n,p[maxn],f[maxn][maxn];
int main()
{
	cin>>n;
	for(int i=1;i<=n;i++) cin>>p[i];
	f[0][0]=1;
	for(int i=1;i<=n;i++)
	{
		f[i][0]=f[i-1][0]*(1-p[i]+mod)%mod;
		for(int j=1;j<=i;j++)
			f[i][j]=(f[i-1][j-1]*p[i]+f[i-1][j]*(1-p[i]+mod))%mod;
	}
	for(int i=0;i<=n;i++) cout<<f[n][i]<<" ";
	cout<<endl;
	return 0;
}
```
### D. 数三角
#### 题意简述
给出$n$个点$(1\leq n\leq 500)$，求挑出三个点组成钝角三角形有多少种情况
#### 题解
一开始以为500不能$n^3$枚举，后来发现这个复杂度没问题啊）就注意一下怎么判断钝角三角形就好了，这场好像有点卡距离的精度，不过本来也应该尽量避免浮点运算，我们可以用向量来处理，两个向量点乘小于0是钝角，还要考虑一下三点共线的情况，叉乘为0三点贡献把这种情况去除即可
#### AC代码
```cpp
const int maxn=550;
int n;
int x[maxn],y[maxn];
bool calc(int a,int b,int c)
{
	int flag=1;
	if((x[c]-x[a])*(x[c]-x[b])+(y[c]-y[a])*(y[c]-y[b])>=0) return false;
	if((x[c]-x[a])*(y[c]-y[b])-(x[c]-x[b])*(y[c]-y[a])==0) return false;
	return true;
}
int main()
{
	cin>>n;
	int ans=0;
	for(int i=1;i<=n;i++) cin>>x[i]>>y[i];
	for(int i=1;i<=n;i++)
	{
		for(int j=i+1;j<=n;j++)
		{
			for(int k=j+1;k<=n;k++)
			{
				if(calc(i,j,k)||calc(i,k,j)||calc(j,k,i)) ans++;
			}
		}
	}
	cout<<ans<<endl;
	return 0;
}
```
### E. 做计数
#### 题意简述
找出不同的三元组$(i,j,k)$满足$\sqrt{i}+\sqrt{j}=\sqrt{k}$且$i\cdot j\leq n$  当$i,j,k$任意有一个不同时就认为三元组不同$(1\leq n\leq4\cdot 10^7)$
#### 题解
~~果然是好久没写高中数学了，这种技巧都忘记了~~
乍一看难以处理，我们其实可以两边平方变形成$i+j+\sqrt{i\cdot j}=k$，这样$i\cdot j$一定要是完全平方数，我们可以枚举小于等于$n$的完全平方数，复杂度$O(\sqrt{n})$然后求这些数的约数对的个数即可，大概$O(n)$吧
#### AC代码
```cpp
int n;
vector<int> a;
int main()
{
	cin>>n;
	int ans=0;
	for(int i=1;i<=n/i;i++) a.push_back(i*i);
	for(int i=0;i<a.size();i++)
	{
		int m=a[i];
		for(int j=1;j<=m/j;j++)
		{
			if(m%j==0) ans+=2;
            if(j*j==m) ans--;
		}
	}
	cout<<ans<<endl;
	return 0;
}
```
### F. 拿物品
#### 题意简述
有$n$个数每个数两个属性$a_i,b_i$，两个人$A$与$B$轮流拿，最后$A$的分数是拿到的物品的$a$属性之和，$B$则是$b$属性之和，两人都希望最大化自己与对方的得分查，请你输出最后两人分别选择的集合
#### 题解
有点博弈论的味道，以为两个人都绝对聪明，我们不妨考虑一下交换两个物品比如$(a_1,b_1)$和$(a_2,b_2)$，那么$A'=A-a_1+a_2$,$B'=B-b_1+b_2$，假如这时候变得更优的话，那么$A'-B'>A-B$化简一下可得$a_1+b_1<a_2+b_2$而$B$同理，所以他们都一定会取$a+b$较大的数，那么直接排序然后输出即可
#### AC代码
```cpp
const int maxn=200050;
int n;
struct node{
	int id,a,b;
	bool operator<(const node&t) const{
		return (a+b)>(t.a+t.b);
	}
}c[maxn];
int main()
{
	cin>>n;
	for(int i=1;i<=n;i++) c[i].id=i;
	for(int i=1;i<=n;i++) read(c[i].a);
	for(int i=1;i<=n;i++) read(c[i].b);
	sort(c+1,c+n+1);
	for(int i=1;i<=n;i++) if(i&1) cout<<c[i].id<<" ";
	cout<<endl;
	for(int i=1;i<=n;i++) if(!(i&1)) cout<<c[i].id<<" ";
	return 0;
}
```
### G. 判正误
#### 题意简述
给出$a,b,c,d,e,f,g$，判断$a^d+b^e+c^f=g$是否正确$(-10^9\leq a,b,c,g\leq 10^9,0\leq d,e,f\leq 10^9)$
#### 题解
这题显然无法直接写~~直接pow竟然可以过~~我们可以通过模相等来判断相等，为了防止被卡多取几个模数即可，不过赛场上写的代码太丑陋了，或许以后会改良一下吧）
#### AC代码
```cpp
const ll p1=1222827239;
const ll p2=1e9+7;
const ll p3=2181271;
int T;
ll a,b,c,d,e,f,g;
ll p[]={1222827239,1000000007,2181271,122777,51787,9209};
ll ksm(ll a,ll b,int i)
{
	ll res=1%p[i];
	while(b)
	{
		if(b&1) res=(res*a)%p[i];
		a=(a*a)%p[i];
		b>>=1;
	}
	return res;
}
int main()
{
	cin>>T;
	while(T--)
	{
		cin>>a>>b>>c>>d>>e>>f>>g;
		ll t1,t2,t3;
		int flag=1;
		for(int i=0;i<6;i++)
		{
			
		if(a<0) 
		{
			t1=ksm(-a,d,i);
			if(d%2==1) t1=(p[i]-t1)%p[i];
		}
		else if(a==0) t1=0;
		else t1=ksm(a,d,i);
		
		if(b<0) 
		{
			t2=ksm(-b,e,i);
			if(e%2==1) t2=(p[i]-t2)%p[i];
		}
		else if(b==0) t2=0;
		else t2=ksm(b,e,i);
		
		if(c<0) 
		{
			t3=ksm(-c,f,i);
			if(f%2==1) t3=(p[i]-t3)%p[i];
		}
		else if(c==0) t3=0;
		else t3=ksm(c,f,i);
		if(((t1+t2+t3)%p[i]+p[i])%p[i]==(g%p[i]+p[i])%p[i]) continue;
		else flag=0;
		}
		if(flag==0) puts("No");
		else puts("Yes");
	}
	
	return 0;
}
```
### H. 施魔法
#### 题意简述
有$n$个元素，每次可以至少选择$k$个元素消除，会消耗这些元素极差的能量，每个元素要恰好被消灭1次，问至少需要多少力量$(1\leq k\leq n\leq 3\cdot 10^5,0\leq a_i \leq 10^9)$
#### 题解
既然是极差的话我们首先很自然的想法就是排序，排完序以后的话因为必须至少取$k$个数，假设当前取到$i$的话，那么它可以从$i-k+1$到$1$的任意一个位置一直取到$i$，当然前提是取了以后前面也要能够构成至少$k$这样的性质，我们可以考虑一个dp，$f[i]$表示为前$i$个所需要的最小能量，转移方程如下
$f[i]=min(f[i],f[j-1]+a[i]-a[j]),(1\leq j\leq i-k+1)$
也就是去枚举断点$j$，注意一下由于前$k-1$个$f$显然不成立，因为哪怕全包含都小于$k$，所以我们初始化时候全复制为inf这样就不会更新答案了，我们把$a[i]$提出来，也就是说我们只要去保存$f[j-1]-a[j]$的最小值temp然后每次去更新即可，最后输出$f[n]$
#### AC代码
```cpp
const int maxn=300050;
ll n,k,a[maxn],f[maxn];
int main()
{
	cin>>n>>k;
	for(int i=1;i<=n;i++) read(a[i]);
	sort(a+1,a+n+1);
	for(int i=1;i<k;i++) f[i]=inf;
	ll temp=-a[1];
	for(int i=k;i<=n;i++)
	{	
		f[i]=temp+a[i];
		temp=min(temp,f[i-k+1]-a[i-k+2]);
	}
	cout<<f[n]<<endl;
	return 0;
}
```
### I. 建通道
#### 题意简述
把$n$个数相互建一些边要求所有数之间都存在一个路径可以到达，每条边所连的数$i,j$产生能量$lowbit(a[i]^a[j])$，求使得所消耗的能量最少的建图方式$(1\leq n\leq 2\cdot 10^5,0\leq v_i\leq 2^30)$
#### 题解
因为$lowbit$的性质，所以我们肯定要对它造成贡献的1的位置越小越好，所以我们只要从小往大找数位，找到一个位既有0又有1即可，这样所有该位为0的往1连，所有该位为1的往0连，最后这两个一连，总共$n-1$条边，因为之前所有位所有的数都是相等的，所以异或为0，直到这一位产生了贡献，最后统计答案即可，注意有可能所有数或者一些数相等的情况，因为这些数连在一起异或起来肯定是0，所以可以先做去重即可
```cpp
const int maxn=200050;
ll n,a[maxn];
ll vis[35][2],ans;
int main()
{
	cin>>n;
	for(int i=1;i<=n;i++) read(a[i]);
	sort(a+1,a+n+1);
	n=unique(a+1,a+n+1)-(a+1);
	for(int i=1;i<=n;i++) 
	{
		for(int j=0;j<=30;j++)
		{
			if((a[i]>>j)&1) vis[j][1]=1;
			else vis[j][0]=1;
		}
	}
	for(int i=0;i<=30;i++)
	{
		if(vis[i][0]&&vis[i][1])
		{
			ans=(1ll<<i)*(n-1);
			break;
		}
	}
	cout<<ans<<endl;
	return 0;
}
```