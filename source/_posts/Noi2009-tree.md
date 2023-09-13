---
title: NOI2009 二叉查找树
date: 2023-09-02 22:10:18
tags: 
    - 动态规划
    - 区间dp
---

# [NOI2009 二叉查找树](https://www.luogu.com.cn/problem/P1864)

## 题目描述

已知一棵特殊的二叉查找树。根据定义，该二叉查找树中每个结点的数据值都比它左儿子结点的数据值大，而比它右儿子结点的数据值小。

另一方面，这棵查找树中每个结点都有一个权值，每个结点的权值都比它的儿子结点的权值要小。

已知树中所有结点的数据值各不相同；所有结点的权值也各不相同。这时可得出这样一个有趣的结论：如果能够确定树中每个结点的数据值和权值，那么树的形态便可以唯一确定。因为这样的一棵树可以看成是按照权值从小到大顺序插入结点所得到的、按照数据值排序的二叉查找树。

一个结点在树中的深度定义为它到树根的距离加 $1$。因此树的根结点的深度为 $1$。

每个结点除了数据值和权值以外，还有一个访问频度。我们定义一个结点在树中的访问代价为它的访问频度乘以它在树中的深度。整棵树的访问代价定义为所有结点在树中的访问代价之和。

现在给定每个结点的数据值、权值和访问频度，你可以根据需要修改某些结点的权值，但每次修改你会付出 $K$ 的额外修改代价。你可以把结点的权值改为任何实数，但是修改后所有结点的权值必须仍保持互不相同。现在你要解决的问题是，整棵树的访问代价与额外修改代价的和最小是多少？

## 输入格式

输入文件中的第一行为两个正整数 $N,K$。其中 $N$ 表示结点的个数，$K$ 表示每次修改所需的额外修改代价。

接下来的一行为 $N$ 个非负整数，表示每个结点的数据值。

再接下来的一行为 $N$ 个非负整数，表示每个结点的权值。

再接下来的一行为 $N$ 个非负整数，表示每个结点的访问频度。

其中：所有的数据值、权值、访问频度均不超过 $4 \times 10^5$。

## 输出格式

输出文件中仅一行为一个数，即你所能得到的整棵树的访问代价与额外修改代价之和的最小值。

## 分析

首先读完这个题是有个 $\texttt{dp}$ 的冲动的，但是有了这个修改权值从而导致树进行了左旋和右旋使得我们无从下手，~~那么我们熟练的打开题解(bushi~~。我们再看一下这个树的结构，他说数据值小的再上面，那么我们直接从小到大排序一遍先把树的大体结构给整出来，我们继续想他这个修改操作，对于一个点，我们是可以将其点值修改成任意实数，既然这样的话，我们想要修改点直接取小数点后无数位就行，所有我们实际上是可以随便调整大小关系也就是父子关系，那么我们这个不能有相同权值的限制实际上就没有了。接着看我们怎么 $dp$ 啊，感觉还是比较难搞，我们再仔细想想这个树，~~啪的一下很快啊~~，我们发现这个树的中序遍历不管怎么变换时不会改变的，那么就来了，我们可以对这个中序遍历得到的序列进行 $\texttt{dp}$ 了，那么现在局势就明朗多了，我们可以设 $f[i][j][k]$ 为在 $i$ 到 $j$ 这个区间内的数构成了一棵树，且每个点的权值都大于等于 $k$ 的花费的最小的代价，计 $sum[i]$ 为到 $i$ 的频度值的前缀和$。

然后考虑转移，首先我们枚举到的点要满足他的点值是要大于 $k$ 的，因为这是我们这个式子的定义啊。如果当前的点不做调整，那么转移的式子就是 

- $f[i][j][k] = min(f[i][j][k], f[i][t - 1][w_t] + f[t + 1][j][w_t] + (sum[j] - sum[i - 1]))$ 

其中 $t$ 为当前枚举到的点，至于为啥我们要加区间的频度和，待会说完所有的转移方程后一起说，我们如果是修改的话也是非常的好想到就是 

- $f[i][j][k] = min(f[i][j][k], f[i][t - 1][w_t] + f[t + 1][j][w_t] + k + (sum[j] - sum[i - 1]))$ 

然后就是这个加频度的东西了，其实我们模拟下这个转移的过程，我们会发现我们是每个点多递归一层，那么他的频度值就会多贡献一次，然后我们每次是从小区间到大区间，如果之前一个点被计算过那么后面将其深度降低时会再加一次，那么我们的深度贡献实际上就可以被保证了，对于求区间和相信不用我多说，直接上前缀和即可。

对了，初状态直接设 $f[i][i - 1][k] = 0$，剩下的是 $INF$ 即可，答案即为 $f[1][n][1]$

代码实现如下

```cpp
#include <cstring>
#include <iostream>
#include <algorithm>

namespace read_write
{
    template <typename T>
    inline void read(T &x)
    {
        x = 0;
        T f = 1;
        char ch = getchar();
        while (ch > '9' || ch < '0')
        {
            if (ch == '-')
                f = -1;
            ch = getchar();
        }
        while (ch >= '0' && ch <= '9')
            x = x * 10 + ch - '0', ch = getchar();
        x *= f;
        return;
    }

    template <typename T>
    inline void write(T x)
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

    template <typename T>
    T max(T x, T y)
    {
        return x > y ? x : y;
    }

    template <typename T>
    T min(T x, T y)
    {
        return x > y ? y : x;
    }

    template <typename T>
    void swap(T &a, T &b)
    {
        T tem = b;
        b = a;
        a = tem;
    }
}

namespace Solve
{
    using namespace read_write;

    const int N = 120;

    int n, K;
    int a[N], f[N][N][N];
    int sum[N];

    struct Node
    {
        int val, w, fre;

        bool operator < (const Node & a) const {return val < a.val; }
    } node[N];

    void solve()
    {
        read(n), read(K);
        memset(f, 0x3f, sizeof(f));

        for(int i = 1 ; i <= n ; ++ i)
            read(node[i].val);
        for(int i = 1 ; i <= n ; ++ i)
        {
            int x;
            read(x);
            a[i] = node[i].w = x;
        }
        for(int i = 1 ; i <= n ; ++ i)
            read(node[i].fre);
        std :: sort(a + 1, a + n + 1);
        std :: sort(node + 1, node + n + 1);
        
        for(int i = 1 ; i <= n ; ++ i)
        {
            node[i].w = std :: lower_bound(a + 1, a + n + 1, node[i].w) - a;
            sum[i] = sum[i - 1] + node[i].fre;
        }

        for(int i = 1 ; i <= n + 1 ; ++ i)
            for(int k = 1 ; k <= n ; ++ k)
                f[i][i - 1][k] = 0;
        for(int i = n ; i >= 1 ; -- i)
        {
            for(int j = i ; j <= n ; ++ j)
            {
                for(int k = 1 ; k <= n ; ++ k)
                {
                    for(int t = i ; t <= j ; ++ t)
                    {
                        if(node[t].w >= k)
                            f[i][j][k] = min(f[i][j][k], f[i][t - 1][node[t].w] + f[t + 1][j][node[t].w] + sum[j] - sum[i - 1]);
                        f[i][j][k] = min(f[i][j][k], f[i][t - 1][k] + f[t + 1][j][k] + K + sum[j] - sum[i - 1]);
                    }
                }
            }
        }

        write(f[1][n][1]);
    }
}

int main()
{
    Solve :: solve();

    return 0;
}
```