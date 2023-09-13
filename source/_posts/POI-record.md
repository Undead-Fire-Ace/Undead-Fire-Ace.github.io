---
title: POI做题记录
date: 2023-09-05 11:55:55
tags: 学习笔记
---

做了差不多两天的 [POI]{.rainbow} 的题，算是有所收获，真的感觉有点像做中国的 [OI]{.rainbow} 题一样，也算是练了练基础和学习了一些技巧吧，还有四十分钟就要去吃饭了，就写一写总结/ww

# [POI2009 LYZ-Ice Skates](https://www.luogu.com.cn/problem/P3488)

## 题面翻译

滑冰俱乐部初始有 $[1,n]$ 号码溜冰鞋各 $k$ 双，已知 $x$ 号脚的人可以穿 $[x,x+d]$ 号码的鞋子。

现在有 $m$ 次操作，每次两个数 $r,x$，表示来了 $x$ 个 $r$ 号脚的人，$x$ 为负则表示离开。在每次操作之后，你需要判断溜冰鞋是否足够。

## 分析

首先我们看这个区间操作很容易想到用线段树维护，但是他每次给的是一个范围，并不是具体的减某一个位置或是区间全减，这就让我们很头疼，这时候我们就要用到一个东西，叫做 $\mathcal Hall$ (霍尔)定理，我就说了，不会点 -> [这里](https://www.zhihu.com/tardis/zm/art/460373184?source_id=1005) 。那么由这个定理我们就可以知道我们需要满足 

$$
\sum_{i = l}^{r} a_i \leq (r - l + 1 + d) \times k 
$$ 

但是这样看着还是有些复杂，接着进行化简

$$
\begin{alignat*}{6}
\sum_{i = l}^{r} a_i & \leq k \times (r - l + 1) + k \times d\\
\sum_{i = l}^{r} a_i & \leq \sum_{i = l}^{r}k + k \times d\\
\sum_{i = l}^{r} a_i - \sum_{i = l}^{r}k & \leq k \times d\\
\sum_{i = l}^{r} a_i- k & \leq k \times d
\end{alignat*}
$$

然后我们就只需要让上面化简出来的 $\sum_{i = l}^{r} a_i- k$ 小于等于 $k \times d$ 即可，接下来就是用线段树维护最大的字段和即可，初始化的时候每个元素都是 $k$ 因为没有一个被借用过。

+++danger 代码
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

namespace Segment_Tree
{
    using namespace read_write;

    #define int long long

    const int N = 1e6 + 10;

    int n, m, k, d;

    struct Tree
    {
        int sum, lmx, rmx, res;
    } tr[N];

    void pushup(int u)
    {
        tr[u].lmx = max(tr[u << 1].lmx, tr[u << 1].sum + tr[u << 1 | 1].lmx);
        tr[u].rmx = max(tr[u << 1 | 1].rmx, tr[u << 1 | 1].sum + tr[u << 1].rmx);
        tr[u].sum = tr[u << 1].sum + tr[u << 1 | 1].sum;
        tr[u].res = max(tr[u << 1].res, max(tr[u << 1 | 1].res, tr[u << 1].rmx + tr[u << 1 | 1].lmx));
    }

    void build(int u, int l, int r)
    {
        if(l == r)
        {
            tr[u].sum = tr[u].res = -k;;
            return ;
        }
        int mid = l + r >> 1;
        build(u << 1, l, mid), build(u << 1 | 1, mid + 1, r);
        pushup(u);
    }

    void modify(int u, int l, int r, int pos, int w)
    {
        if(l == r)
        {
            tr[u].sum += w;
            tr[u].res += w;
            tr[u].lmx = tr[u].rmx = max(0ll, tr[u].sum);
            return ;
        }
        int mid = l + r >> 1;
        if(pos <= mid)
            modify(u << 1, l, mid, pos, w);
        else
            modify(u << 1 | 1, mid + 1, r, pos, w);
        pushup(u);
    }
}

namespace Solve
{
    using namespace Segment_Tree;

    void solve()
    {
        read(n), read(m), read(k), read(d);
        build(1, 1, n);

        while(m -- )
        {
            int x, y;
            read(x), read(y);
            modify(1, 1, n, x, y);
            if(tr[1].res <= k * d)
                puts("TAK");
            else 
                puts("NIE");
        }
    }
}

signed main()
{
    Solve :: solve();

    return 0;
}
```
+++

# [POI2010 KLO-Blocks](https://www.luogu.com.cn/problem/P3503)

## 题面翻译

给出 $n$ 个正整数 $a_1,a_2,\cdots,a_n$，再给出一个正整数 $k$，现在可以进行如下操作：

- 每次选择一个大于 $k$ 的正整数 $a_i$，将 $a_i$ 减去 $1$ ，选择 $a_{i-1}$ 或 $a_{i+1}$ 中的一个加上 $1$。

经过一定次数的操作后，问最大能够选出多长的一个连续子序列，使得这个子序列的每个数都不小于 $k$。$m$ 组询问。

## 分析

首先我们肯定要找一个区间的平均值是大于等于 $k$ 的，然后我们可以考虑先将整个序列减去一个 $k$ ，然后我们的任务就变成了找到一个区间之和大于等于0，我们先对其求一下前缀和，记为 $s[i]$ ，然后实际上就是要找到一个区间满足 $s[j] - s[i - 1] >= 0$ ，同时要求区间尽可能大，然后就是怎么去求这个区间。我们考虑维护一个单调栈满足 $s[i] > s[j]$ 同时 $j > i$ ，我们会发现如果当前一个点（该点充当这个区间的右端点时）能与栈中的某一个点凑成合法的区间，那么一定是能跟那栈中的那个点上方的所有点都得到一个合法的方案，因此我们直接枚举右端点，如果一个右端点能跟栈顶得到合法的答案，直接将栈顶弹出，因为在他下面的点就都不可以得到更长的区间，然后就看新的点是否合法即可。

+++danger 代码
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

    #define int long long

    const int N = 1e6 + 10;

    int n, m, k;
    int a[N], temp[N];
    int stk[N], top;

    void solve()
    {
        read(n), read(m);
        for(int i = 1 ; i <= n ; ++ i)
            read(a[i]);
        while(m -- )
        {
            int ans = 0;
            top = 0;
            read(k);
            for(int i = 1 ; i <= n ; ++ i)
            {
                temp[i] = temp[i - 1] + a[i] - k;
                if(temp[i] >= 0)
                    ans = max(ans, i);
                if(!top || temp[i] < temp[stk[top]])
                    stk[++ top] = i;
            }

            for(int i = n ; i >= 1 ; -- i)
            {
                while(top && temp[i] - temp[stk[top]] >= 0)
                    ans = max(ans, i - stk[top]), -- top;
            }

            write(ans), putchar(' ');
        }
    }
}

signed main()
{
    Solve :: solve();

    return 0;
}
```
+++

# [POI2010 MOT-Monotonicity 2](https://www.luogu.com.cn/problem/P3506)

## 题目描述

给出N个正整数a[1..N]，再给出K个关系符号（>、<或=）s[1..k]。

选出一个长度为L的子序列（不要求连续），要求这个子序列的第i项和第i+1项的的大小关系为s[(i-1)mod K+1]。

求出L的最大值。

## 分析

首先我们肯定要先把这个字符串给补全。然后我们设 $f[i]$ 为以 $a_i$ 结尾的最长合法序列，那么我们的 $O(n^2)$ 的转移也就不难想了，直接枚举 $i$ ，然后枚举 $j$ ，即从 $i$ 到 $i- 1$ 中选择一个数 $a_j$ 使得将 $a_i$ 接到 $a_j$ 后为一个合法的方案。但是这复杂度肯定是无法接受的，考虑怎么优化。对于我们将 $a_i$ 拼接在 $a_j$ 后时，实际上是确定 $a_i$ 与 $a_j$ 的大小关系的，那么我们在拼接 $a_i$ 的时候，最优的情况一定是 $j$ 是所有满足情况中最大的右边的那个点，然后我们每次拼接完之后就会更改一下这个前缀中的最大值或是后缀中的最大值即可。

+++danger 代码
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

    const int N = 1e6 + 10;

    int n, k, ans, pos;
    int a[N], f[N];
    int t[N], t1[N], t2[N], son[N];
    char str[N];

    int lowbit(int x)
    {
        return x & -x;
    }

    void change(int x)
    {
        for(int i = a[x] ; i <= 1e6 ; i += lowbit(i))
            if(f[x] > f[t1[i]])
                t1[i] = x;
    }

    int query(int x)
    {
        int res = 0;
        for(int i = a[x] - 1 ; i ; i -= lowbit(i))
            if(f[res] < f[t1[i]])
                res = t1[i];
        return res;
    }

    void modify(int x)
    {
        for(int i = a[x] ; i ; i -= lowbit(i))
            if(f[x] > f[t2[i]])
                t2[i] = x;
    }

    int ask(int x)
    {
        int res = 0;
        for(int i = a[x] + 1 ; i <= 1e6 ; i += lowbit(i))
            if(f[res] < f[t2[i]])
                res = t2[i];
        return res;
    }

    void Write(int x)
    {
        if(son[x])
            Write(son[x]);
        write(a[x]), putchar(' ');
    }

    void solve()
    {
        read(n), read(k);
        for(int i = 1 ; i <= n ; ++ i)
            read(a[i]), f[i] = 1;
        for(int i = 1 ; i <= k ; ++ i)
            scanf(" %c", &str[i]);
        for(int i = k + 1 ; i < n ; ++ i)
            str[i] = str[(i - 1) % k + 1];
        
        for(int i = 1 ; i <= n ; ++ i)
        {
            int temp = query(i);
            if(f[i] < f[temp] + 1)
                f[i] = f[temp] + 1, son[i] = temp;
            temp = ask(i);
            if(f[i] < f[temp] + 1)
                f[i] = f[temp] + 1, son[i] = temp;
            if(f[i] < f[temp = t[a[i]]] + 1)
                f[i] = f[temp] + 1, son[i] = temp;
            if(ans < f[i])
                ans = f[i], pos = i;
            if(str[f[i]] == '<')
                change(i);
            if(str[f[i]] == '>')
                modify(i);
            if(str[f[i]] == '=' && f[i] > f[t[a[i]]])
                t[a[i]] = i;
        }

        write(ans), puts("");
        Write(pos);
    }
}

int main()
{
    Solve :: solve();

    return 0;
}
```
+++

# [POI2014 KUR-Couriers](https://www.luogu.com.cn/problem/P3567)

## 题面翻译

给一个长度为 $n$ 的正整数序列 $a$。共有 $m$ 组询问，每次询问一个区间 $[l,r]$ ，是否存在一个数在 $[l,r]$ 中出现的次数严格大于一半。如果存在，输出这个数，否则输出 $0$。

$1 \leq n,m \leq 5 \times 10^5$，$1 \leq a_i \leq n$。

## 分析

首先这个很快就能想到用主席树维护一下就行了，也是最板子的一道题了。

+++danger 代码
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

namespace Segment_Tree
{
    using namespace read_write;

    const int N = 1e7 + 10;

    int n, m;
    int idx, R;
    int a[N], b[N];
    int root[N];

    struct Tree
    {
        int l, r;
        int cnt;
    } tr[N];

    int insert(int p, int x, int l, int r)
    {
        int q = ++ idx;
        tr[q] = tr[p];
        tr[q].cnt ++;
        if(l == r)
            return q;
        int mid = l + r >> 1;
        if(x <= mid)
            tr[q].l = insert(tr[p].l, x, l, mid);
        else 
            tr[q].r = insert(tr[p].r, x, mid + 1, r);
        return q;
    }

    int query(int p, int q, int l, int r, int x)
    {
        if(l == r)
            return l;
        int mid = l + r >> 1;
        if(2 * (tr[tr[q].l].cnt - tr[tr[p].l].cnt) > x)
            return query(tr[p].l, tr[q].l, l, mid, x);
        if(2 * (tr[tr[q].r].cnt - tr[tr[p].r].cnt) > x)
            return query(tr[p].r, tr[q].r, mid + 1, r, x);
        return 0;
    }
}

namespace Solve
{
    using namespace Segment_Tree;

    void solve()
    {
        read(n), read(m);

        for(int i = 1 ; i <= n ; ++ i)
        {
            read(a[i]);
            b[i] = a[i];
        }

        std :: sort(b + 1, b + n + 1);
        R = std :: unique(b + 1, b + n + 1) - b - 1;

        for(int i = 1 ; i <= n ; ++ i)
        {
            int temp = std :: lower_bound(b + 1, b + R + 1, a[i]) - b;
            root[i] = insert(root[i - 1], temp, 1, R);
        }

        while(m -- )
        {
            int l, r;
            read(l), read(r);
            write(b[query(root[l - 1], root[r], 1, R, r - l + 1)]);
            puts("");
        }
    }
}

int main()
{
    Solve :: solve();

    return 0;
}
```
+++

# [POI2014 FAR-FarmCraft](https://www.luogu.com.cn/problem/P3574)

## 题面翻译

在一个叫做比特村的小村庄中，有$n-1$条路连接着这个村庄中的全部$n$个房子。

每两个房子之间都有一条唯一的通路。这些房子的编号为1至$n$。

1号房子属于村庄的管理员比特安萨尔。

为了提升村庄的科技使用水平，$n$台电脑被快递到了比特安萨尔的房子。每个房子都应该有一台电脑，且分发电脑的任务就落在了比特安萨尔的肩上。

比特村的居民一致同意去玩农场物语这个游戏的最新快照版，而且好消息是他们很快就要得到他们最新的高配置电脑了。

比特安萨尔将所有电脑都装在了他的卡车上，而且他准备好完成这个艰巨的任务了。

**他的汽油恰好够走每条路两遍。**

在每个房子边，比特安萨尔把电脑贴心的配送给居民，且立即前往下一个房子。（配送过程不花费任何时间）

只要每间房子的居民拿到了他们的新电脑，它们就会立即开始安装农场物语。安装农场物语所用的时间根据居民的科技素养而定。幸运的是，每间房子中居民的科技素养都是已知的。

在比特安萨尔配送完所有电脑后，他会回到他自己的1号房子去安装他自己的农场物语。

用卡车开过每条路的时间恰好是1分钟，而居民开电脑箱的时间可以忽略不计。（因为他们太想玩农场物语了）

请你帮助比特安萨尔算出从开始配送到所有居民都玩上了农场物语的最少时间。

## 分析

这个也比较好看出来是个树上 $\mathcal dp$ ，对于根节点我们是最后一个装，因此我们直接递归左右子树，然后我们考虑先递归一个子树对另一个子树的贡献，那么显然是遍历完一个子树的时间，那么怎么确定子树顺序。

我们计 $f[x]$ 为以 $x$ 为根的子树中的最大值，那么子树的大小乘 2 即为遍历完子树所用的时间，那么转移式子就是 $f[x]=max{(f[x], f[y] + siz[x] \times 2 + 1)}$，现在有两棵子树 $y$ 和 $z$ ，那么倘若先走 $z$ 比先走 $y$ 更优，即 $max{(f[z], f[y] + siz[z] \times 2 + 2)} \leq max{(f[y], f[z] + siz[y] \times 2 + 2)}$ 。拆掉括号最终即为 $siz[z] \times 2 - f[z] \leq siz[y] \times 2 - f[y]$ 。然后按照这个排一下所有子树的序，按照顺序遍历即可。

+++danger 代码
```cpp
#include <vector>
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

    #define int long long

    const int N = 1e6 + 10;

    int n;
    int c[N];
    int f[N], g[N];
    std :: vector<int> v[N];

    bool cmp(int x, int y)
    {
        return g[x] - f[x] > g[y] - f[y];
    }

    void dfs(int u, int fath)
    {
        for(auto i: v[u])
        {
            if(i == fath)
                continue;
            dfs(i, u);
        }

        std :: sort(v[u].begin(), v[u].end(), cmp);
        if(u != 1)
            g[u] = c[u];
        for(auto i: v[u])
        {
            if(i == fath)
                continue;
            g[u] = max(g[u], g[i] + f[u] + 1);
            f[u] = f[u] + f[i] + 2;
        }

        return ;
    }

    void solve()
    {
        read(n);
        for(int i = 1 ; i <= n ; ++ i)
            read(c[i]);
        for(int i = 1 ; i < n ; ++ i)
        {
            int a, b;
            read(a), read(b);
            v[a].push_back(b), v[b].push_back(a);
        }
        dfs(1, 0);

        write(max(g[1], f[1] + c[1]));
    }
}

signed main()
{
    Solve :: solve();

    return 0;
}
```
+++

# [POI2014 ZAL-Freight](https://www.luogu.com.cn/problem/P3580)

## 题面翻译

Upper Bytown和Lower Bytown的火车站通过一条轨道铁路连接。

沿任何一个方向在它们之间行驶都需要s分钟。

但是，离开车站的火车必须至少间隔一分钟。

而且，在任何时候，铁路上的所有列车都必须朝同一方向行驶。

根据我们的时间表，前往下拜镇的n列货运列车将通过上拜镇。 他们将在下拜敦装载货物，然后返回上拜敦。 为简单起见，我们假设将货物装载到火车上几乎不需要时间。

我们将确定最后一班火车返回Upper Bytown的最短时间。

**两个车站发车都必须至少间隔一分钟。**

## 分析

对于这个题，我们考虑对时间的影响是每一波发车时放了多少车，那么我们可以设 $f[i]$ 为以 $i$ 结尾的最后一波车最早能到的时间，枚举 $j$ ，转移即为 $f[i] = min{(max{(t_i, f[j] + i + j - 1)} + 2 \times s + i - j - 1)}$ 。其中 $t_i$ 为 $i$ 的发车时间，直接转移肯定是 $O(n^2)$ 的，考虑怎么优化。

当 $t_i > f[j] + i - j - 1$ 时，那么转移方程为 $f[i] = min{(f[i], t_i + 2 \times s + i - j - 1)}$ ，将 $\mathcal min$ 拆开，若当 $k$ 比 $j$ 更优，需要满足

$$
\begin{alignat*}{6}
t_i+2 \times s + i - j - 1 &> t_i + 2 \times s + i - k - 1 \\
-j&>-k\\
\end{alignat*}
$$

当 $t_i > f[j] + i - j - 1$ 时，那么转移方程为 $f[i] = min{(f[j] + 2 \times (s + i - j - 1))}$ ，同样我们拆开

$$
\begin{alignat*}{6}
f[j] + 2 \times (s + i - j - 1) &> f[k] + 2 \times (s + i - j - 1)\\
f[j] - 2 \times j &> f[k] - 2 \times k
\end{alignat*}
$$

所以我们直接拿单调队列维护一下就行了

+++danger 代码
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

    #define int long long

    const int N = 1e6 + 10, INF = 0x7f7f7f7f;

    int n, S;
    int q[N], hh, tt;
    int t[N], f[N];

    void solve()
    {
        read(n), read(S);

        for(int i = 1 ; i <= n ; ++ i)
            read(t[i]), f[i] = INF;
        std :: sort(t + 1, t + n + 1);
        t[0] = -1;
        for(int i = 1 ; i <= n ; ++ i)
            t[i] = max(t[i], t[i - 1] + 1);
        for(int i = 1 ; i <= n ; ++ i)
        {
            while(hh < tt && f[q[hh]] - q[hh] < t[i] - i + 1)
                ++ hh;
            f[i] = min(f[i], f[q[hh]] + 2 * (S + i - q[hh] - 1));
            f[i] = min(f[i], t[i] + 2 * S + i - q[hh - 1] - 1);
            while(hh < tt && f[q[tt]] - 2 * q[tt] > f[i] - 2 * i)
                -- tt;
            q[++ tt] = i;
        }

        write(f[n]);
    }
}

signed main()
{
    Solve :: solve();

    return 0;
}
```
+++

# [POI2004 PRZ](https://www.luogu.com.cn/problem/P5911)

## 题目背景

一只队伍在爬山时碰到了雪崩，他们在逃跑时遇到了一座桥，他们要尽快的过桥。

## 题目描述

桥已经很旧了, 所以它不能承受太重的东西。任何时候队伍在桥上的人都不能超过一定的限制。 所以这只队伍过桥时只能分批过，当一组全部过去时，下一组才能接着过。队伍里每个人过桥都需要特定的时间，当一批队员过桥时时间应该算走得最慢的那一个，每个人也有特定的重量，我们想知道如何分批过桥能使总时间最少。

## 提示

对于 $100\%$ 的数据，$100\le W \le400$ ，$1\le n\le 16$，$1\le t\le50$，$10\le w\le100$。

## 分析

算是一种比较常见的状压的题，对于这个我们可以将所有状态所要用的时间和重量预处理出来，那么我们枚举所有状态，然后再枚举这个状态的所有子集，判断一下重量是否合法即可。

+++danger 代码
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

    const int N = 1e6 + 10;

    int lim, n, tot;
    int t[N], w[N], dp[N];
    int sum_t[N], sum_w[N];

    void solve()
    {
        memset(dp, 0x3f, sizeof(dp));
        read(lim), read(n);

        tot = (1 << n) - 1;

        for(int i = 1 ; i <= n ; ++ i)
            read(t[i]), read(w[i]);
        for(int i = 0 ; i <= tot ; ++ i)
        {
            for(int j = 1 ; j <= n ; ++ j)
            {
                if(i & (1 << (j - 1)))
                {
                    sum_t[i] = max(sum_t[i], t[j]);
                    sum_w[i] += w[j];
                }
            }
        }

        dp[0] = 0;
        for(int i = 0 ; i <= tot ; ++ i)
        {  
            for(int j = i ; ; j = i & (j - 1))
            {
                if(sum_w[i ^ j] <= lim)
                    dp[i] = min(dp[i], dp[j] + sum_t[i ^ j]);//, std :: cout << dp[i] << " " << dp[j] << " zxc" << std :: endl;
                if(!j)
                    break;
            }
        }

        write(dp[tot]);
    }
}

int main()
{
    Solve :: solve();

    return 0;
}
```
+++


# [終わり]{.rainbow}

以后有可能会更新吧，谁知道呢