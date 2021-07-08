---
title: Codeforces Round 640 (Div. 4) 全题解
aliases:
- /ACM/CF640div4
featured_image: https://img.songhn.com/img/7.png?imageView2/0/format/webp/q/75|imageslim
excerpt: CF出div4了，我的青春结束了
date: 2020-05-10 01:03:36
tags:
- 模拟
- 构造
- 前缀和
- codeforces
categories:
- ACM
description: CF出div4了，我的青春结束了
summary: CF出div4了，我的青春结束了
enableMathJax: true
katex: true
---

~~大人，时代真的变了~~

第一场div4的话就我的体验来说前面几道比较偏模拟，后面几道比较偏构造思维，~~个人体验还行~~

（注意：由于还未终测所以代码不保证正确性，AB以外的题没写题意简述）

### A. Sum of Round Numbers
#### 题意简述

$T$组数据，给你一个数字$n$，要你用最少的round number表示出$n$，round number意思是最左边不为$0$，其他都为$0$的数字，其实就是$k\cdot 10^x$这样

##### 数据范围

$1\leq t\leq 10^4,1\leq n\leq 10^4$

---

#### 题解

很显然，直接把每位拆开来就行，我的写法是在计算第$k$位数是几的同时维护一下base，意思是现在的数需要乘base

---

#### 代码
```cpp
int T,n;
vector<int> a;
int main()
{
    cin>>T;
    while(T--)
    {
        cin>>n;
        int ans=0;a.clear();
        int base=1;
        while(n)
        {
            if(n%10!=0) 
            {
                a.push_back((n%10)*base);
                ans++;
            }
            n/=10;
            base*=10;
        }
        cout<<ans<<endl;
        for(int i=0;i<a.size();i++) cout<<a[i]<<" ";
        cout<<endl;
    }
    return 0;
}
```

---

### B. Same Parity Summands

#### 题意简述

$T$组数据，给定$n$和$k$，求$n$可不可以被$k$个奇数或者$k$个偶数表示，可以输出YES，否则输出NO

##### 数据范围

$1\leq t\leq 1000,1\leq n\leq 10^9,1\leq k\leq 100$

---

#### 题解

首先$k$个同奇偶性的数其实取多大都不会影响性质，我取了$k-1$个偶数，最后是偶数那就成立，奇数也一样，$k-1$个偶数取多少其实造不成影响，那么我们就尽量往小去取，取$k-1$个$1$或者$k-1$个$2$，再看剩下的数是否满足即可（假如取最小都已经大于$n$则输出NO即可）

---

#### 代码
```cpp
int T;
int n,k;
int main()
{
    cin>>T;
    while(T--)
    {
        cin>>n>>k;
        int temp=k-1;
        if(n-temp<=0) {puts("NO"); continue;}
        if((n-temp)%2==1)
        {
            puts("YES"); 
            for(int i=1;i<k;i++) cout<<1<<" ";
            cout<<n-temp<<endl;
            continue;
        }
        temp=(k-1)*2;
        if(n-temp<=0) {puts("NO"); continue;}
        if((n-temp)%2==0)
        {
            puts("YES"); 
            for(int i=1;i<k;i++) cout<<2<<" ";
            cout<<n-temp<<endl;
            continue;
        }
        puts("NO");
        continue;
    }
    return 0;
}
```

---

### C. K-th Not Divisible by n
#### 题解

每$n$个数会对答案产生$n-1$的贡献，我们先算出大致的区间，即$k$最后落在的区间，前面的每一整段都会多加1，最后加上剩余的即可

我们首先算比$k$大需要多少区间，直接$\lceil \frac{k}{n-1}\rceil$之后减一就是小于等于$k$的最大的区间数量，这每一个区间都会比原来多产生1的贡献，即这一段会变成$(\lceil \frac{k}{n-1}\rceil -1)\cdot n$之后再用$k-\lceil \frac{k}{n-1}\rceil$即剩下来的，这一部分不会多，和原来一样，直接加上即可

另外代码里有一个细节，$\lceil \frac{a}{b}\rceil = \lfloor \frac{a+b-1}{b}\rfloor$这样直接用默认的向下取整就好了

---

#### 代码
```cpp
ll T,n,k;
int main()
{
    cin>>T;
    while(T--)
    {
        cin>>n>>k;
        if(k<=n-1) cout<<k<<endl;
        else
        {
            ll t=1;
            t=(k+n-2)/(n-1);
            t--;
            ll temp=t*n-t;
            ll rest=k-temp;
            cout<<t*n+rest<<endl;
        }
    }
    return 0;
}
```

---

### D. Alice, Bob and Candies
#### 题解

这题就是一个模拟题，按照题意去写就好，我这里是用了类似双指针的方法，维护$l$和$r$分别往左右移，不断while即可，这个模拟题注意细节即可，直接贴一下代码

---

#### 代码

```cpp
const int maxn=1050;
int T,n;
int a[maxn];
int main()
{
    cin>>T;
    while(T--)
    {
        rd(n);
        rep(i,1,n) rd(a[i]);
        int ans=0;
        int l=0,r=n+1;
        int pre=0;
        int sum1,sum2;sum1=sum2=0;
        if(n==1) 
        {
            cout<<1<<" "<<a[1]<<" "<<0<<endl;
            continue;
        }
        if(n==2)
        {
            cout<<2<<" "<<a[1]<<" "<<a[2]<<endl;
            continue;
        }
        while(l<r)
        {
            l++;
            int t1=a[l];
            while(l+1<r&&t1<=pre) l++,t1+=a[l];
            if(t1<=pre||l+1==r)
            {
                ans++;
                sum1+=t1;
                break;
            }
            else sum1+=t1,ans++;
            pre=t1;
            r--;
            int t2=a[r];
            while(r-1>l&&t2<=pre) r--,t2+=a[r];
            if(t2<=pre||r-1==l)
            {
                ans++;
                sum2+=t2;
                break;
            }
            else sum2+=t2,ans++;
            pre=t2;
        }
        cout<<ans<<" "<<sum1<<" "<<sum2<<endl;
    }
    return 0;
}
```

---

### E. Special Elements
#### 题解
这道题我们观察到$n\leq 8000$，好像可以$O(n^2)$但又感觉有点吃紧，但是题面一开始特意强调了一堆卡时间和空间，那感觉就应该是$n^2$了，加点优化应该就行

这题的重点是$1\leq a_i\leq n$，这就意味着一段区间和大于$n$就肯定不会对答案产生贡献，所以我们可以直接用一个数组去当作map(注意这题你用map之类的数据结构会被卡)，先统计前缀和，然后$n^2$枚举区间的时候如果$s_j-s_i\leq n$就记录进map否则不管，最后统计答案即可

---

#### 代码

```cpp
const int maxn=10005;
int T;
int n,a[maxn],s[maxn];
int mp[maxn];
int main()
{
    cin>>T;
    while(T--)
    {
        cin>>n;
        for(int i=0;i<=n;i++) mp[i]=0;
        s[0]=0;
        rep(i,1,n) 
        {
            rd(a[i]);
            s[i]=s[i-1]+a[i];
            if(i>1&&s[i]<=n) mp[s[i]]=1;
        }
        for(int i=1;i<=n;i++)
        {
            for(int j=i+2;j<=n;j++)
                if(s[j]-s[i]<=n) mp[s[j]-s[i]]=1;
        }
        int ans=0;
        rep(i,1,n) if(mp[a[i]]) ans++;
        printf("%d\n",ans);
    }
    return 0;
}
```

---

### F. Binary String Reconstruction
#### 题解

这题比较简单的构造方法应该是，首先$n_2+1$个$1$，之后$n_0+1$个$1$，注意这时候已经产生了一个$n_1$，最后看看$n_1$还差几个交错输出$10$即可

当然因为一开始已经产生了一个$n_1$所以$n_1$为$0$时候要特判一下，因为$n_1$为$0$所以$n_0$和$n_2$不可能同时出现

---

#### 代码

```cpp
const int maxn=505;
int T,a,b,c;
int t[maxn];
int main()
{
    cin>>T;
    while(T--)
    {
        cin>>a>>b>>c;
        if(b==0)
        {
            if(c>0) for(int i=1;i<=c+1;i++) printf("1");
            if(a>0) for(int i=1;i<=a+1;i++) printf("0");
            cout<<endl;
            continue;
        } 
        for(int i=1;i<=c+1;i++) printf("1");
        for(int i=1;i<=a+1;i++) printf("0");
        for(int i=1;i<=b-1;i++) 
        {
            if(i&1) printf("1");
            else printf("0");
        }
        cout<<endl; 
    }
    return 0;
}
```

---

### G. Special Permutation
#### 题解

这道题想到分成奇数偶数然后手玩推一推应该就出来了，我的构造方案是从最大的一个奇数开始往前推一直到1，然后跳到4，然后跳到2，这样后面就可以6，8然后一直到最后，小的数注意特判一下即可

---

#### 代码

```cpp
int T,n;
int main()
{
    cin>>T;
    while(T--)
    {
        cin>>n;
        if(n==2||n==3) puts("-1");
        else if(n==4) puts("3 1 4 2");
        else if(n==5) puts("5 3 1 4 2");
        else
        {
            int t=n;
            if(t%2==0) t--;
            for(int i=t;i>=1;i-=2) cout<<i<<" ";
            cout<<4<<" "<<2<<" "<<6<<" ";
            for(int i=8;i<=n;i+=2) cout<<i<<" ";
            cout<<endl;
        }
    }
    return 0;
}
```