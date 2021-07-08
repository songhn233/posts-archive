模板2运用到了差分的思想，维护一个数组去记录元素与前一个元素的差，在区间(l,r)加时只需要add(l,x),add(r+1,-x)，之后要求元素即为前缀和
```cpp
//模板1
#include<bits/stdc++.h>
using namespace std;
const int maxn=500050;
int c[maxn],a[maxn],m,n;
inline int read()
{
	int x=0,t=1;char ch=getchar();
	while(ch>'9'||ch<'0'){if(ch=='-')t=-1;ch=getchar();}
	while(ch<='9'&&ch>='0') x=x*10+ch-'0',ch=getchar();
	return x*t;
}
int lowbit(int x)
{
	return x&-x;
}
void add(int x,int num)
{
	while(x<=n)
	{
		c[x]+=num;
		x+=lowbit(x);
	}
}
int sum(int x)
{
	int s=0;
	while(x)
	{
		s+=c[x];
		x-=lowbit(x);
	}
	return s;
}
int main()
{
	n=read(),m=read();
	for(int i=1;i<=n;i++)
	{
		a[i]=read();
		add(i,a[i]);
	}
	for(int i=1;i<=m;i++)
	{
		int p;
		cin>>p;
		if(p==1)
		{
			int x,y;
			x=read(),y=read();
			add(x,y);
		}
		else
		{
			int x,y;
			x=read(),y=read();
			cout<<sum(y)-sum(x-1)<<endl;
		}
	}
	return 0;
}
```
```cpp
//模板2
#include<iostream>
#include<cstring>
#include<cstdio>
using namespace std;
int a[500050],b[500050];
int n,m;
int lowbit(int n)
{
	return n&-n;
}
void add(int x,int num)
{
	while(x<=n)
	{
		a[x]+=num;
		x+=lowbit(x);
	}
}
int query(int x)
{
	int sum=0;
	while(x)
	{
		sum+=a[x];
		x-=lowbit(x);
	}
	return sum;
}
int main()
{
	int t=0;
	ios::sync_with_stdio(false);
	cin>>n>>m;
	for(int i=1;i<=n;i++)
	{
		cin>>b[i];add(i,b[i]-t);t=b[i];
	}
	
	for(int i=1;i<=m;i++)
	{
		int c;
		cin>>c;
		if(c==1)
		{
			int x,y,k;
			cin>>x>>y>>k;
			add(x,k);
			add(y+1,-k);
		}
		else
		{
			int x;
			cin>>x;
			cout<<query(x)<<endl;
		}
	}
	return 0;
}
```