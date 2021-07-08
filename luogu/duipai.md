---
title: 对拍
tags:
- 对拍
categories:
- OI
date: 2019-07-05 09:00:00
aliases:
- /OI/duipai
---
## 在对拍时我们需要四个文件
### 1. zj.cpp 
这是复杂度较有优但正确性未知的代码
### 2. baoli.cpp
这是暴力但正确的代码
### 3. random.cpp
这是随机生成数据的代码
### 4. duipai.cpp
这是用于对拍的代码，在对拍时执行它的exe文件即可
<!--more-->
## random基本模板
```cpp
#include<cstdlib>
#include<ctime>
#include<cstdio>
int random(int n)
{
	return (long long)rand()*rand()%n;
}
int main()
{
	freopen("data.in","w",stdout);
	srand((unsigned)time(0));
	//具体内容
	return 0;
}
```
调用time(0)可以返回一个较大数据
函数random返回0-n-1之间的随机数
### 随机生成实例
#### 1.随机生成树
随机生成一个n个结点的树
```cpp
    int n=random(100000)+1;
	printf("%d\n",n);
	for(int i=2;i<=n;i++)
	{
        //从2-n的每个节点i向1-i-1随机连边
		int fa=random(i-1)+1;
		int val=random(100000)+1;
		printf("%d %d %d\n",fa,i,val);
	}
```
### 对拍程序基本模板
```cpp
#include<cstdio>
#include<ctime>
#include<cstdlib>
using namespace std;
int main()
{
	for(int t=1;t<=30;t++)
	{
		system("random.exe");
		double st=clock();
		system("zj.exe");
		double ed=clock();
		system("baoli.exe");
		if(system("fc data.out data.ans"))
		{
			puts("wrong");
			return 0;
		}
		else 
		printf("Accept,测试点#%d,用时%.0lfms\n",t,ed-st);
	}
	
	return 0;
}
```
data.out data.ans分别为两个程序的输出文件，在对拍时点duipai.exe即可对拍，当程序闪退时，data.in即为有问题的数据