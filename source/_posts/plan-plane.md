---
title: 国家集训队 航班安排
date: 2023-08-22 11:30:28
tags: 
    - 题解
    - 网络流
    - 费用流
---
# [P4452 [国家集训队]航班安排](https://www.luogu.com.cn/problem/P4452)

首先我们这道题直接读题就可以知道就是一个裸的求最小费用最大流，然后我们考虑如何建图，因为网络流的难点就是判断网络流和建图。

对于每个飞机场，一开始我们是否能去到，因为去不了的也不会对我们的答案做出贡献，这一步也好判断，我们只需要看一下每个机场要求的S是否小于等于从起点到那个机场的距离即可。

上面的是建立出发时的图，然后我们考虑飞机场之间的转移。设当前飞机场为 $x$ ， 要判断是否连边的飞机场为 $y$ ，这也非常好想，我们只需要判断 $x$ 的 $s$ 加上 $x$ 到 $y$ 的时间是否小于 $y$ 的 $s$ 即可。

最后我们考虑一下一个方案是否能够返回。因为我们一定是结束时间小的往结束时间大的飞，因此只需要判断每个飞机场是否 $t$  加上从这个飞机场到出发点的时间是否小于这一天的终止时间点即可。

下面是代码实现，由于我用的是PD，就只大概知道一下思想就行，你们也没太大的必要学，毕竟普通的 $dinic$ 就可以过了，而且更好调

```cpp
#include <iostream>
#include <algorithm>
#include <cstdio>
#include <cstring>
#include <queue>

using namespace std;

const int N = 210, M = 1e6 + 10, INF = 0x3f3f3f3f;

int n, m, k, T, s, t, s1;
int maxf, minc, idx;
int tim[N][N];
int dist[M], h[M];
int flo[N][N];
int cnt = 1, he[M];
bool st[M];

struct edge
{
    int ne, f, c, v;
} e[M];

struct mypair
{
    int dis, id;
    bool operator<(const mypair &a) const { return dis > a.dis;};
    mypair(int w, int x)
    {
        dis = w, id = x;
    }
};

struct node
{
    int e, v;
} p[M];

struct dot
{
    int a, b, c, s, t;
} q[M];

inline void read(int &a)
{
    int x = 0, f = 1;
    char ch = getchar();
    while (ch > '9' || ch < '0')
    {

        if (ch == '-')
            f = -1;
        ch = getchar();
    }
    while (ch >= '0' && ch <= '9')
        x = x * 10 + ch - '0', ch = getchar();
    a = x * f;
}

inline void write(int x)
{
    if (x < 0)
    {
        x = -x;
        putchar('-');
    }

    if (x > 9)
        write(x / 10);
    putchar(x % 10 + '0');
}

void add(int u, int v, int f, int c)
{
    e[++cnt].v = v, e[cnt].f = f, e[cnt].c = c, e[cnt].ne = he[u], he[u] = cnt;
}

void adds(int u, int v, int w, int x)
{
    add(u, v, w, -x), add(v, u, 0, x);
}

bool dijkstra()
{
    priority_queue<mypair> q;
    for (register int i = 1; i <= t; i++)
        dist[i] = INF;
    memset(st, false, sizeof(st));
    q.push(mypair(0, s));
    dist[s] = 0;

    while (!q.empty())
    {
        int u = q.top().id;
        q.pop();
        if (st[u])
            continue;
        st[u] = true;

        for (register int i = he[u]; i; i = e[i].ne)
        {
            int v = e[i].v, nc = e[i].c + h[u] - h[v];
            if (e[i].f && dist[v] > dist[u] + nc)
            {
                dist[v] = dist[u] + nc;
                p[v].v = u;
                p[v].e = i;

                if (!st[v])
                    q.push(mypair(dist[v], v));
            }
        }
    }
    return dist[t] != INF;
}

void spfa()
{
    queue<int> q;
    memset(h, 0x3f, sizeof(h));
    h[s] = 0;
    q.push(s), st[s] = true;

    while (!q.empty())
    {
        int u = q.front();
        q.pop();
        st[u] = false;

        for (register int i = he[u]; i; i = e[i].ne)
        {
            int v = e[i].v;
            if (h[v] > h[u] + e[i].c && e[i].f)
            {
                h[v] = h[u] + e[i].c;
                if (!st[v])
                {
                    q.push(v);
                    st[v] = true;
                }
            }
        }
    }
}

int main()
{
    read(n), read(m), read(k), read(T);

    s = 2 * m + 1, t = s + 2;

    for (register int i = 1; i <= n; i++)
    {
        for (register int j = 1; j <= n; j++)
            read(tim[i][j]);
    }

    for (register int i = 1; i <= n; i++)
    {
        for (register int j = 1; j <= n; j++)
            read(flo[i][j]);
    }

    add(s, s + 1, k, 0);
    add(s + 1, s, 0, 0);

    for (register int i = 1; i <= m; i++)
    {
        read(q[i].a), read(q[i].b), read(q[i].s), read(q[i].t), read(q[i].c);
        q[i].a++, q[i].b++;

        adds(i, i + m, 1, q[i].c);
        if (tim[1][q[i].a] <= q[i].s)
            adds(s + 1, i, 1, -flo[1][q[i].a]);
        if (q[i].t + tim[q[i].b][1] <= T)
            adds(i + m, t, 1, -flo[q[i].b][1]);
        for (int j = 1; j < i; j++)
        {
            if (q[i].t + tim[q[i].b][q[j].a] <= q[j].s)
                adds(i + m, j, 1, -flo[q[i].b][q[j].a]);
            if (q[j].t + tim[q[j].b][q[i].a] <= q[i].s)
                adds(j + m, i, 1, -flo[q[j].b][q[i].a]);
        }
    }

    spfa();

    while (dijkstra())
    {
        int minf = INF;

        for (register int i = 1; i <= t; i++)
            h[i] += dist[i];
        for (register int i = t; i != s; i = p[i].v)
            minf = min(minf, e[p[i].e].f);
        for (register int i = t; i != s; i = p[i].v)
        {
            e[p[i].e].f -= minf;
            e[p[i].e ^ 1].f += minf;
        }

        maxf += minf;
        minc += minf * h[t];
    }

    write(-minc);

    return 0;
}
```