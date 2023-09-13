---
title: ARC089E GraphXY题解
date: 2023-08-22 11:21:58
tags: 
    - 题解
    - 构造
    - 思维题
---
[题目传送门](https://www.luogu.com.cn/problem/AT_arc089_c)

我们先看题意，发现他让求的起点和终点之间的最短路径一直在变，那必然在最短路上会有 $X$ 和 $Y$ 。

我们考虑如何得到最短距离，我们拉一条边权全是 $X$ 的链和一条全是 $Y$ 的链，然后我们可以在这两条链之间连边，因为我们拉的链是从起点直接连到终点的，因此我们设走了 $a$ 个 $X$ 和 $b$ 个 $Y$，我们两条链间连的边就设 $f_{a,b}$ , 那么 $d[x][y] = min(x * i + y * j + f_{i,j})$。然后移一个项就会发现 $f[i][j] = max(d[x][y] - i * x - j * y)$， 那么我们实际上就是可以通过枚举求出来的。然后我们又看一下这个数据范围，发现并不是很大，而且最小距离是小于等于 100 的，那么 $X$ 和 $Y$ 的总数是小于等于 100 。但是我们为了避免枚举出的 $f[i][j]$ 不一定是符合要求的，但是我们再判断一下就彳亍。

具体的细节可以看下面的代码。（温馨提示：由于我们特殊的构造方法，肯定好多是和样例不一样的，不用担心，直接提交看对错）

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>

using namespace std;

const int N = 105, INF = 0x3f3f3f3f;

int n, m;
int f[N][N], d[N][N];

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

int main()
{
    n = read(), m = read();

    for(register int i = 1 ; i <= n ; i ++ )
    {
        for(register int j = 1 ; j <= m ; j ++ )
            d[i][j] = read();
    }

    for(register int i = 0 ; i <= 100 ; i ++ )
    {
        for(register int j = 0 ; j <= 100 ; j ++ )
        {
            for(register int x = 1 ; x <= n ; x ++ )
            {
                for(register int y = 1 ; y <= m ; y ++ )
                {
                    f[i][j] = max(f[i][j], d[x][y] - i * x - j * y);
                }
            }
        }
    }

    int pre;

    for(register int x = 1 ; x <= n ; x ++ )
    {
        for(register int y = 1 ; y <= m ; y ++ )
        {
            pre = INF;

            for(register int i = 0 ; i <= 100 ; i ++ )
            {
                for(register int j = 0 ; j <= 100 ; j ++ )
                    pre = min(pre, f[i][j] + i * x + j * y);
            }

            if(pre != d[x][y])
            {
                puts("Impossible");
                exit(0);
            }
        }
    }

    puts("Possible");
    puts("202 10401");   //算上链上的点和起点还有终点总共是202个点，链和链之间的边是10401条

    for(register int i = 1 ; i <= 100 ; i ++ )   //拉X那条链
    {
        write(i), putchar(' '), write(i + 1), puts(" X");
    }

    for(register int i = 102 ; i < 202 ; i ++ )   //拉Y那条链
    {
        write(i), putchar(' '), write(i + 1), puts(" Y");
    }

    for(register int i = 0 ; i <= 100 ; i ++ )
    {
        for(register int j = 0 ; j <= 100 ; j ++ )   //连两条链之间的边
        {
            write(i + 1), putchar(' '), write(202 - j), putchar(' '), write(f[i][j]), puts("");
        }
    }

    puts("1 202");

    return 0;
}
```

然后一道让人眼前一黑的思维题就结束了。*★,°*:.☆(￣▽￣)/$:*.°★* 。