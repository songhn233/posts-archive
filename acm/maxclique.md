---
title: 最大团与极大团算法
date: 2020-01-28 11:00:00
aliases:
- /算法笔记/maxclique
categories:
- 算法笔记
tags:
- 极大团
- 图论
description: 本片笔记主要介绍了极大团与最大团的概念，记录一种dfs计算最大团的方法，介绍了用于求解极大团的Bron-Kerbosch算法及相关优化
excerpt: 本片笔记主要介绍了极大团与最大团的概念，记录一种dfs计算最大团的方法，介绍了用于求解极大团的Bron-Kerbosch算法及相关优化
summary: 本片笔记主要介绍了极大团与最大团的概念，记录一种dfs计算最大团的方法，介绍了用于求解极大团的Bron-Kerbosch算法及相关优化
---
### 基本定义
首先看一下维基百科里对最大团与极大团的定义：
> 团是一个无向图的完全子图
> **极大团**是指增加任一顶点都不再符合团定义的团，也就是说，极大团不能被任何一个更大的团所包含
> **最大团**是一个图中顶点数最多的团

### 回溯法求解最大团
#### 搜索初步
计算最大团有许多算法，一般比较常见的dfs回溯法，首先考虑一下暴力的情况，即找到一个图的所有子图然后判断连通性，这样就可以计算出最大团，但是这样效率太低，下面考虑如何优化这种做法
首先dfs的主体是dfs(d,num)，d表示当前的深度也就是现在最大团的点的数量，num是备选集合，也就是在当前状态下还可以加哪些点还能继续保持完全图的性质
我们在搜索的时候首先确定一个初始点，然后找与这个点相连的边作为dfs的起点，注意为了避免重复的搜索所以通过固定搜索的顺序来起到优化的作用，代码如下：
```cpp
for(int i=n-1;i>=1;i--)
{
	tot=0;
	for(int j=i+1;j<=n;j++)
	{
		if(a[i][j]) temp[1][++tot]=j;
	}
	dfs(1,tot);
	f[i]=ans;
}
```
#### dfs部分
tot表示第一层的备选集合，由团定义可知两个点相连是一个完全图，所以把邻边直接加入备选集合即可，确定起点后后面节点的序号只能比起点大从而保证不会出现重复的搜索，关于外层循环为什么要反着搜索以及f数组的意义会在下文做说明
现在已经有了第一个点起点和后面的备选集合，我们可以开始dfs了，那么如何dfs呢？首先看一下比较简单的终止条件，什么时候会结束搜索，结果也很显然当备选集合为空的时候就已经是一个极大团了不能继续搜索，此时我们可以对最大团答案做一个更新
```cpp
if(num==0)
{
	if(d>ans) 
	{
		ans=d;
		return true;
	}
	else return false;
}
```
关于dfs采用bool返回值的原因会在下文说明）
下面就来到了整个搜索的主体部分，搜索备选集合，我们枚举其中的一个点把它作为下一个状态，接下来需要更新备选集合，这里也同样采用相似的限制搜索顺序的方法来避免重复搜索，假定选定节点i，那么备选集合就会从i+1到num里找防止重复搜索，假如这个j节点与i相连那么就把它加入备选集合，注意一下与i相连就符合了完全图的性质，下面来感性理解一下
因为每一层的备选集合都是从上一层的备选集合里继承过来的，备选集合放在一个数组里，每次只在数组里判断，所以在i层的这些备选集合假定为t，那么t里任意一点连上后都仍然满足完全图的性质，那么我选择一点为下一层扩展，其他点与这一层其他点都完全相连，那么它只要满足与选择的这个点相连就可以继续维持完全图的性质了
下面看一下初步的代码：
```cpp
for(int i=1;i<=num;i++)
{
	int v=temp[d][i];
	int cnt=0;
	for(int j=i+1;j<=num;j++)
	{
		int vv=temp[d][j];
		if(a[v][vv]) temp[d+1][++cnt]=vv;
	}
}
```
#### 剪枝及相关优化
其中表示下一层选定的点，然后循环找下一层的备选集合，简单介绍一下temp数组，其实就是备选数组，第一维为当前搜索到的层数，然后第二维是备选点的数量，保存的值是备选结点序号
这样的代码其实已经可以得出结果了，但在模板题[hdu1530]("http://acm.hdu.edu.cn/showproblem.php?pid=1530")上会超时
下面介绍一下常见的优化与剪枝
首先就是开头的倒序搜索，显然点的可能性少搜索花的时间也就少，而且我们可以得到当前的最大团，这样也就可以在搜索中剪枝，假如从前往后一开始就要面对巨大的搜索量就很容易超时
刚刚我们发现倒序搜索记录当前最大团后可以剪枝，下面就来介绍一下常见的两种剪枝

1.`if(d+num-i+1<=ans) return false;`
因为每次搜索都是在现有的备选集合里搜索，所以当前把第i个作为选择点的话，最多只能选择剩下的num-i+1个点（因为限制了搜索的顺序）那么假如当前有的点的个数d加上最多可选还是不能超过现有答案的话就直接剪掉
2.`if(d+f[v]<=ans) return false;`
首先来介绍一下f数组，~~一般认为这是dp优化虽然我感觉没啥关系~~，我们再回顾一下一开始的代码
```cpp
for(int i=n-1;i>=1;i--)
{
	tot=0;
	for(int j=i+1;j<=n;j++)
	{
		if(a[i][j]) temp[1][++tot]=j;
	}
	dfs(1,tot);
	f[i]=ans;
}
```
这个f数组用来记录把i作为起点的话最多能产生多大的最大团，注意这里其实不是把i作为起点，而且i以及i的所有后驱，因为我们在搜索的时候一定是从前往后的，所以假如到了i那么我们只能i的后续了，那假如i的后续所能组成的最大团加上当前的数量还是不能比当前答案大就剪掉，这个剪枝就是那么来的
这就是主要的两个剪枝，其实不少博客还有第三个剪枝但我觉得已经被第一个剪枝考虑了所以在此不再说明~~也许以后会补充一下~~
到这里已经基本完成了，下面是最后一步也就是之前提到的dfs值为bool
考虑一下，因为我们一次只会往前考虑一个结点，那么上一次搜索到的最大团，其实这次只能扩展一次，不然上一次的最大团就不会是它，用反证法不难证明，所以我们一旦更新了答案就可以立刻返回不需要继续搜索了，所以这是最后的一个剪枝
在经过这些优化dfs跑最大团已经效率很可观了，下面附上完整代码（hdu1530）：
#### 模板代码
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
const int maxn=55;
int n;
int a[maxn][maxn],f[maxn],temp[maxn][maxn],ans,tot;
bool dfs(int d,int num)
{
	if(num==0)
	{
		if(d>ans) 
		{
			ans=d;
			return true;
		}
		else return false;
	}
	for(int i=1;i<=num;i++)
	{
		if(d+num-i+1<=ans) return false;
		int v=temp[d][i];
		if(d+f[v]<=ans) return false;
		int cnt=0;
		for(int j=i+1;j<=num;j++)
		{
			int vv=temp[d][j];
			if(a[v][vv]) temp[d+1][++cnt]=vv;
		}
		if(dfs(d+1,cnt)) return true;
	}
	return false;
}
int main()
{
	while(~scanf("%d",&n))
	{
		if(n==0) break;
		memset(f,0,sizeof(f));
		for(int i=1;i<=n;i++) for(int j=1;j<=n;j++) read(a[i][j]);
		f[n]=ans=1;
		for(int i=n-1;i>=1;i--)
		{
			tot=0;
			for(int j=i+1;j<=n;j++)
			{
				if(a[i][j]) temp[1][++tot]=j;
			}
			dfs(1,tot);
			f[i]=ans;
		}
		cout<<ans<<endl;
	}
	return 0;
}
```