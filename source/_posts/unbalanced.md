---
title: AGC045B 01 Unbalanced 题解
date: 2023-08-22 11:33:10
tags: 
    - 题解
    - 贪心
    - 思维题
---

# [[AGC045B] 01 Unbalanced](https://www.luogu.com.cn/problem/AT_agc045_b)

## 题意：

给定一个字符串，其中 $?$ 可以替换成 $0$ 或 $1$。让最小化 $0$ 和 $1$ 的数量差。

## 分析：

一般对于最小化最大值的题，往往都可以用二分去做，（~~但是这道题的二分的 check 函数我不会写~~），那么就想一想有没有什么平替，我们可以想到贪心和 dp 也是可以解决类似的问题的，我们可以考虑因为是跟个数有关且只有两个不同的数，那么我们就可以将 $0$ 看成是 $-1$，$1$ 是 $+1$，然后就转换成了最小化最大的累积和与最小的累积和的差值。先可以设，在累计和最大值不超过 $limit$ 时，$f_{limit}$ 为最大累计和的最小值，$g$ 为最小累计和的最大值，那么我们的答案就是求 $\min \left\{limit - f_{limit}\right\}(g\leq limit)$。

然后我们需要去求 $g$，对于求 $g$，我们可以贪心的去求解，在一开始我们先将所以的问号都设置成 $-1$，然后在检查一遍，如果累计和不超过我们的 $limit$ 的话，就将 $-1$ 变成 $1$。

那么这样贪心为什么是正确的呢？

当 $limit$ 增加 $2$ 时，最多只有一个 $?$ 发生变换，所以 $f_{limit+2}\leq f_{limit}+2$，那么 $limit-f_{limit+2}\leq (limit+2)-(f_{limit}+2)$，那么我们最终的答案为 $\min \left\{limit-f_{limit} \right\} \leq \min \left\{ (limit+2)-f_{limit+2} \right\}$。

那么正确性保证了，最后的答案也就是 $\min(g-f_{g},(g+1)-f_{g+1})$。

代码实现起来也是比较容易的，一些细节也在代码中标注了。

```cpp
#include <cstring>
#include <iostream>
#include <algorithm>

namespace read_write   //快读快写可以忽略
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

    const int N = 1e6 + 10;

    std::string a;
    int sum[N];   //记录后缀中1的个数(与0抵消之后的个数)
    int len;

    int calc(int limit)
    {
        int cnt, mn;
        cnt = mn = 0;

        for(int i = 1 ; i <= len ; ++ i)
        {
            if(a[i] == '0')
                cnt --;
            else if(a[i] == '1')
                cnt ++ ;
            else 
            {
                if(cnt + sum[i + 1] + 1 <= limit)
                    cnt ++;
                else 
                    cnt --;
            }

            mn = min(mn, cnt);    //计算f[g]
        }

        return limit - mn;
    }

    void solve()
    {
        std::cin >> a;
        len = a.size();
        a = ' ' + a;   //在字符串前加个空格，调整下标（个人习惯

        for(int i = len ; i >= 1 ; -- i)
            sum[i] = max(0, sum[i + 1] + (a[i] == '1' ? 1 : -1));   //在计算1的个数时也将 '?' 看成是0
        write(min(calc(sum[1]), calc(sum[1] + 1)));   //加1为了防止sum[1]为0时，出些奇怪的错误
    }
}

int main()
{
    Solve::solve();

    return 0;
}
```
