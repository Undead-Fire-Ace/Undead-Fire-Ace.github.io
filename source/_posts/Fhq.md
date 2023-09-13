---
title: 平衡树全家桶
date: 2023-08-25 21:26:13
tags: 
    - 笔记
    - 数据结构
    - 平衡树
---
平衡树的种类很多，每种的平衡树的功能和效率也不尽相同，平时主要用的平衡树应该就是 $treap$ 类的， $splay$ ，再带上替罪羊和 $pb\_ds$ 也就差不多了，当然还有使用简便的 $set$ （底层实现是红黑树，非常的快）。对于每种平衡树，他们都需要维护自身的平衡来保证复杂度。

# [treap族]{.rainbow}

首先我们先看 $treap$ 类的。

## 有旋treap 

### 维护平衡的方式

根据他的名字我们就知道他是通过旋转来维护平衡的，旋转分成左旋和右旋，同时为了防止特殊构造，我们普遍在写平衡树的时候，每个节点的键值是随机生成的。

### 旋转

我们旋转的目的其实就是希望不改变树本身的结构，只改变每一层深度的节点的数量，以此来保证树高，那么我们的旋转分为左旋和右旋。左旋其实就是将某个点的右儿子给变成整棵树的根。右旋同理。

代码实现如下 

```cpp
void lrotate(int &k) 
{
    int t = r[k];
    r[k] = l[t];   //将儿子的左子树接到父亲的右子树上（因为我们是按照权值维护的平衡树）
    l[t] = k;    //将父亲设为左儿子
    size_[t] = size_[k];    //我们让右儿子继承之前为根的父亲的siz
    pushup(k);   //更新siz
    k = t;
}
```

### 插入

插入操作比较的常规，就直接按照权值递归的左右子树，然后插入完成后，如果不满足堆的性质就通过旋转来维护即可

代码实现如下

```cpp
void insert(int &k, int x) 
{
    if (!k) 
    {
        sz++;
        k = sz;
        size_[k] = w[k] = 1;
        val[k] = x, rnd[k] = rand();
      return;
    }

    size_[k]++;
    if (val[k] == x) 
        w[k]++;
    else if (val[k] < x) 
    {
        insert(r[k], x);
        if (rnd[r[k]] < rnd[k]) lrotate(k);  //插入后不满足堆的性质就旋转
    } 
    else 
    {
        insert(l[k], x);
        if (rnd[l[k]] < rnd[k]) rrotate(k);
    }
}
```

### 删除

删除比插入要稍微复杂一些，有因为删除后涉及到儿子的更换问题，所有我们删去儿子的时候需要判断我们应该让谁当删去的点的父亲的哪个儿子，其实就是一个分类讨论。按照权值维护好结构后，还要通过旋转维护堆的性质。

代码实现如下

```cpp
bool del(int &k, int x)
{
    if (!k)
        return false;
    if (val[k] == x)
    {
        if (w[k] > 1)
        {
            w[k]--;
            size_[k]--;
            return true;
        }
        if (l[k] == 0 || r[k] == 0)
        {
            k = l[k] + r[k];
            return true;
        }
        else if (rnd[l[k]] < rnd[r[k]])
        {
            rrotate(k);
            return del(k, x);
        }
        else
        {
            lrotate(k);
            return del(k, x);
        }
    }
    else if (val[k] < x)
    {
        bool succ = del(r[k], x);
        if (succ)
            size_[k]--;
        return succ;
    }
    else
    {
        bool succ = del(l[k], x);
        if (succ)
            size_[k]--;
        return succ;
    }
}
```

### 查询

#### 查询排名（按值查序）

查询以 $k$ 为根的子树中，值 $x$ 的排名，直接根据值的关系去递归求解即可

代码实现如下

```cpp
int queryrank(int k, int x)
{
    if (!k)
        return 0;
    if (val[k] == x)
        return size_[l[k]] + 1;
    else if (x > val[k])   //递归右子树，同时加上左子树的贡献
        return size_[l[k]] + w[k] + queryrank(r[k], x);
    else
        return queryrank(l[k], x);
}
```

#### 查询值（按序查值）

那么这时，我们就不能通过值的关系递归求解，而是根据树的 $siz$ 来递归，代码大体一样，但是要注意递归到右子树时要减去左子树的贡献，因为左子树中已经占据了那么多的排名

代码实现如下

```cpp
int querynum(int k, int x)
{
    if (!k)
        return 0;
    if (x <= size_[l[k]])
        return querynum(l[k], x);
    else if (x > size_[l[k]] + w[k])
        return querynum(r[k], x - size_[l[k]] - w[k]);
    else
        return val[k];
}
```

#### 求前驱（第一个比它小的值）

根据定义我们就直接找，当前点比这个值小就去找右子树，如果右子树中的值大于该值，那么前驱即为当前点，递归求解即可

代码实现如下

```cpp
void querypre(int k, int x)
{
    if (!k)
        return;
    if (val[k] < x)
        ans = k, querypre(r[k], x);
    else
        querypre(l[k], x);
}
```

#### 求后继（第一个比它大的值）

同理，就直接放代码了

```cpp
void querysub(int k, int x)
{
    if (!k)
        return;
    if (val[k] > x)
        ans = k, querysub(l[k], x);
    else
        querysub(r[k], x);
}
```

至此为止无旋 $treap$ 就结束了，功能还是非常全的，常数也不是很大。

## [无旋treap (Fhq treap) !!!]{.rainbow}


这种平衡树是我最喜欢的，他的思路更好理解，常数也是非常的优秀，而且核心操作少，但是仅仅通过两个核心操作就可以实现许多功能。

### 维护平衡的方式

既然他都叫无旋 $treap$ 了，肯定不是通过旋转维护的平衡，而是通过分裂和合并来维护，那么这也就为他可持久化埋下了伏笔。

### [分裂(Split)]{.rainbow}

#### [按照值的大小进行分裂]{.rainbow}

我们传入分裂依据的值 $val$ ，那么我们在执行完分裂的操作的时候是会得到两棵树，一颗树的所有值都小于 $val$ ，另外一棵全都大于等于 $val$ 。

代码实现如下

```cpp
void split_v(int pos, int k, int &l, int &r)  //l 和 r是我们分裂后得到的两个根
{
    if (!pos)
    {
        l = r = 0;
        return ;
    }
    else 
    {
        if (tr[pos].val <= k)
            l = pos, split_v(tr[pos].r, k, tr[pos].r, r);
        else
            r = pos, split_v(tr[pos].l, k, l, tr[pos].l);
    }
    update(pos);
}
```

#### [按照子树的大小进行分裂]{.rainbow}

我们会得到两棵树（前提是整棵树足够大），然后一棵树是等于我们传入的参数 $siz$ ，另外一颗大小随意。

代码实现如下 

```cpp
void split_s(int now, int k, int &x, int &y)
{
    if(!now)
    {
        x = y = 0;
        return;
    }
    if(tr[tr[now].l].siz + 1 <= k)
    {
        x = now;
        split_s(tr[now].r, k - tr[tr[now].l].siz - 1, tr[x].r, y);
        update(x);
    }
    else 
    {
        y = now;
        split_s(tr[now].l, k, y, tr[now].l);
        update(y);
    }
}
```

#### 作用

我们实现了上面两种形式的分裂那么有什么用呢？显然易见按值分裂时用来处理跟值有关的问题，比如求前驱后继啥的，按照子树分裂就是来处理排名啥的，当然这两种形式还有别的功能，这将会在 $merge$ 之后一起说。

### [合并(merge)]{.rainbow}

合并就是将两棵树合并为一棵树，我们依旧要考虑树的平衡问题，但是我们会发现我们的树合并之后时保证平衡的，那么将两棵树合并的时候这两棵树也一定时平衡的，所以直接按照根节点的值的大小和堆的性质合并即可。

代码实现如下 

```cpp
int merge(int x, int y)   //这是返回根节点
{
    if(!x || !y)  
        return x | y;
    if(tr[x].key > tr[y].key) 
    {
        tr[x].r = merge(tr[x].r, y);
        update(x);
        return x;
    }

    tr[y].l = merge(x, tr[y].l);
    update(y);
    return y;
}
```

### 功能

我们现在具备了合并和分裂两个核心操作，我们就可以搞更多的事情了。

#### [插入]{.rainbow}

插入操作也是可以分成两种类型的，一种是直接插入某个值，一个是在某个位置插入值，那么我们就直接分别按照对应形式分裂，最后再合并即可

代码实现如下

```cpp
void insert_k(int key)   //插入一个值为key的数
{
    int x, y;
    split_v(rt, key, x, y);
    rt = merge(merge(x, new_node(key)), y);
}

void inset_p(int pos, int key)   //在pos后插入一个数
{
    int x, y;
    split_s(rt, pos, x, y);
    rt = merge(merge(x, new_node(key)), y);
}
```

#### [删除]{.rainbow}

有了插入当然还有删除操作，也是分两种，一个是直接删除某个值，一个是删除某个位置的值，也是根插入一个道理,当然也可以删除一个区间。

代码实现如下

```cpp
void remove(int l, int r)   //删除一个区间
{
    int x, y, z;
    split_s(rt, l - 1, x, y), split_s(y, r - l + 1, y, z);
    rt = merge(x, z);
}

void del(int k)   //删去一个数
{
    int x, y, z;
    split_v(rt, k, x, y), split_v(y, k, y, z);
    if(y)
        y = merge(tr[y].l, tr[y].r);
    rt = merge(merge(x, y), z);
}

void remove_a(int k)   //删去所有值为k的数
{
    int x, y, z;
    split_v(rt, k - 1, x, y), split_v(y, k, y, z);
    merge(x, z);
}
```

#### 其他

还有各种查询，就按照需要什么区间就分裂啥区间，最后再对应操作即可，由于我们可以将一个区间分裂出来，那么我们做区间操作也就更加的方便。

```cpp
int get_k(int k)   //按值查序
{
    int dl, dr;
    split_v(rt, k, dl, dr);
    int rank = dl.siz + 1;
    rt = merge(dl, dr);
    return rank;
}

int get_v(int k)  //按序查值
{
    int p = root;
    while(p)
    {
        if(tr[tr[p].l].siz + 1 == k)
            break;
        else if(tr[tr[p].l].siz >= k)
            p = tr[p].l;
        else 
        {
            k -= tr[tr[p].l].siz + 1;
            p = tr[p].r;
        }
    }
    return tr[p].val
}
```

至此我们的 $treap$ 家族就基本上讲完了


# 替罪羊树

一种比较暴力的平衡树，为啥暴力呢？因为他的思想是不平衡就重构，但是这个不平衡是有忍耐度的，意思就是我们会设定一个系数，如果某个子树的系数超过设定的系数，就将这个子树给拆了重构，以此来维护平衡

我们先构建一个权值平衡树，以及计算根节点的信息（将权值相同的点直接合并到一个点上）

代码实现如下

```cpp
int tot,             // 树中元素总数
    rt,              // 根节点，初值为 0 代表空树
    w[N],            // 点中的数据 / 权值
    l[N], r[N],      // 左右子树
    cnt[N],          // 本数据出现次数（为 0 代表已删除）
    s[N],            // 以本节点为根的子树大小（每个节点记 1 次）
    sz[N],           // 以本节点为根的子树大小（每个节点记 cnt[k] 次）
    del[N];          // 不计已删除节点的子树大小（每个节点记 1 次）

void Cal(int k)   // 重新计算以 k 为根的子树大小
{
    s[k] = s[l[k]] + s[r[k]] + 1;
    sz[k] = sz[l[k]] + sz[r[k]] + cnt[k];
    del[k] = del[l[k]] + del[r[k]] + (cnt[k] != 0);
}
```

## 重构 

首先我们要设定一个系数 $alpha$ ，一般取 $0.7$ 或者 $0.8$。如果一个点的子节点占比大于设定的系数就直接重构。同时我们删除的时候也不全删，打个标记，但是我们会发现，如果我们删除的结点非常多的时候重构的复杂度也是非常高的，所以就删除的节点占比超过系数也重构.

代码实现如下

```cpp
bool can_rebuild(int k)
{
    return wn[k] && (alpha * s[k] <= (double)max(s[l[k]], s[r[k]]) || (double)del[k] <= alpha * s[k]);
}
```

然后就是具体的怎么重构了，分为两步，我们首先中序遍历一遍，然后再二分重建。

```cpp
void Rbu_dfs(int &top, int k)
{
    // 中序遍历展开以 k 节点为根子树
    if (!k)
        return;
    Rbu_dfs(top, l[k]);
    if (wn[k])
        stk[top++] = k;
    // 若当前节点已删除则不保留
    Rbu_dfs(top, r[k]);
}

int Rbu_Build(int l, int r)   //重建
{
    // 将 stk[] 数组内 [l, r) 区间重建成树，返回根节点
    int mid = l + r >> 1; // 选取中间为根使其平衡
    if (l >= r)
        return 0;
    lc[stk[mid]] = Rbu_Build(l, mid);
    rc[stk[mid]] = Rbu_Build(mid + 1, r); // 建左右子树
    Calc(stk[mid]);
    return stk[mid];
}

void Rbu(int &k)
{
    // 重构节点 k 的全过程
    int top = 0;
    Rbu_dfs(top, k);
    k = Rbu_Build(0, top);
}
```

## 插入 & 删除

跟普通的线段树同理，即到了空节点直接新建，然后再 $cnt++$ 就行。删除同理，区别是到了空节点不操作，然后 $cnt--$ ，插入或者是删除后，如果有需要重构的点就重构

代码实现如下

```cpp
void insert(int &k, int p)
{
    // 在以 k 为根的子树内添加权值为 p 节点
    if (!k)
    {
        k = ++cnt;
        if (!rt)
            rt = 1;
        w[k] = p;
        l[k] = rc[k] = 0;
        cnt[k] = s[k] = sz[k] = sd[k] = 1;
    }
    else
    {
        if (w[k] == p)
            cnt[k]++;
        else if (w[k] < p)
            insert(rc[k], p);
        else
            insert(l[k], p);
        Calc(k);
        if (can_rebuild(k))
            Rbu(k);
    }
}

void del(int &k, int p)
{
    // 从以 k 为根子树移除权值为 p 节点
    if (!k)
        return;
    else
    {
        if (w[k] == p)
        {
            if (cnt[k])
                cnt[k]--;
        }
        else
        {
            if (w[k] < p)
                del(rc[k], p);
            else
                del(l[k], p);
        }
        Calc(k);
        if (can_rebuild(k))
            Rbu(k);
    }
}
```

## 其他

求前驱后继，排名，值之类的操作跟普通平衡树一样，这里就直接放代码了

```cpp
int Upper_bound(int k, int p)
{
    // 在以 k 为根子树中，大于 p 的最小数的名次
    if (!k)
        return 1;
    else if (w[k] == p && cnt[k])
        return sz[l[k]] + 1 + cnt[k];
    else if (p < w[k])
        return Upper_bound(l[k], p);
    else
        return sz[l[k]] + cnt[k] + Upper_bound(r[k], p);
}


int Lower_bound(int k, int p)    //查某个数的排名直接 lower_boun(rt, x) + 1即可
{
  if (!k)
    return 0;
  else if (w[k] == p && cnt[k])
    return sz[l[k]];
  else if (w[k] < p)
    return sz[l[k]] + cnt[k] + Lower_bound(r[k], p);
  else
    return Lower_bound(lc[k], p);
}

int query_w_by_pos(int k, int p)
{
    // 以 k 为根的子树中，名次为 p 的权值
    if (!k)
        return 0;
    else if (sz[l[k]] < p && p <= sz[l[k]] + cnt[k])
        return w[k];
    else if (sz[l[k]] + cnt[k] < p)
        return query_w_by_pos(rc[k], p - sz[l[k]] - cnt[k]);
    else
        return query_w_by_pos(l[k], p);
}
```

# Splay

最后就是 $splay$ 了，为啥把他放最后？因为个人觉得不是很好理解，而且他还有一个非常重要的应用就是 $LCT$ ，也就是用来解决动态树问题，可能占的篇幅比较大，所以就放在最后。我们先看一些 $splay$ 的基本操作。

## Rotate

这个操作就是将一个节点的深度变低一层，原来我们是需要左旋和右旋，但是经过不断的迭代，左旋和右旋合并成了一个操作，就是 $Rotate$ 。

代码实现如下

```cpp
inline void rotate(int x)
{
    int y = tr[x].fa;   //取出父节点
    int z = tr[y].fa; 
    int k = tr[y].s[1] == x;     //k = 0表示x是y的左儿子；k = 1表示x是y的右儿子

    tr[z].s[tr[z].s[1] == y] = x, tr[x].fa = z;    //将当前点变成z的左儿子或者是右儿子
    tr[y].s[k] = tr[x].s[k ^ 1],tr[tr[x].s[k ^ 1]].fa = y;
    tr[x].s[k ^ 1] = y,tr[y].fa = x;

    pushup(y), pushup(x);
}
```

## Splay

将节点旋转到根，我们为什么要将一个节点旋转到根呢？目的有两个，一个是有时操作需要，另外一个就是根据期望来算，我们对一个点操作后，如果需要经常查询和使用一个数，那么把它旋转到根节点，这样下次访问它就只需查一次就找到了。

代码实现如下

```cpp
inline void splay(int x,int k)      //将点x旋转至点k下面   splay(x,0):将x旋转到根
{
    while(tr[x].p != k)
    {
        int y = tr[x].p, z = tr[y].p;
        if(z != k)
        {
            if((tr[y].s[1] == x) ^ (tr[z].s[1] == y)) //如果不是一条链就正常转
                rotate(x);
            else rotate(y);   //是一条链我们需要将其链的结构改变
        }    
        rotate(x);
    }

    if(!k) root = x;
}
```

## 插入

这个也比较好说，类比 [treap]{.rainbow} ，我们应该能很自然的想到我们可以直接按照权值关系递归插入，实现起来也不难。

代码实现如下

```cpp
struct Tree
{
    int s[2];
    int fa, v;   //fa：父节点  v：编号
    int siz, cnt;   //siz：子树大小  cnt：当前位置的数出现次数

    void init(int _v,int _p)   //初始化函数
    {
        v = _v,p = _p;
        siz = 1, cnt = 1;
    }
}tr[N];

void insert(int v)
{
    int u = root,p = 0;
    while(u) 
        p = u,u = tr[u].s[v > tr[u].v];   //根据值的关系递归左右子树

    u = ++ idx;
    if(p)
        tr[p].s[v > tr[p].v] = u;
    tr[u].init(v,p);

    splay(u,0);    //记得旋转到根，来保证复杂度
}
```

## 查询

### 按值查序

其实说了那么多应该能想到，我们直接递归左右区间，递归到右子树的时候再加上左子树的贡献即可

```cpp
int get_k(int x)
{
    int res = 0, pos = rt;
    while(true)
    {
        if(x < tr[pos].v)
            pos = tr[pos].s[0];
        else 
        {
            res += tr[pos].siz;
            if(x == tr[pos].v)
            {
                splay(pos);
                return res + 1;
            }
            res += tr[u].cnt
            pos = tr[u].s[1];
        }
    }
}
```

### 按序查值

同理就直接放代码了

```cpp
int get_v(int x)
{
    int pos = rt;
    while(true)
    {
        if(tr[pos].s[0] && k <= tr[tr[pos].s[0]].siz)
            pos = tr[u].s[0];
        else 
        {
            k -= tr[pos].cnt + tr[tr[pos].s[0]].siz;
            if(k <= 0)
            {
                splay(pos);
                return tr[pos].v;
            }
            pos = tr[pos].v;
        }
    }
}
```

# [LCT]{.rainbow}

终于到了这个大家伙了，这个东西的函数是真的多啊，但是每个函数的实现感觉还是比较好理解的。首先我们对于一个树，我们的剖分方式其实之前是学过两种的，有重链剖分和长链剖分，这里 [LCT]{.rainbow} 的剖分方式就是实链剖分，剖分后，我们得到的树是由若干的实边和虚边的组成的这也就导致它有一些性质。

+++danger 性质
- 对于这棵树每一条实链，我们是通过一颗 $splay$ 维护的， $splay$ 的中序遍历就是我们所维护的路径
- 然后对于每棵 $splay$ ，我们是通过前驱和后继来维护序列，即我们一条实链中的每个点是只一个父亲和最多一个儿子
- 对于两个实链，他们是通过虚边来连接的，所以是一个 $splay$ 的根的父亲为另一个 $splay$ 中的一个点，同时对于虚边，是父亲不认儿子，但是儿子认父亲，也就是我们常说的认父不认子
+++

然后就是我们的若干操作了。

## [access]{.rainbow}

这个 $access$ 函数的功能就是把从根到 $x$ 的路径都变成实边，同时 $x$ 变成 $splay$ 的根节点。

代码实现如下

```cpp
void access(int x)    //把从根到 x 的路径都变成实边，同时 x 变成 splay 的根节点
{
    int z = x;
    for(int y = 0 ; x ; y = x, x = tr[x].fa)   //一直跳父亲
    {
        splay(x);    //我们先将 x 转到根节点
        tr[x].s[1] = y;    //因为按照中序遍历，x是最后一个节点，因此直接让y变成 x 的后继即可
        pushup(x);
    }
    splay(z);
}
```

## makeroot

即将 $x$ 变成原树的根

代码实现如下

```cpp
void pushrev(int x)
{
    swap(tr[x].s[0], tr[x].s[1]);
    tr[x].rev ^= 1;
}

void makeroot(int x)
{
    access(x);   //建边
    pushrev(x);    //然后翻转路径,因为翻转路径实际上是不会影响遍历的
}
```

## findroot

找到 $x$ 在原树中的根节点，然后将原树的根节点转到 $splay$ 的的根节点

代码实现如下

```cpp
int findroot(int x)    //找到 x 在原树中的根节点，然后将原树的根节点转到splay的的根节点
{
    access(x);
    while(tr[x].s[0])  //建完实边之后，那么他就成为了根节点，此时一直跳前驱就能找到父亲
        pushdown(x), x = tr[x].s[0];   
    splay(x);
    return x;
}
```

## split

这个函数得和 [Fhq]{.rainbow} 中的 [split]{.rainbow} 区分开，这里的 $split$ 是给 $x$ 和 $y$ 之间的点建一条路径，然后根节点为 $y$ 。但是这仅仅是定义，在实际应用上我们通常认为 $split$ 是将 $x$ 到 $y$ 的路径给提取出来，提取出来后这个路径也是一颗 $splay$ ，那么我们得到了一棵这样的 $splay$ 就更方便我们对链上的信息进行维护。

代码实现如下

```cpp
void split(int x, int y)
{
    makeroot(x);  //让 x 变成根
    access(y);   //然后给 y 和根建路径即可，其实就等于跟 x 建边
}
```

## link

这个函数应该是能猜出他是啥功能就是如果 $x$ 和 $y$ 不连通，那么就连一条边

代码实现如下

```cpp
void link(int x, int y)    //如果 x 和 y 不连通，那么就连一条边
{
    makeroot(x);
    if(findroot(y) != x)
        tr[x].fa = y;
}
```

## cut

相当于 $link$ 的逆操作

```cpp
void cut(int x, int y)      //如果 x 和 y 之间有边就删
{
    makeroot(x);
    if(findroot(y) == x && tr[y].fa == x && !tr[y].s[0])
    {
        tr[x].s[1] = tr[y].fa = 0;
        pushup(x);
    }
}
```

## rotate & splay

这里还是说一下吧，由于我们的这两个函数中都涉及到父子关系的改变，但是我们如果在更改过程中涉及到虚边和根节点之类的得特判，代码大体相同

```cpp
void rotate(int x)   
{
    int y = tr[x].fa, z = tr[y].fa;
    int k = tr[y].s[1] == x;
    if(!is_root(y)) 
        tr[z].s[tr[z].s[1] == y] = x;
    tr[x].fa = z;
    tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
    tr[x].s[k ^ 1] = y, tr[y].fa = x;
    pushup(y), pushup(x);
}

void splay(int x)     //正常的splay操作，但是在改变父子关系时要注意虚边的影响
{
    int top = 0, r = x;
    stk[++ top] = r;     //下放懒标记时，因为操作时从下往上做，所以要从下开始释放懒标记
    while(!is_root(r))
        stk[++ top] = r = tr[r].fa;
    while(top)
        pushdown(stk[top --]);
    while(!is_root(x))
    {
        int y = tr[x].fa, z = tr[y].fa;
        if(!is_root(y))
        {
            if((tr[y].s[1] == x) ^ (tr[z].s[1] == y)) 
                rotate(x);
            else 
                rotate(y);
        }
        rotate(x);
    }
}
```

终于！[LCT]{.rainbow} 的所有基本函数就都写完了，其实看着代码，还是比较好理解的，虽然多但是都挺短的，真要背也是比较好背。

# [知識ポイントは終わりました]{.rainbow}

除了 [LCT]{.rainbow} ，大家应该是做了许多的题了，那么这里就讲几道 [LCT]{.rainbow} 的例题来趁热打铁。

# [例题]{.rainbow}

## [SDOI2008 洞穴勘测](https://www.luogu.com.cn/problem/P2147)

### 题目描述

辉辉有一台监测仪器可以实时将通道的每一次改变状况在辉辉手边的终端机上显示：

如果监测到洞穴u和洞穴v之间出现了一条通道，终端机上会显示一条指令 `Connect u v`

如果监测到洞穴u和洞穴v之间的通道被毁，终端机上会显示一条指令 `Destroy u v`

经过长期的艰苦卓绝的手工推算，辉辉发现一个奇怪的现象：无论通道怎么改变，任意时刻任意两个洞穴之间至多只有一条路径。

因而，辉辉坚信这是由于某种本质规律的支配导致的。因而，辉辉更加夜以继日地坚守在终端机之前，试图通过通道的改变情况来研究这条本质规律。 然而，终于有一天，辉辉在堆积成山的演算纸中崩溃了……他把终端机往地面一砸（终端机也足够坚固无法破坏），转而求助于你，说道：“你老兄把这程序写写吧”。

辉辉希望能随时通过终端机发出指令 `Query u v`，向监测仪询问此时洞穴u和洞穴v是否连通。现在你要为他编写程序回答每一次询问。 已知在第一条指令显示之前，JSZX洞穴群中没有任何通道存在。

### 分析

读完题干后我们就会立刻反应过来，这就是一个板子啊，维护树的连通性也算是一个题型吧，这个连边操作和删边操作太经典了，然后对于维护是否联通直接看一下是否同属于一颗 $Splay$ 即可，这个我们可以直接看一下是否时同一个根节点即可

```cpp
namespace LCT
{
    using namespace read_write;

    const int N = 1e6 + 10;

    int stk[N];

    struct Tree
    {
        int s[2], fa;
        int rev;
    } tr[N];

    void pushrev(int x)
    {
        swap(tr[x].s[0], tr[x].s[1]);
        tr[x].rev ^= 1;
    }

    void pushdown(int x)
    {
        if(tr[x].rev)
        {
            pushrev(tr[x].s[0]), pushrev(tr[x].s[1]);
            tr[x].rev = 0;
        }
    }

    bool is_root(int x)
    {
        return (tr[tr[x].fa].s[0] != x && tr[tr[x].fa].s[1] != x);   
    }

    void rotate(int x)
    {
        int y = tr[x].fa, z = tr[y].fa;
        int k = tr[y].s[1] == x;
        if(!is_root(y))
            tr[z].s[tr[z].s[1] == y] = x;
        tr[x].fa = z;
        tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
        tr[x].s[k ^ 1] = y, tr[y].fa = x;
    }

    void splay(int x)
    {
        int top = 0, r = x;
        stk[++ top] = r;
        while(!is_root(r))
            stk[++ top] = r = tr[r].fa;
        while(top)
            pushdown(stk[top --]);
        while(!is_root(x))
        {
            int y = tr[x].fa, z = tr[y].fa;
            if(!is_root(y))
            {
                if((tr[z].s[1] == y) ^ (tr[y].s[1] == x))
                    rotate(x);
                else 
                    rotate(y);
            }
            rotate(x);
        }
    }

    void access(int x)
    {
        int z = x;
        for(int y = 0 ; x ; y = x, x = tr[x].fa)
        {
            splay(x);
            tr[x].s[1] = y;
        }
        splay(z);
    }

    void makeroot(int x)
    {
        access(x);
        pushrev(x);
    }

    int findroot(int x)
    {   
        access(x);
        while(tr[x].s[0])
            pushdown(x), x = tr[x].s[0];
        splay(x);
        return x;
    }

    void link(int x, int y)
    {
        makeroot(x);
        if(findroot(y) != x)
            tr[x].fa = y;
    }

    void cut(int x, int y)
    {
        makeroot(x);
        if(findroot(y) == x && tr[y].fa == x && !tr[y].s[0])
            tr[x].s[1] = tr[y].fa = 0;
    }
}

namespace Solve
{
    using namespace LCT;

    int n, m;
    char op[20];

    void solve()
    {
        read(n), read(m);

        for(int i = 1 ; i <= m ; ++ i)
        {
            int u, v;
            scanf("%s", op);
            read(u), read(v);
            if(*op == 'C')
                link(u, v);
            else if(*op == 'D')
                cut(u, v);
            else 
            {
                if(findroot(u) == findroot(v))
                    puts("Yes");
                else 
                    puts("No");
            }
        }
    }
}
```


## [国家集训队 Tree II](https://www.luogu.com.cn/problem/P1501)

### 题目描述

一棵 $n$ 个点的树，每个点的初始权值为 $1$。  
对于这棵树有 $q$ 个操作，每个操作为以下四种操作之一：

- `+ u v c`：将 $u$ 到 $v$ 的路径上的点的权值都加上自然数 $c$；
- `- u1 v1 u2 v2`：将树中原有的边 $(u_1,v_1)$ 删除，加入一条新边 $(u_2,v_2)$，保证操作完之后仍然是一棵树；
- `* u v c`：将 $u$ 到 $v$ 的路径上的点的权值都乘上自然数 $c$；
- `/ u v`：询问 $u$ 到 $v$ 的路径上的点的权值和，将答案对 $51061$ 取模。

### 输入格式

第一行两个整数 $n,q$。

接下来 $n-1$ 行每行两个正整数 $u,v$，描述这棵树的每条边。

接下来 $q$ 行，每行描述一个操作。

### 输出格式

对于每个询问操作，输出一行一个整数表示答案。

### 分析

首先这个题考察的是 [LCT]{.rainbow} 维护树链信息，其实也比较的板子，而且非常锻炼码力和细节，首先断边和加边的基本操作就不用再说了，我们这里看一下我们如何对一条路径的信息进行维护，我们是有一个 $split$ 函数的，它的作用就是将一段路径给分离出来，然后我们还是知道分离出来后的根的，所以这就非常方便我们进行操作了，分离出来后我们直接进行按照 $splay$ 的维护方式维护即可，注意乘法和加法在下放时候的优先级。

代码如下

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

namespace LCT
{
    using namespace read_write;

    #define int long long

    const int N = 1e6 + 10, mod = 51061;

    int n, q;
    int stk[N];
    char op[2];

    struct Tree
    {
        int s[2], v, siz, fa;
        int sum, add, mul, rev;
    } tr[N];

    void pushup(int x)
    {
        tr[x].siz = tr[tr[x].s[0]].siz + tr[tr[x].s[1]].siz + 1;
        tr[x].sum = tr[tr[x].s[0]].v + tr[tr[x].s[1]].v + tr[x].v % mod;
    }

    void pushrev(int x)
    {
        swap(tr[x].s[0], tr[x].s[1]);
        tr[x].rev ^= 1;
    }

    void pushdown(int x)
    {   
        if(tr[x].mul != 1)
        {
            if(tr[x].s[0])
            {
                tr[tr[x].s[0]].v = tr[tr[x].s[0]].v * tr[x].mul % mod;
                tr[tr[x].s[0]].mul = tr[tr[x].s[0]].mul * tr[x].mul % mod;
                tr[tr[x].s[0]].add = tr[tr[x].s[0]].add * tr[x].mul % mod;
                tr[tr[x].s[0]].sum = tr[tr[x].s[0]].sum * tr[x].mul % mod;
            }

            if(tr[x].s[1])
            {
                tr[tr[x].s[1]].v = tr[tr[x].s[1]].v * tr[x].mul % mod;
                tr[tr[x].s[1]].mul = tr[tr[x].s[1]].mul * tr[x].mul % mod;
                tr[tr[x].s[1]].add = tr[tr[x].s[1]].add * tr[x].mul % mod;
                tr[tr[x].s[1]].sum = tr[tr[x].s[1]].sum * tr[x].mul % mod;
            }

            tr[x].mul = 1;
        }

        if(tr[x].add)
        {
            if(tr[x].s[0])
            {
                tr[tr[x].s[0]].v = tr[tr[x].s[0]].v + tr[x].add % mod;
                tr[tr[x].s[0]].add = tr[tr[x].s[0]].add + tr[x].add % mod;
                tr[tr[x].s[0]].sum = (tr[tr[x].s[0]].sum + tr[tr[x].s[0]].siz * tr[x].add % mod) % mod;
            }

            if(tr[x].s[1])
            {
                tr[tr[x].s[1]].v = tr[tr[x].s[1]].v + tr[x].add % mod;
                tr[tr[x].s[1]].add = tr[tr[x].s[1]].add + tr[x].add % mod;
                tr[tr[x].s[1]].sum = (tr[tr[x].s[1]].sum + tr[tr[x].s[1]].siz * tr[x].add % mod) % mod;
            }

            tr[x].add = 0;
        }

        if(tr[x].rev)
        {
            pushrev(tr[x].s[0]), pushrev(tr[x].s[1]);
            tr[x].rev = 0;
        }
    }

    bool is_root(int x)
    {
        return (tr[tr[x].fa].s[0] != x && tr[tr[x].fa].s[1] != x);
    }

    void rotate(int x)
    {
        int y = tr[x].fa, z = tr[y].fa;
        int k = tr[y].s[1] == x;
        if(!is_root(y))
            tr[z].s[tr[z].s[1] == y] = x;
        tr[x].fa = z;
        tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
        tr[x].s[k ^ 1] = y, tr[y].fa = x;
        pushup(y), pushup(x);
    }

    void splay(int x)
    {
        int top = 0, r = x;
        stk[++ top] = r;
        while(!is_root(r))
            stk[++ top] = r = tr[r].fa;
        while(top)
            pushdown(stk[top --]);
        while(!is_root(x))
        {
            int y = tr[x].fa, z = tr[y].fa;
            if(!is_root(y))
            {
                if((tr[z].s[1] == y) ^ (tr[y].s[1] == x))
                    rotate(x);
                else 
                    rotate(y);
            }
            rotate(x);
        }
    }

    void access(int x)
    {
        int z = x;
        for(int y = 0 ; x ; y = x, x = tr[x].fa)
        {
            splay(x);
            tr[x].s[1] = y;
            pushup(x);
        }
        splay(z);
    }

    void makeroot(int x)
    {
        access(x);
        pushrev(x);
    }

    int findroot(int x)
    {
        access(x);
        while(tr[x].s[0])
            splay(x), x = tr[x].s[0];
        splay(x);
        return x;
    }

    void link(int x, int y)
    {
        makeroot(x);
        if(findroot(y) != x)
            tr[x].fa = y;
    }

    void cut(int x, int y)
    {
        makeroot(x);
        if(findroot(y) == x && tr[y].fa == x && !tr[y].s[0])
        {
            tr[x].s[1] = tr[y].fa = 0;
            pushup(x);
        }
    }

    void split(int x, int y)
    {
        makeroot(x);
        access(y);
    }
}

namespace Solve
{
    using namespace LCT;

    void solve()
    {
        read(n), read(q);

        for(int i = 1 ; i <= n ; ++ i)
            tr[i].v = tr[i].mul = tr[i].siz = 1;
        for(int i = 1 ; i < n ; ++ i)
        {
            int u, v;
            read(u), read(v);
            link(u, v);
        }

        for(int i = 1 ; i <= q ; ++ i)
        {
            scanf("%s", op);
            if(*op == '+')
            {
                int u, v, c;
                read(u), read(v), read(c);
                split(u, v);
                
                tr[v].v = tr[v].v + c % mod;
                tr[v].add = tr[v].add + c % mod;
                tr[v].sum = (tr[v].sum + c * tr[v].siz % mod) % mod;
                pushup(v);
            }
            else if(*op == '-')
            {
                int u1, v1, u2, v2;
                read(u1), read(v1), read(u2), read(v2);
                cut(u1, v1), link(u2, v2);
            }
            else if(*op == '*')
            {
                int u, v, c;
                read(u), read(v), read(c);
                split(u, v);

                tr[v].sum = tr[v].sum * c % mod;
                tr[v].v = tr[v].v * c % mod;
                tr[v].mul = tr[v].mul * c % mod;
                pushup(v);
            }
            else 
            {
                int u, v;
                read(u), read(v);
                split(u, v);

                write(tr[v].sum % mod), puts("");
            }
        }
    }
}

signed main()
{
    // freopen("test.in", "r", stdin);
    // freopen("test.out", "w", stdout);

    Solve :: solve();

    return 0;
}
```

## [最小差值生成树](https://www.luogu.com.cn/problem/P4234)

### 题目描述

给定一个点标号从 $1$ 到 $n$ 的、有 $m$ 条边的无向图，求边权最大值与最小值的差值最小的生成树。图可能存在自环。

### 输入格式

第一行有两个整数，表示图的点数 $n$ 和边数 $m$。

接下来 $m$ 行，每行三个整数 $u, v, w$，表示存在一条连接 $u, v$ 长度为 $w$ 的边。

### 输出格式

输出一行一个整数，表示答案。

-------------------------

### 分析 

这是讲的最后一道例题了，应该至此 [LCT]{.rainbow} 的所有题型应该是都涉及到了，这个考察的是 [LCT]{.rainbow} 维护边权，这里不得不要提一个小技巧了，对于一般的树上问题，如果是静态的，我们处理边权的手段往往都是将边权下放到这条边中深度更深的点上，但是一旦整棵树动了起来，我们维护边权的时候就要将一条边拆成一个点，那么我们只需让该点连接这两个点。对于这个题我们仍可以贪心的求，当我们出现了一条返租边时，我们将最小的边删去加入这个更大的边一定是更优的，因为当我们的最大值一定的时候，最小值更大是一定符合最优解的。

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

namespace LCT
{
    using namespace read_write;

    const int N = 1e6 + 10, INF = 0x3f3f3f3f;

    int n, m, cnt;
    int stk[N];

    struct Tree
    {
        int s[2], id;
        int rev, fa;
    } tr[N];

    void pushup(int u)
    {
        tr[u].id = u;
        if(tr[tr[u].s[0]].id > n && (tr[u].id <= n || tr[u].id > tr[tr[u].s[0]].id))
            tr[u].id = tr[tr[u].s[0]].id;
        if(tr[tr[u].s[1]].id > n && (tr[u].id <= n || tr[u].id > tr[tr[u].s[1]].id))
            tr[u].id = tr[tr[u].s[1]].id;
    }

    void pushrev(int x)
    {
        swap(tr[x].s[0], tr[x].s[1]);
        tr[x].rev ^= 1;
    }

    void pushdown(int x)
    {
        if(tr[x].rev)
        {
            pushrev(tr[x].s[0]), pushrev(tr[x].s[1]);
            tr[x].rev = 0;
        }
    }

    bool is_root(int x)
    {
        return (tr[tr[x].fa].s[0] != x && tr[tr[x].fa].s[1] != x);
    }

    void rotate(int x)
    {
        int y = tr[x].fa, z = tr[y].fa;
        int k = tr[y].s[1] == x;
        if(!is_root(y))
            tr[z].s[tr[z].s[1] == y] = x;
        tr[x].fa = z;
        tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
        tr[x].s[k ^ 1] = y, tr[y].fa = x;
        pushup(y), pushup(x);
    }

    void splay(int x)
    {
        int top = 0, r = x;
        stk[++ top] = r;
        while(!is_root(r))
            stk[++ top] = r = tr[r].fa;
        while(top)
            pushdown(stk[top --]);
        while(!is_root(x))
        {
            int y = tr[x].fa, z = tr[y].fa;
            if(!is_root(y))
            {
                if((tr[z].s[1] == y) ^ (tr[y].s[1] == x))
                    rotate(x);
                else 
                    rotate(y);
            }
            rotate(x);
        }
    }

    void access(int x)
    {
        int z = x;
        for(int y = 0 ; x ; y = x, x = tr[x].fa)
        {
            splay(x);
            tr[x].s[1] = y;
            pushup(x);
        }
        splay(z);
    }

    void makeroot(int x)
    {
        access(x);
        pushrev(x);
    }

    int findroot(int x)
    {
        access(x);
        while(tr[x].s[0])
            pushdown(x), x = tr[x].s[0];
        splay(x);
        return x;
    }

    void split(int x, int y)
    {
        makeroot(x);
        access(y);
    }

    void link(int x, int y)
    {
        makeroot(x);
        if(findroot(y) != x)
            tr[x].fa = y;
    }

    void cut(int x, int y)
    {
        makeroot(x);
        if(findroot(y) == x && tr[y].fa == x && !tr[y].s[0])
        {
            tr[x].s[1] = tr[y].fa = 0;
            pushup(x);
        }
    }

    bool check(int x, int y)
    {
        makeroot(x);
        return findroot(y) != x;
    }
}

namespace Solve
{
    using namespace LCT;

    int sum, ans, pos, tot;
    bool st[N];

    struct Edge
    {
        int u, v, w;

        bool operator < (const Edge &a) const {return w < a.w;}
    } e[N];

    void solve()
    {
        read(n), read(m);
        for(int i = 1 ; i <= m ; ++ i)
        {
            int u, v, w;
            read(u), read(v), read(w);
            e[i] = {u, v, w};
        }

        cnt = n, sum = 1, ans = INF;
        std :: sort(e + 1, e + m + 1);

        for(int i = 1 ; i <= m ; ++ i)
        {
            ++ cnt;
            int x = e[i].u, y = e[i].v;
            if(x == y)
            {
                st[i] = true;
                continue;
            }

            if(check(x, y))
                link(x, cnt), link(cnt, y), ++ tot;
            else
            {
                split(x, y);
                pos = tr[y].id;
                st[pos - n] = true, splay(pos);

                tr[tr[pos].s[0]].fa = tr[tr[pos].s[1]].fa = 0;
                link(x, cnt), link(cnt, y);
            }

            while(st[sum] && sum <= i)
                ++ sum;
            if(tot >= n - 1)
                ans = min(ans, e[i].w - e[sum].w);
        }

        write(ans);
    }
}

int main()
{
    Solve :: solve();

    return 0;
}
```

## [SDOI2017 树点涂色](https://www.luogu.com.cn/problem/P3703)

### 题目描述

Bob 有一棵 $n$ 个点的有根树，其中 $1$ 号点是根节点。Bob 在每个点上涂了颜色，并且每个点上的颜色不同。

定义一条路径的权值是：这条路径上的点（包括起点和终点）共有多少种不同的颜色。

Bob可能会进行这几种操作：

- `1 x` 表示把点 $x$ 到根节点的路径上所有的点染上一种没有用过的新颜色。


- `2 x y` 求 $x$ 到 $y$ 的路径的权值。

- `3 x` 在以 $x$ 为根的子树中选择一个点，使得这个点到根节点的路径权值最大，求最大权值。


Bob一共会进行 $m$ 次操作

-------

### 分析

再补充一道题，~~应wjy的要求，我把这道题也放在LCT里讲讲，虽然没必要拿LCT~~。首先按我们看看这道题都有哪些操作，对于第一个操作，我们楞的一看，往 $x$ 到根节点的路径上加一个颜色，我们很容易就能想到 [LCT]{.rainbow} 中的 [access]{.rainbow} 操作，那么一个比较 naive 的想法就是直接修改一下 [access]{.rainbow} 的细节即可。然后我们看第二个操作，求权值，这不好求吗，我们直接看一下这条路径上有多少虚边即可，对于第三个操作我们可以直接求出 $dfs$ 序，用线段树维护一下区间最值。

然后我们看一下我们刚才说的 [access]{.rainbow} 函数应该怎么改，实际上，我们对于这道题我们要求的是虚边的个数，那么我们在 [access]{.rainbow} 的过程中是如果有实边就先将他变成虚边，然后将另一条边建成实边，对于原有的实边，我们造成的贡献就是将他的子树内所有的经过的虚边数量减一，对于新加入的，贡献就自然是加一，然后这道题其实就没啥了。

下面是代码实现

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

namespace Segment
{
    using namespace read_write;

    const int N = 1e6 + 10;

    int n, m;
    int dep[N], siz[N], son[N], top[N], id[N];
    int dfn[N], h[N], fa[N], timestamp, idx;

    struct Edge
    {
        int ne, v;
    } e[N];

    struct TREE
    {
        int l, r;
        int mx, tag;
    } Tr[N];

    void add(int u, int v)
    {
        e[++ idx].v = v, e[idx].ne = h[u], h[u] = idx;
    }

    void dfs1(int u, int fath)
    {
        fa[u] = fath, siz[u] = 1;
        dep[u] = dep[fath] + 1;
        
        for(int i = h[u] ; i ; i = e[i].ne)
        {
            int v = e[i].v;
            if(v == fath)
                continue;
            dfs1(v, u);
            siz[u] += siz[v];
            if(siz[v] > siz[son[u]])
                son[u] = v;
        }
    }

    void dfs2(int u, int tp)
    {
        dfn[u] = ++ timestamp;
        top[u] = tp, id[timestamp] = u;
        if(son[u])
            dfs2(son[u], tp);
        for(int i = h[u] ; i ; i = e[i].ne)
        {
            int v = e[i].v;
            if(v == fa[u] || v == son[u])
                continue;
            dfs2(v, v);
        }
    }

    void Pushup(int u)
    {
        Tr[u].mx = max(Tr[u << 1].mx, Tr[u << 1 | 1].mx);
    }    

    void Pushdown(int u)
    {
        if(Tr[u].tag)
        {
            Tr[u << 1].tag += Tr[u].tag;
            Tr[u << 1 | 1].tag += Tr[u].tag;
            
            Tr[u << 1].mx += Tr[u].tag;
            Tr[u << 1 | 1].mx += Tr[u].tag;
            Tr[u].tag = 0;
        }
    }

    void build(int u, int l, int r)
    {
        Tr[u].l = l, Tr[u].r = r;
        if(l == r)
        {
            Tr[u].mx = dep[id[l]];
            return;
        }

        int mid = l + r >> 1;
        build(u << 1, l, mid), build(u << 1 | 1, mid + 1, r);
        Pushup(u);
    }

    void modify(int u, int l, int r, int w)
    {
        if(l <= Tr[u].l && r >= Tr[u].r)
        {
            Tr[u].tag += w;
            Tr[u].mx += w;
            return ;
        }
        Pushdown(u);
        int mid = Tr[u].l + Tr[u].r >> 1;
        if(l <= mid)
            modify(u << 1, l, r, w);
        if(r > mid)
            modify(u << 1 | 1, l, r, w);
        Pushup(u);
    }

    int query(int u, int l, int r)
    {
        if(l <= Tr[u].l && r >= Tr[u].r)
            return Tr[u].mx;
        Pushdown(u);
        int mid = Tr[u].l + Tr[u].r >> 1;
        int res = -1;
        if(l <= mid)
            res = max(res, query(u << 1, l, r));
        if(r > mid)
            res = max(res, query(u << 1 | 1, l, r));
        return res;
    }
    
    int get_lca(int u, int v)
    {
        while(top[u] != top[v])
        {
            if(dep[top[u]] < dep[top[v]])
                v = fa[top[v]];
            else 
                u = fa[top[u]];
        }

        return dep[u] < dep[v] ? u : v;
    }
}

namespace LCT
{
    using namespace Segment;

    int stk[N];

    struct Splay
    {
        int s[2], rev;
        int fa;
    } tr[N];

    void pushrev(int u)
    {
        swap(tr[u].s[0], tr[u].s[1]);
        tr[u].rev ^= 1;
    }

    void pushdown(int u)
    {
        if(tr[u].rev)
        {
            pushrev(tr[u].s[0]), pushrev(tr[u].s[1]);
            tr[u].rev = 0;
        }
    }

    bool isroot(int x)
    {
        return (tr[tr[x].fa].s[0] != x && tr[tr[x].fa].s[1] != x);
    }

    void rotate(int x)
    {
        int y = tr[x].fa, z = tr[y].fa;
        int k = tr[y].s[1] == x;
        if(!isroot(y))
            tr[z].s[tr[z].s[1] == y] = x;
        tr[x].fa = z;
        tr[y].s[k] = tr[x].s[k ^ 1], tr[tr[x].s[k ^ 1]].fa = y;
        tr[x].s[k ^ 1] = y, tr[y].fa = x;
    }

    void splay(int x)
    {
        int top = 0, r = x;
        stk[++ top] = r;
        while(!isroot(r))
            stk[++ top] = r = tr[r].fa;
        while(top)
            pushdown(stk[top --]);
        while(!isroot(x))
        {
            int y = tr[x].fa, z = tr[y].fa;
            if(!isroot(y))
            {
                if((tr[y].s[1] == x) ^ (tr[z].s[1] == y))
                    rotate(x);
                else 
                    rotate(y);
            }
            rotate(x);
        }
    }

    int findroot(int x)
    {
        while(tr[x].s[0])
            x = tr[x].s[0];
        return x;
    }

    void access(int x)
    {
        int z = x;
        for(int y = 0 ; x ; y = x, x = tr[x].fa)
        {
            splay(x);
            if(tr[x].s[1])
            {
                int temp = findroot(tr[x].s[1]);
                modify(1, dfn[temp], dfn[temp] + siz[temp] - 1, 1);
            }

            if(tr[x].s[1] = y)
            {
                int temp = findroot(tr[x].s[1]);
                modify(1, dfn[temp], dfn[temp] + siz[temp] - 1, -1);
            }
        }
        splay(z);
    }
}

namespace Solve
{
    using namespace LCT;

    void solve()
    {
        read(n), read(m);

        for(int i = 1 ; i < n ; ++ i)
        {
            int u, v;
            read(u), read(v);
            add(u, v), add(v, u);
        }

        dfs1(1, 0),  dfs2(1, 1);
        for(int i = 1 ; i <= n ; ++ i)   //一定要记得初始化 splay 中的每个节点父亲
            tr[i].fa = fa[i];
        build(1, 1, n);

        while(m -- )
        {
            int op, x;
            read(op), read(x);
            if(op == 1)
                access(x);
            else if(op == 2)
            {
                int y, lca, ans;
                read(y);
                lca = get_lca(x, y);
                ans = query(1, dfn[x], dfn[x]) + query(1, dfn[y], dfn[y]) - 2 * query(1, dfn[lca], dfn[lca]) + 1;
                write(ans), puts("");
            }
            else 
            {
                write(query(1, dfn[x], dfn[x] + siz[x] - 1));
                puts("");
            }
        }
    }
}

int main()
{
    Solve :: solve();

    return 0;
}
```

# [エピローグ]{.rainbow}

那么到这里常用平衡树和技巧啥的就都结束了，其实平衡树理解了原理之后学起来应该是比较简单的吧 [/kel]{.rainbow} 。剩下的就是练题了。

+++success 题单
- [LCT](https://www.luogu.com.cn/training/367843#problems)
- [平衡树](https://www.luogu.com.cn/training/368520#information)
+++