---
title: HEOI2016/TJOI2016排序
date: 2023-08-21 19:53:40
tags: 
    - 题解
    - 数据结构
    - 线段树
---
# [[HEOI2016/TJOI2016]排序](https://www.luogu.com.cn/problem/P2824)

## 题目描述

在 $2016$ 年，佳媛姐姐喜欢上了数字序列。因而她经常研究关于序列的一些奇奇怪怪的问题，现在她在研究一个难题，需要你来帮助她。  

这个难题是这样子的：给出一个 $1$ 到 $n$ 的排列，现在对这个排列序列进行 $m$ 次局部排序，排序分为两种：  

- `0 l r` 表示将区间 $[l,r]$ 的数字升序排序  
- `1 l r` 表示将区间 $[l,r]$ 的数字降序排序  

注意，这里是对**下标**在区间 $[l,r]$ 内的数排序。  
最后询问第 $q$ 位置上的数字。

## 分析：

这个题显然是让我们写一种数据结构来维护区间排序，巨佬们可以用线段树分裂直接过，本蒟蒻也不会拿线段树分裂写，因此我就讲一种用普通的线段树的做法。

首先，我们考虑这个时间复杂度的瓶颈就是这个排序操作，如果用正常的排序的话，每次的复杂度都是 $O(n logn)$ ,这个复杂度显然对于这道题的数据范围是不可以接受的。

那么这道题妙的地方就要来了！！！既然这个排序不够高效，我们就让它变得高效。 ~~众所周知~~，线段树维护 01 串的时间复杂度是 $O(logn)$。那么聪明的同学看到这就差不多把这道题切掉了。那么像我这样的蒟蒻还是一脸疑惑，这怎么变成 01 串的问题呢？？？

我们可以先假设经过所有操作后的第 $pos$ 个位置的数的值为 $k$ ，我们可以将整个序列中大于 $k$ 变成是1，小于 $k$ 的数变成0。然后就挨个进行排序，（现在先不讲排序，下面会讲），最后如果第 $pos$ 个数为0，那么就证明 $k$ 小了。因此我们可以二分一下 $k$，根据每次的 $pos$ 的值缩小边界即可。

最后再说一下排序，其实非常简单，就是因为是01串，那么我们只用维护一些区间内的1的数量，然后升序就将1全放在区间的右边。这个操作可以用区间修改就可以实现。

下面是代码实现

```cpp
#include <iostream>
#include <algorithm>
#include <queue>
#include <cstring>

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

const int N = 1e6 + 10;

int n, m;
int ask, L, R;
int w[N], ans;

struct Tag
{
    int col;
    bool flag;
};

struct question
{
    int op, l, r;
} q[N];

struct Segmen_Tree
{
    struct Tree
    {
        int l, r, val;
        Tag tag;
    } tr[N << 1];

    inline void pushup(int u)
    {
        tr[u].val = tr[u << 1].val + tr[u << 1 | 1].val;
    }

    inline void build(int u, int l, int r, int pos)
    {
        tr[u].tag.flag = false;
        tr[u].l = l, tr[u].r = r;
        if (l == r)
        {
            tr[u].val = (w[l] >= pos);
            return;
        }
        int mid = l + r >> 1;
        build(u << 1, l, mid, pos), build(u << 1 | 1, mid + 1, r, pos);
        pushup(u);
    }

    inline void pushdown(int u)
    {
        if (tr[u].tag.flag)
        {
            tr[u << 1].val = (tr[u << 1].r - tr[u << 1].l + 1) * tr[u].tag.col;
            tr[u << 1 | 1].val = (tr[u << 1 | 1].r - tr[u << 1 | 1].l + 1) * tr[u].tag.col;
            tr[u << 1].tag.flag = tr[u << 1 | 1].tag.flag = true;
            tr[u << 1].tag.col = tr[u << 1 | 1].tag.col = tr[u].tag.col;
            tr[u].tag.flag = false;
        }
    }

    inline void modify(int u, int l, int r, int k)
    {
        if (l <= tr[u].l && r >= tr[u].r)
        {
            tr[u].val = (tr[u].r - tr[u].l + 1) * k;
            tr[u].tag.flag = true;
            tr[u].tag.col = k;
            return;
        }

        pushdown(u);
        int mid = tr[u].l + tr[u].r >> 1;
        if (l <= mid)
            modify(u << 1, l, r, k);
        if (r > mid)
            modify(u << 1 | 1, l, r, k);
        pushup(u);
    }

    inline int query(int u, int l, int r)
    {
        if (l <= tr[u].l && r >= tr[u].r)
            return tr[u].val;
        pushdown(u);

        int mid = tr[u].l + tr[u].r >> 1, res = 0;
        if (l <= mid)
            res += query(u << 1, l, r);
        if (r > mid)
            res += query(u << 1 | 1, l, r);
        return res;
    }

    inline bool check(int x)
    {
        build(1, 1, n, x);
        int sum;
        for (register int i = 1; i <= m; i++)
        {
            sum = query(1, q[i].l, q[i].r);
            if (q[i].op)
            {
                modify(1, q[i].l, q[i].l + sum - 1, 1);
                modify(1, q[i].l + sum, q[i].r, 0);
            }
            else
            {
                sum = q[i].r - q[i].l + 1 - sum;
                modify(1, q[i].l, q[i].l + sum - 1, 0);
                modify(1, q[i].l + sum, q[i].r, 1);
            }
        }
        return query(1, ask, ask);
    }
} god;

int main()
{
    n = read();
    m = read();

    for (register int i = 1; i <= n; i++)
        w[i] = read();
    for (register int i = 1; i <= m; i++)
        q[i].op = read(), q[i].l = read(), q[i].r = read();
    ask = read();

    L = 1, R = n;
    while (L <= R)
    {
        int mid = L + R >> 1;
        // cout<<mid<<endl;
        if (god.check(mid))
            ans = mid, L = mid + 1;
        else
            R = mid - 1;
    }

    write(ans);

    return 0;
}
```

完结  *★,°*:.☆(￣▽￣)/$:*.°★* 。