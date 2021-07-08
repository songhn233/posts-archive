---
title: 数论基础学习笔记1
date: 2019-08-22 11:00:00
aliases:
- /算法笔记/math1
tags:
- 数学
categories:
- 算法笔记
enableMathJax: true
katex: true
---
### 1. 质数
#### 1.1 试除法判定$n$是否为质数
```cpp
int is_prime(int n)
{
	if(n<2) return false;
	for(int i=2;i<=n/i;i++)
	if(n%i==0) return false;
	return true;
}
```
<!--more-->
#### 1.2 质因数分解
```cpp
for(int i=2;i<=n/i;i++)
{
	if(n%i==0)
	{
		int t=0;
		while(n%i==0) t++,n/=i;
		cout<<i<<" "<<t<<endl;
	}
}
if(n>1) cout<<n<<" "<<1<<endl;
```
#### 1.3 筛质数
埃筛
```cpp
void is_prime()
{
	for(int i=2;i<=n;i++)
	{
		if(vis[i]) continue;
		prime[++num]=i;
		for(int j=i+i;j<=n;j+=i)
		vis[j]=1;
	}
}
```
线性筛
```cpp
void is_prime()
{
	for(int i=2;i<=n;i++)
	{
		if(!vis[i]) prime[++num]=i;
		for(int j=1;prime[j]<=n/i;j++)
		{
			vis[prime[j]*i]=1;
			if(i%prime[j]==0) break;
		}
	}
}
```
### 2. 约数
#### 2.1 试除法求约数
```cpp
for(int i=1;i<=n/i;i++)
{
	if(n%i==0)
	{
		res.push_back(i);
		if(i!=n/i) res.push_back(n/i);
	}
}
sort(res.begin(),res.end());
```
#### 2.2 约数的个数
对于$n$质因数分解 使得$n=p_1^{c_1}\cdot p_2^{c_2}\cdots p_k^{c_k}$

则约数个数为$(c_1+1)\cdot(c_2+1)\cdots(c_k+1)$
#### 2.3 所有约数的和
对于$n$质因数分解 使得$n=p_1^{c_1}\cdot p_2^{c_2}\cdots p_k^{c_k}$

则所有约数的和为$(1+p_1^1+p_1^2+\cdots+p_1^{c_1})\cdots(1+p_k^1+\cdots+p_k^{c_k})$

在求所有约数的和时 朴素做法对于每个$p_k$需要$O(c_k)$
```cpp
map<int,int> p;
for(auto i:p)
{
	int a=i.first,b=i.second;
	ll t=1;
	while(b--) t=(t*a+1)%mod;
	res=res*t%mod;
} 
```
假如指数较大 我们可以采取递归分治的思想缩短为$O(\log c_k)$

对于$1+p^1+p^2+\cdots+p^c$我们可以分奇偶来讨论

若$c$为奇数 则：

$1+p^1+p^2+\cdots+p^c$

$=(1+p^1+\cdots+p^{\frac{c-1}{2}})+(p^{\frac{c+1}{2}}+\cdots+p^c)$

$=(1+p^{\frac{c+1}{2}})(1+p^1+\cdots+p^{\frac{c-1}{2}})$

若$c$为偶数 则：

$1+p^1+p^2+\cdots+p^c$

$=(1+p^1+\cdots+p^{c-1})+p^c$

$\cdots$

$=(1+p^{\frac{c}{2}})(1+p^1+\cdots+p^{\frac{c}{2}-1})+p^c$

```cpp
int sum(int a,int p)
{
	if(p==0) return 1;
	if(p&1) return (1+ksm(a,(p+1)/2))%mod*sum(a,(p-1)/2)%mod;
	else return (1+ksm(a,p/2))*sum(a,p/2-1)%mod+ksm(a,p)%mod;
}
```
另外还有等比数列的逆元解法（待补）
#### 2.4 最大公约数

欧几里得算法：$(a,b)=(b,a\mod\ b)$
```cpp
int gcd(int a,int b){return b?gcd(b,a%b):a;}
```
更相减损算法：$(a,b)=(b,a-b)$

### 3. 欧拉函数
#### 3.1 求出一个数的欧拉函数

$1-N$中与$N$互质的数的个数被称为欧拉函数 记为$\phi(N)$

欧拉函数可以通过容斥原理求出

令$n=p_1^{c_1}\cdot p_2^{c_2}\cdots p_k^{c_k}$

公式为：$\phi(n)=n\cdot\frac{p_1-1}{p_1}\cdot\frac{p_2-1}{p_2}\cdots\frac{p_k-1}{p_k}$
```cpp
int ans=n;
for(int i=2;i<=n/i;i++)
{
	if(n%i==0)
	{
		while(n%i==0) n/=i;
		ans=ans/i*(i-1); 
	}
}
if(n>1) ans=ans/n*(n-1);
```
#### 3.2 筛法求欧拉函数
```cpp
void is_prime()
{
	phi[1]=1;
	for(int i=2;i<=n;i++)
	{
		if(!vis[i]) prime[++num]=i,phi[i]=i-1;
		for(int j=1;prime[j]<=n/i;j++)
		{
			vis[prime[j]*i]=1;
			if(i%prime[j]==0)
			{
				phi[i*prime[j]]=prime[j]*phi[i];
				break;
			}
			else phi[i*prime[j]]=phi[i]*(prime[j]-1);
		}
	}
}
```
### 4. 快速幂

#### 4.1 快速幂
```cpp
int ksm(int a,int b,int p)
{
	int ans=1%p;
	while(b)
	{
		if(b&1) ans=(long long)ans*a%p;
		a=(long long)a*a%p;
		b>>=1;
	}
	return ans;
}
```
#### 4.2 快速幂求逆元

已知欧拉定理:
当$(a,n)=1$时，$a^{\phi(n)}\equiv1(mod\ n)$

特别的，当$n$为质数时 有费马小定理：$a^{n-1}\equiv1(mod\ n)$

所以$a$的逆元就为$a^{n-2}$ 求逆元还有解不定方程的解法（待补）




