---
title: AT_ap_y题解
date: 2023-08-22 11:20:30
tags: 
    - 题解
    - 动态规划
    - 组合数学
---
[题目传送门](https://www.luogu.com.cn/problem/AT_dp_y)

**题意：**
给一个 $H \times W$ 的网格，每一步只能向右或向下走，给出一些坐标，这些坐标对应的位置不能经过，求从左上角 $(1,1)$ 走到 $(H,W)$ 的方案数，答案对 $10^9 +7$ 取模


从 $(1,1)$ 走到  $(i,j)$ 的方案数为 $C^{i - 1} _ {i + j - 2}$ ，どうしてですか？因为我们总共应该是走 $i - j + 2$ 步，然后得向下走 $i - 1$ 步，因此就能到 $(i,j)$ 。

然后我们就加上障碍物就行了，经过障碍物 $(x,y)$ 的方案数为从 $(1,1)$ 到 $(x,y)$ 的方案数与 $(x,y)$ 到 $(h,w)$ 的方案数之积。这是一个障碍物的做法，多个障碍物的话都这么做就会复杂度极高。因此需要选择其他的方式求这玩意。

根据上面推得的求一点到另外一点的公式，设第 $j$ 个障碍物在第 $i$ 个障碍物之前，那么从 $j$ 到 $i$ 的方案数为 $C_{xi - xj + yi - yj}^{xi - xj}$ 。然后根据乘法原理，我们走到一个障碍物的方案数应该将在它之前的障碍物的方案数减去。そう，最终的转移方程为 $dp[i]= {C_{xi + yi - 2}^{xi - 1}} - {\sum^{i - 1}_{j = 1} dp[j]} \times {C_{xi - xj + yi - yj}^{xi - xj}}$

我们计 $dp(i)$ 为 $(1,1)$到第 $i$ 个障碍物 $(xi,yi)$ ，中间不经过其他障碍的方案数，如果我们然终点为第 $n+ 1$ 个障碍物，答案自然就是 $dp[n + 1]$。

```cpp
#include<cstring>
#include<iostream>
#include<algorithm>

using namespace std;

#define int long long

const int N = 1e6 + 10, mod = 1e9 + 7;

int n, h, w;
int fac[N], inv[N];
int dp[N];

struct pos
{
    int x, y;
}a[N];

inline void read(int &a)
{
    int x = 0,f = 1;
    char ch = getchar();
    while(ch > '9' || ch < '0')
    {
        if(ch == '-')
            f = -1;
        ch = getchar();
    }

    while(ch >= '0' && ch <= '9')
        x = x * 10 + ch - '0',ch = getchar();
    a = x * f;
}

inline int qmi(int a,int k)
{
    int res = 1;

    while(k)
    {
        if(k & 1)
            res = res * a % mod;
        a = a * a % mod;
        k >>= 1;
    }

    return res;
}

bool cmp(pos b, pos c)   
{   //按照横坐标排序,因为发现我们的式子在横坐标排序后
    //如果前面的那个点的纵坐标小于后面的点实际上是没有影响的,而且在后面进行取出点的时候也是会检查是否为合法点
    if(b.x == c.x)
        return b.y < c.y;
    return b.x < c.x;
}

inline int C(int a,int b)
{
    return fac[a] * inv[a - b] % mod * inv[b] % mod;
}

signed main()
{
    read(h), read(w), read(n);

    for(register int i = 1 ; i <= n ; i ++ )
    {
        int x, y;
        read(x), read(y);
        a[i].x = x - 1, a[i].y = y -1;
    }

    a[++ n] = (pos) {h - 1, w - 1};
    sort(a + 1, a + n + 1, cmp);
    
    fac[0] = inv[0] = 1;

    for(register int i = 1 ; i <= 3e5 + 10 ; i ++ )
    {
        fac[i] = fac[i - 1] * i % mod;
        inv[i] = qmi(fac[i], mod - 2);
    }
    
    for(register int i = 1 ; i <= n ; i ++ )
    {
        int x = a[i].x ,y = a[i].y;
        dp[i] = C(x + y, x); 
        
        for(register int j = 1 ; j < i ; j ++ )
        {
            if(a[j].y <= y && a[j].x <= x)
            {
                int xx = x - a[j].x;
                int yy = y - a[j].y;
                //cout<<xx<<" "<<yy<<endl;
                dp[i] = (dp[i] - dp[j] * C(xx + yy, xx) % mod + mod) % mod;
                //cout<<dp[i]<<endl;
            }
        }
    }
    
    printf("%lld", dp[n] % mod);

    return 0;
}
```