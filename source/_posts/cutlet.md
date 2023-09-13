---
title: CF939F Cutlet 题解
date: 2023-08-22 11:28:26
tags: 
    - 题解
    - 动态规划
---
# [Cutlet](https://www.luogu.com.cn/problem/CF939F)

## 题面翻译

有$2\times n$ 的时间去煎一块两面的肉

给你$k$ 个不相交时间区间$[l_i,r_i]$ 

你可以在这些区间的时间内任意次翻转这块肉

问是否存在合法的方案使得两面恰好都只煎了$n$ 分钟

如果不存在则输出 "Hungry" , 否则第一行输出 "Full", 第二行输出最小次数

$n\le10^5,k\le100$ 

----------------------

## 分析：

首先看题意让我们判断是否有解，然后求最小的次数，我们判断无解之类的我们都可以先求，如果要是求不出来就肯定无解，所以我们现在要想怎么求。

首先他说在一个时间段中我们可以反转任意次，但是由于他让我们求最小的反转次数，因此我们会哦发现，实际上在一个区间中我们最多会翻转2次，因为你再翻实际上是和之前等效的，而且不是最优解。然后我们再接着思考怎么去计算，我们可以设 $f[i][j]$ 为在第 $i$ 时刻，当前烤着的面的背面烤了 $j$ 秒，然后我们考虑怎么转移，首先对于两个相邻的时间段我们可以直接进行复制，因为你没办法进行操作，然后对于两个区间里，明显是从一个区间转移到另外一个区间，然后我们就可以对 $f[i][j]$ 进行一步优化变成了在前 $i$ 个区间里背面烤了 $j$ 秒的翻转的次数，这样我们原本的 $1e10$ 的二维数组就优化成了 $1e7$ ，是可以开的下的。

### 一：当我们在第 $i$ 个区间只对牛排翻转1次

我们的设翻转之前烤了 $k$ 秒, ($k \leq r - l$) , $f[i][j] = max(f[i - 1][r - j - k] + 1)$,
上一轮的背面的总共的时间就是 $j + k$ ，然后因为 $k \leq r - l$，所以 $r - j - k \geq l - j$。

### 二：当我们在第 $j$ 个区间只对牛排翻转2次

还是设烤了 $k$ 次，此时的转移也很好推就是 $f[i][j] = min(f[i -1][j - k] + 2)$

然后我们考虑当 $j$ 增加的时候翻转一次的范围是越来越小的，翻转两次的范围是越来越大的，所以要分开处理,同样我们可以用单调队列维护区间最大值来维护转移过程。

ps:如果你想再压一下空间可以考虑滚动一下，因为每个区间的转移只跟上一次转移有关

下面是代码实现

```cpp
#include <iostream>
#include <queue>

namespace read_write
{
    template <typename T>
    void read(T &x)
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
}

namespace Solve
{
    using namespace read_write;

    const int N = 1e6 + 10, INF = 0x3f3f3f3f;

    int n, k;
    int f[2][N]; // f[i][j]: 在第i时刻，反面煎了j秒，翻面的次数，但是空间开不下，那就滚一下
    int q[N], hh, tt;

    void solve()
    {
        read(n), read(k);

        for(int i = 0 ; i <= N ; ++ i)
            f[0][i] = INF;
        f[0][0] = 0;
        for (int i = 1; i <= k; ++i)
        {
            int l, r;
            read(l), read(r);

            for(int j = 0 ; j <= n ; ++ j)
                f[i & 1][j] = f[!(i & 1)][j];
            hh = 1, tt = 0;

            for(int j = r ; j >= 0 ; -- j)
            {
                while(hh <= tt && q[hh] < l - j)
                    hh ++;
                while(hh <= tt && f[!(i & 1)][q[hh]] > f[!(i & 1)][r - j])
                    tt --;
                q[++ tt] = r - j;
                f[i & 1][j] = min(f[i & 1][j], f[!(i & 1)][q[hh]] + 1);
            }

            hh = 1, tt = 0;
            for(int j = 0 ; j <= r ; ++ j)
            {
                while(hh <= tt && q[hh] < j - r + l)
                    hh ++;
                while(hh <= tt && f[!(i & 1)][q[tt]] > f[!(i & 1)][j])
                    tt --;
                q[++ tt] = j;
                f[i & 1][j] = min(f[i & 1][j], f[!(i & 1)][q[hh]] + 2);
            }
        }

        if(f[k & 1][n] == INF)
            puts("Hungry");
        else
        {
            puts("Full");
            write(f[k & 1][n]);
        }
    }
}

using namespace Solve;

int main()
{
    solve();

    return 0;
}
```
*★,°*:.☆(￣▽￣)/$:*.°★* 。