---
title: SCOI2007压缩 题解
date: 2023-08-22 11:27:29
tags: 
    - 题解
    - 字符串
    - 动态规划
---
# [[SCOI2007]压缩](https://www.luogu.com.cn/problem/P2470)

## 题目描述

给一个由小写字母组成的字符串，我们可以用一种简单的方法来压缩其中的重复信息。压缩后的字符串除了小写字母外还可以（但不必）包含大写字母R与M，其中M标记重复串的开始，R重复从上一个M（如果当前位置左边没有M，则从串的开始算起）开始的解压结果（称为缓冲串）。


`bcdcdcdcd` 可以压缩为 `bMcdRR`，下面是解压缩的过程：


已经解压的部分|解压结果|缓冲串
---|---|---
b|b|b
bM|b|.
bMc|bc|c
bMcd|bcd|cd
bMcdR|bcdcd|cdcd
bMcdRR|bcdcdcdcd|cdcdcdcd

## 输入格式

输入仅一行，包含待压缩字符串，仅包含小写字母，长度为n。

## 输出格式

输出仅一行，即压缩后字符串的最短长度。

## 样例 #1

### 样例输入 #1

```
aaaaaaa
```

### 样例输出 #1

```
5
```

## 样例 #2

### 样例输入 #2

```
bcdcdcdcdxcdcdcdcd
```

### 样例输出 #2

```
12
```

## 提示

在第一个例子中，解为aaaRa，在第二个例子中，解为bMcdRRxMcdRR。


【限制】

50%的数据满足：1<=n<=20

 
100%的数据满足：1<=n<=50

------------------------

## 简要题意：你对字符串进行压缩，你压缩的部分必须是连续的重复的，压缩之后为 M 开始， R 结束，M 和 R 之间是重复的部分（也就是你进行压缩的那部分的一个循环节）


读完题之后我们可以发现这个题非常像[这道题](https://www.luogu.com.cn/problem/P4302)。建议先做那个题再做这道题。

如果你做完了刚才那道题你就会发现这两道题怎么这么像，~~（不禁会抱怨四川怎么出重题啊）~~。但是我们要想一想他这个题为什么会是个紫题而刚才的不是，~~紫题肯定有紫题的道理（bushi~~。

我们其实手玩一下样例会发现其实这两个题的唯一的差距就是缩了之后的长度会不一样，这道题是你选取不同的循环节可能答案不同，但是上道题是如果长度一定了，能缩就缩，同时上道题的每个缩短之间是不会相互制约的，但是这道题如果是前面有过了 M ，后面有的时候是可以少加 M 的，所以我们会发现其实跟上道题的区别就在于这个 M 上了，然后我们考虑怎么处理。

# 有位学长说过：DP 不了，再加一维！！！

那既然我们 M 有影响，我们就在上一道题的基础上再加一维变成 $f[i][j][0/1]$ ， 0 表示之前没有 M ， 1 反之。转移方程也跟上道题极为相似，然后你就可以开心的~~爆~~切这道题了。

什么？？？你问怎么判断是否可以压缩，我们看一下字符串的长度，发现小的可怜，就直接暴力判断前半部分和后半部分是否相同即可。

老爹：还有一件事情~~，我的代码读入字符串的时候是用的string，但是我习惯循环的时候从1开始，我们在读入之后在那个串前面加个空格即可
### 还有一定要先求字符串长度再加空格，要不然长度就错了（你后求长度的话要减一）！！

下面是代码实现

```cpp
#include <iostream>
#include <queue>
#include <cstring>

#define cin std::cin

namespace read_write
{
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

    int max(int x, int y)
    {
        return x > y ? x : y;
    }

    int min(int x, int y)
    {
        return x > y ? y : x;
    }
}

using namespace read_write;

const int N = 100 + 10;

int n;
int f[N][N][2];
std::string a;

bool check(int l, int r)    //判断前半部分和后半部分是否相等
{
    if((r - l + 1) & 1)
        return false;
    int mid = l + r >> 1;
    for(int i = l ; i <= mid;  ++ i)
        if(a[i] != a[i + mid - l + 1])
            return false;
    return true;
}

int main()
{
    cin >> a;
    n = a.size();
    a = ' ' + a;

    std::memset(f, 0x3f, sizeof(f));

    for(int i = 1 ; i <= n ; ++ i)
    {
        for(int j = i ; j <= n ; ++ j)
            f[i][j][0] = f[i][j][1] = j - i + 1;
    }

    for(int len = 2 ; len <= n ; ++ len)
    {
        for(int l = 1 ; l + len - 1 <= n ; ++ l)
        {
            int r = l + len - 1;

            if(check(l, r))
                f[l][r][0] = min(f[l][(l + r) / 2][0] + 1, f[l][r][0]);
            for(int k = l ; k < r ; ++ k)
                f[l][r][0] = min(f[l][r][0], f[l][k][0] + r - k);
            for(int k = l ; k < r ; ++ k)
                f[l][r][1] = min(f[l][r][1], min(f[l][k][0], f[l][k][1]) + min(f[k + 1][r][0], f[k + 1][r][1]) + 1);
        }
    }

    write(min(f[1][n][1], f[1][n][0]));

    return 0;
}
```
*★,°*:.☆(￣▽￣)/$:*.°★* 。