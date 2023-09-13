---
title: AtCoder ABC306E题解
date: 2023-08-22 11:26:36
tags: 
    - 题解
    - 数据结构
---
# [题目传送门](https://atcoder.jp/contests/abc306/tasks/abc306_e)

先看题意发现是让求前k大的数之和，且有修改。我们考虑每次修改对答案有影响，当且仅当修改后的数在整个序列中是前k大，那么我们可以维护两个multiset，分别维护前大k的数和剩下的数。然后我们每次插入时只需判断是哪个区间的数，然后在对应的区间删除或者插入即可。

此外，必须保证一个multiset里的元素的个数都是前k大的数。具体的操作细节可以看下面的代码，比较好理解，也有很多的佬写的权值线段树，也是可以过的。

下面是代码实现
```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>
#include <set>

using namespace std;

inline int read()
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
    return x * f;
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

int n, k, q;
multiset<int> a, b;
long long ans;

void change()
{
    while (a.size() < k)
    {
        auto iy = b.end();
        iy--; // 取出最大值
        a.insert((*iy));
        ans += (*iy);
        b.erase(iy);
    }

    if (a.empty() || b.empty())
        return;
    while (1)
    {
        auto ix = a.begin();
        auto iy = b.end();
        iy--;
        int ex = (*ix);
        int ey = (*iy);
        if (ex >= ey)
            break;
        ans += (ey - ex);
        a.erase(ix);
        b.erase(iy);
        a.insert(ey);
        b.insert(ex);
    }
}

void add(int v)
{
    b.insert(v);
    change();
}

void erase(int v)
{
    auto ix = a.find(v);
    if (ix != a.end())
        ans -= v, a.erase(ix);
    else
        b.erase(b.find(v));
    change();
}

int main()
{
    n = read(), k = read();

    vector<int> x(n, 0);
    for (register int i = 0; i < k; i++)
        a.insert(0);
    for (register int i = k; i < n; i++)
        b.insert(0);
    ans = 0;
    q = read();
    while (q--)
    {
        int pos, y;
        pos = read(), y = read();
        pos--;
        add(y);
        erase(x[pos]);
        x[pos] = y;
        printf("%lld", ans);
        puts("");
    }

    return 0;
}
```