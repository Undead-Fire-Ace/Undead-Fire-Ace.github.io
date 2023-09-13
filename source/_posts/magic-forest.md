---
title: NOI2014 魔法森林
date: 2023-08-22 11:31:39
tags: 
    - 题解
    - 图论
---
[题目传送门](https://www.luogu.com.cn/problem/P2387)

# 简要题意：

给定一些无向边，每条边有 $a$ 和 $b$ 两个权值，要求判断能否到达终点 $n$ ，能到达，就输出路径的最小的值，定义路径的最小值为这条路径上 $a$ 和 $b$ 的最大值之和。

# 分析：

这道题正解好像是LCT，~~但是我不会LCT啊~~。所以就只能用一些自己会的算法了。

首先我们先主观臆断一下，发现如果我们先将 路径按照 $a$ 的值的大小去排个序，然后再挨个加边，我们就每次加边都更新一下。因为题干说了会出现重边和自环，按照我们的思路，我们要保证 $b$ 尽可能小，出现重边很好处理，就保留小的即可，那么如果是出现了环呢？我们实际上是不需要环的，因此我们就需要考虑删去哪一条边呢？因为环中删掉一条边，连通性是不变的，那么就删去 $b$ 值最大的边即可。那么总体的思路就是先将边按照 $a$ 排序，然后从小到大拓展，连边的时候边的权值为 $b$ ， 然后跑一边 $spfa$ ，如果每次更新 $b$ ，最后再和 $ans$ 求一下最小值即可。

聪明的你肯定会发现在处理环的时候，为什么能保证删掉最大的边一定能保证不会影响最优解，因为如果当前保留一个较大的可能会少加一个更大的。

那么这种思路的正确性如何保证 ？？？

我们将 $b$ 最大的那条边删去，连通性不变，那么就将这个删去的边替换成环中的剩下的边，这样会使走这个删去的边的解变的更小，所以并不会影响最优解。

那么我们就可以开新的动态加边，然后跑 $spfa$ 了 ， $spfa$ 比 $LCT$ 好写又好调而且还快。多是一件美事啊！！！

下面是代码，目前是洛谷最优解 (2023年6月5日10点03分)

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>

using namespace std;

const int N = 50005, M = 100005, INF = 0x3f3f3f3f;

int n, m, d[N], hh, tt, q[N], ans = INF;
int head[N], idx;
bool vis[N];

struct Edge    //树的边（最小生成树的结构体）
{
    int u, v, a, b;
    bool operator<(const Edge &z) const
    {
        return a < z.a;
    }
} g[M];

struct E
{
    int next, v, w;
} e[M << 1];

inline void add(int u, int v, int w)
{
    e[++idx] = (E){head[u], v, w};
    head[u] = idx;
}

inline int read()
{
    int x = 0, f = 1;
    char ch = getchar();
    while(ch > '9' || ch < '0')
    {
        if(ch == '-')
            f = -1;
        ch = getchar();
    }
    while(ch >= '0' && ch <= '9')
        x = x * 10 + ch - '0', ch = getchar();
    return x * f;
}

inline void write(int x)
{
    if(x < 0)
    {
        x = -x;
        putchar('-');
    }
    if(x > 9)
        write(x / 10);
    putchar(x % 10 + '0');
}

inline void spfa()
{
    while (hh != tt)
    {
        int u = q[hh++];
        vis[u] = false;
        if (hh == N)   //证明跑完了所有边
            hh = 0;
        for (int i = head[u]; i; i = e[i].next)
        {
            int v = e[i].v;
            if (max(e[i].w, d[u]) < d[v])
            {
                d[v] = max(e[i].w, d[u]);
                if (!vis[v])
                {
                    q[tt++] = v, vis[v] = true;
                    if (tt == N)
                        tt = 0;
                }
            }
        }
    }
}

int main()
{
    memset(d, 0x3f, sizeof (d));
    d[1] = 0;
    n = read(), m = read();

    for (register int i = 1; i <= m; i++)
    {
        g[i].u = read();
        g[i].v = read();
        g[i].a = read();
        g[i].b = read();
    }

    sort(g + 1, g + 1 + m);  //按照a的大小排序

    for (register int i = 1; i <= m; i++)
    {
        add(g[i].u, g[i].v, g[i].b);
        add(g[i].v, g[i].u, g[i].b);
        
        hh = 0, tt = 2;
        q[0] = g[i].u, q[1] = g[i].v;
        vis[q[0]] = vis[q[1]] = true;
        spfa();
        
        if (d[n] != INF)
            ans = min(ans, g[i].a + d[n]);
    }

    if (ans == INF)
        puts("-1");
    else
        write(ans);
    return 0;
}
```