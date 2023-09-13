---
title: 炸脖龙Ⅰ
date: 2023-08-21 19:51:44
tags: 
	- 题解
	- 数论
	- 数据结构
	- 树状数组
---
# [题目传送门](https://www.luogu.com.cn/problem/P3934)

前置芝士：扩展欧拉定理

$$ a^b\equiv\left\{
\begin{aligned}
a^b (b < \varphi(p)) \\
a^{b \hspace{1mm} mod \hspace{1mm} \varphi(p)+\varphi(p)} (b\geq\varphi(p)) \\
\end{aligned}
\right.(mod\hspace{1mm}p)
$$



```cpp
//ふかんぜん (不完美的代码)
int solve(int l,int r,int p)
{
    if(p == 1)    //模数为1时永远为0
        return 0;
    if(l == r)    //区间为1时，为原值
        return a[l] % p;
    return qmi(a[l], solve(l + 1, r, phi(p)) + phi(p), p)   //根据公式推导出来
} 
```



为何不完美？？？？

我们可以清楚的看到扩展欧拉定理的有$\hspace{1mm}b\hspace{1mm}$和$\hspace{1mm}\varphi(p)\hspace{1mm}$的大小的判断，而我们的ふかんぜん并没有体现这一点，所以就搞一下大小关系的判断就行。

最朴素的存储方式就是存到一个 構造体です(结构体) 里

```cpp
struct NUM
{
    int val;   // 数字的大小
    bool flag; // 是否大于模数的欧拉函数

    NUM(int val = 0, bool flag = false) : val(val), flag(flag) {} // 初始化
};
```

接着我们快乐的将这玩意补到上面的 ふかんぜん 的里面就可以愉快的解决这道题最难的部分——区间查询

```cpp
NUM solve(int l,int r,int p)
{
    NUM res;   //实际上是在求a[l]的指数
    if(p == 1) 
        return NUM(0,true);
    
    if(a[l] == 1)
        return NUM(1,false);
    
    if(l == r)
    {
        if(a[l] >= p)
            return NUM(a[l] % p,true);
        else 
            return NUM(a[l],false);    //这里不要 % p，要是%了就与公式不一样了
    }

    res = solve(l + 1,r,phi(p));   //递归求解
    if(res.flag)
        res.val += phi(p);
    return qmi(a[l],res.val,p);   //快速幂的返回类型和这个函数的类型要保持一致
}
```

上面的是不带修改的，带上修改，因为树状数组足够快（比线段树快），修改直接暴力修改。接下来的就是树状数组的常规操作和求解欧拉函数：借助lowbit去查询和修改，就不再放代码了

我就先在这放一下树状数组的区间修改和区间查询操作的代码

```cpp
void modify(int i,int x)
{
	while(i <= n)
	{
		a[i] += x;
		i += lowbit(i);
	}
}

void add(int l,int r,int x)
{
	modify(l,x);
	modify(r + 1,-x);
}

inline int query(int i)
{
    int res = 0;
    while(i){
        res += a[i];
        i -= lowbit(i);
    }
    return res;
}
```
其实就是利用差分的数组去维护一个树状数组的差分数据然后就行了。快速幂其实也有点细节需要注意下，注意点放在下面的代码里了

```cpp
inline NUM qmi(int a,int t,int p)
{
	NUM res = NUM(1,0);
	if(a >= p)  
	{
		a %= p;
		res.flag = 1;
	}

	while(t)
	{
		if(t & 1)
			res.val *= a;
		if(res.val >= p)   //刚才说了要判断是否大于p，因此在mod的时候就判断一下
		{
			res.flag = 1;
			rea.val %= p;
		}

		a *= a;

		if(a >= p)
		{
			res.flag = 1;
			a %= p;
		}
		t >>= 1;
	}
	return res;
}
```
然后这道题就愉快的结束了，这道题可以说是Ynoi里非常简单的一道题，总体实现也不难，算上预处理欧拉函数的复杂度，整体的复杂度为$O(p \hspace{1mm}+\hspace{1mm} q^{log\hspace{1mm}np\hspace{1mm}log\hspace{1mm}p})$。

**撒花完结✿✿ヽ(°▽°)ノ✿**


 