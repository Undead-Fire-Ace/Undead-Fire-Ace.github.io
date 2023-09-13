---
title: SHOI2009 舞会 题解
date: 2023-08-22 11:29:08
tags: 
	- 题解
	- 动态规划
---
# [题目传送门](https://www.luogu.com.cn/problem/P2159)

# 简要题意：
问这给定的 $n$  对舞伴中只有 $k$ 对女伴比男伴高的方案数

# 分析：

先浅浅的看一下题干，要求组合数但是没有给 $mod$ 而且 $N$ 和 $K$ 达到了200，这必然会爆掉，然后就可以得出一个显然易见结论，要写高精度（~~**不想写开溜**~~）。

先浅挂一下高精度的板子,因为有些人可能就懒得写了（doge。下面挂的是重载运算符的板子，可以省去函数名的重复敲击
```cpp
struct Int     //高精度的板子就先不说了
{
#define max_size (501)
	int val[max_size];
	int len;
	const int base = 10;
	Int(int _val = 0)
	{
		len = 1;
		for (lv_WXT int i = 0; i < max_size; i++)
			val[i] = 0;
		*this = _val;
	}
	Int(const char s[max_size]) { init(s); }
	Int(char s[max_size]) { init(s); }
	int &operator[](const int id) { return val[id]; }
	void operator=(int _val)
	{
		for (lv_WXT int i = 0; i <= len; i++)
			val[i] = 0;
		len = 0;
		while (_val)
			val[++len] = _val % 10, _val /= 10;
	}
	void operator=(Int pos)
	{
		*this = 0;
		len = pos.len;
		for (lv_WXT int i = 1; i <= len; i++)
			val[i] = pos.val[i];
	}
	void carry_bit()
	{
		for (lv_WXT int i = 1; i <= len; i++)
		{
			if (val[i] > 9)
			{
				val[i + 1] += val[i] / 10;
				val[i] %= 10;
				if (i == len)
					len++;
			}
		}
	}
	void eat_zero()
	{
		for (; !val[len] && len > 1; len--)
			;
		if (!len)
			len = 0, val[len] = 0;
	}
	Int operator+(int _val)
	{
		Int now = *this;
		now[1] += _val;
		now.carry_bit();
		return now;
	}
	Int operator+(Int _val)
	{
		Int now = *this;
		now.len = max(now.len, _val.len);
		for (lv_WXT int i = 1; i <= _val.len; i++)
			now[i] += _val[i];
		now.carry_bit();
		return now;
	}
	Int operator*(int _val)
	{
		Int now = *this;
		for (lv_WXT int i = 1; i <= len; i++)
			now[i] *= _val;
		now.carry_bit();
		return now;
	}
	Int operator*(Int _val)
	{
		Int now;
		now.len = len + _val.len - 1;
		for (lv_WXT int i = 1; i <= now.len; i++)
		{
			for (lv_WXT int j = 1; j <= _val.len; j++)
			{
				now[i + j - 1] += val[i] * _val[j];
			}
		}
		now.eat_zero();
		now.carry_bit();
		return now;
	}
	Int operator/(int _val)
	{
		Int now = *this;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (i)
				now[i - 1] += base * (now[i] % _val);
			now[i] /= _val;
		}
		now.eat_zero();
		return now;
	}
	bool operator<(Int _val) const
	{
		if (len != _val.len)
			return len < _val.len;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (val[i] != _val[i])
				return val[i] < _val[i];
		}
		return 0;
	}
	bool operator==(Int _val) const
	{
		if (len != _val.len)
			return 0;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (val[i] != _val[i])
				return 0;
		}
		return 1;
	}
	bool operator>(Int _val) const
	{
		if (len != _val.len)
			return len > _val.len;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (val[i] != _val[i])
				return val[i] > _val[i];
		}
		return 0;
	}
	bool operator<=(Int _val) const { return !(*this > _val); }
	bool operator>=(Int _val) const { return !(*this < _val); }
	void init(const char s[max_size])
	{
		len = 0;
		int begin = 0;
		for (lv_WXT int i = 1; s[i] >= '0' && s[i] <= '9'; i++)
		{
			if (!begin && s[i] == 48)
				continue;
			if (!begin)
			{
				begin = i;
			}
			len++;
		}
		if (!begin)
		{
			len = 0, val[1] = 0;
			return;
		}
		for (lv_WXT int i = 1; i <= len; i++)
		{
			val[i] = s[len - i + begin] - 48;
		}
	}
	void operator=(const char s[max_size]) { this->init(s); }
	void print()
	{
		eat_zero();
		if (!len)
			putchar(48);
		else
			for (lv_WXT int i = len; i >= 1; i--)
				putchar(val[i] + 48);
		putchar('\n');
	}
};
```

然后又想，高精度是个 $(n^2)$ 复杂度的算法，这就必然不能用 $(n^3)$ 的状态转移方程了。因此我们先对男生和女生分别排序，どうしてですか？因为我们发现排序后会有单调性就更有利于我们的状态转移，然后设 $dp(i,j)$ 为前 $i$ 个男生和女生已经两两匹配总共有 $j$ 对符合题意的方案数。那么最终的答案就应该是 $\sum_{i = 0}^{k}dp(n,i)$ 。

接下来就要考虑怎么转移状态了，假如当前的女生的身高小于等于男生的身高：

## 当前女生不高于男生:

### ①：这个女生不使 $j$ 增加。
然后将这个女生放在身高大于等于她的那个男生的位置，就会形成一个新的不会影响$j$的数量的方案。

我们可以从当前位置向前扫一遍就可以得到符合 这种情况的位置的个数，暂且设个数为 $k$ 。我们也可以将当前女生放到任意原来就是女生更高的位置。由于女生身高已经过排序，交换后“原来就是女生更高的位置”仍然是女生更高，而当前位置也仍然是男生不矮于女生。

因为我们枚举的时候是从小到大枚举的,更换位置是将更高的放在前面，而这样的女生在换过来之后就一定是小于当前的男生的，显而易见，这样的位置有 $j$ 个。

并且 $k$ 和 $j$ 不会重叠，因为$k$位置的部分满足男生比女生高。所以，对于 $dp(i,j)$ 共有 $j+k$ 个位置使 $j$ 不增加

### ②:这个女生使 $j$ 增加.
因为刚才已经求出了不会使 $j$ 增加的位置数，那么用总数 $i$ 减去上面的个数就是能使 $j$ 增加的个数

然后这种情况下的递推方程为:

$dp[i][j]\ = \ dp[i - 1][j] \ \times \ (j \ + \ k ) \ + \ dp[i \ - \ 1][j \ - \ 1]\ \times \  (i-k-j-1)$

第一类的情况解决掉了，接下来看第二类情况。

## 当前的女生比男生高：

### ①：当前女生不使 $j$ 增加。

这个时候无论将她放在哪里都会形成一组女生比男生高的组合，因为此时的女生一定高于当前所枚举的所有男生。我们现在考虑她不影响 $j$ 的位置有哪些，然后可以轻松地发现其实将她放在一个之前就是女生比男生高的位置同时那个位置的女生不高于当前位置的男生。

女生高于男生的位置数就是已经求出来的 $j$ ，然后再扫一遍得出有多少女生高于当前男生，计个数为 $k$ 个，那么满足不影响 $j$ 的位置数就是有 $j - k$ 个。

### ②：当前女生使 $j$ 增加

同理第一大类的这种情况，总数 $i$ 减去上面的方案数就行。

那么我们也就得出了这类情况的转移方程：

$dp[i][j] \ = \ dp[i - 1][j] \ \times\ (j-k) \ + \ dp[i-1][j-1] \ \times \ (i-j+k+1)$

最后的最后，因为我们发现转移过程只与 $i$ 和 $i - 1$ 有关系，因此可以用滚动数组来优化空间，（其实这道题的数据规模不用滚动数组优化空间也行，而且会跑到更快，但是还是要学一下这种思想的）具体代码实现方式会在下面的代码中说明

```cpp
#include <cstring>
#include <iostream>
#include <algorithm>

#define lv_WXT register     //无关紧要的define，忽略就行(doge

using namespace std;

const int N = 200 + 10;

int n, jk;
int m[N], w[N];

struct Int     //高精度的板子就先不说了
{
#define max_size (501)
	int val[max_size];
	int len;
	const int base = 10;
	Int(int _val = 0)
	{
		len = 1;
		for (lv_WXT int i = 0; i < max_size; i++)
			val[i] = 0;
		*this = _val;
	}
	Int(const char s[max_size]) { init(s); }
	Int(char s[max_size]) { init(s); }
	int &operator[](const int id) { return val[id]; }
	void operator=(int _val)
	{
		for (lv_WXT int i = 0; i <= len; i++)
			val[i] = 0;
		len = 0;
		while (_val)
			val[++len] = _val % 10, _val /= 10;
	}
	void operator=(Int pos)
	{
		*this = 0;
		len = pos.len;
		for (lv_WXT int i = 1; i <= len; i++)
			val[i] = pos.val[i];
	}
	void carry_bit()
	{
		for (lv_WXT int i = 1; i <= len; i++)
		{
			if (val[i] > 9)
			{
				val[i + 1] += val[i] / 10;
				val[i] %= 10;
				if (i == len)
					len++;
			}
		}
	}
	void eat_zero()
	{
		for (; !val[len] && len > 1; len--)
			;
		if (!len)
			len = 0, val[len] = 0;
	}
	Int operator+(int _val)
	{
		Int now = *this;
		now[1] += _val;
		now.carry_bit();
		return now;
	}
	Int operator+(Int _val)
	{
		Int now = *this;
		now.len = max(now.len, _val.len);
		for (lv_WXT int i = 1; i <= _val.len; i++)
			now[i] += _val[i];
		now.carry_bit();
		return now;
	}
	Int operator*(int _val)
	{
		Int now = *this;
		for (lv_WXT int i = 1; i <= len; i++)
			now[i] *= _val;
		now.carry_bit();
		return now;
	}
	Int operator*(Int _val)
	{
		Int now;
		now.len = len + _val.len - 1;
		for (lv_WXT int i = 1; i <= now.len; i++)
		{
			for (lv_WXT int j = 1; j <= _val.len; j++)
			{
				now[i + j - 1] += val[i] * _val[j];
			}
		}
		now.eat_zero();
		now.carry_bit();
		return now;
	}
	Int operator/(int _val)
	{
		Int now = *this;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (i)
				now[i - 1] += base * (now[i] % _val);
			now[i] /= _val;
		}
		now.eat_zero();
		return now;
	}
	bool operator<(Int _val) const
	{
		if (len != _val.len)
			return len < _val.len;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (val[i] != _val[i])
				return val[i] < _val[i];
		}
		return 0;
	}
	bool operator==(Int _val) const
	{
		if (len != _val.len)
			return 0;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (val[i] != _val[i])
				return 0;
		}
		return 1;
	}
	bool operator>(Int _val) const
	{
		if (len != _val.len)
			return len > _val.len;
		for (lv_WXT int i = len; i >= 1; i--)
		{
			if (val[i] != _val[i])
				return val[i] > _val[i];
		}
		return 0;
	}
	bool operator<=(Int _val) const { return !(*this > _val); }
	bool operator>=(Int _val) const { return !(*this < _val); }
	void init(const char s[max_size])
	{
		len = 0;
		int begin = 0;
		for (lv_WXT int i = 1; s[i] >= '0' && s[i] <= '9'; i++)
		{
			if (!begin && s[i] == 48)
				continue;
			if (!begin)
			{
				begin = i;
			}
			len++;
		}
		if (!begin)
		{
			len = 0, val[1] = 0;
			return;
		}
		for (lv_WXT int i = 1; i <= len; i++)
		{
			val[i] = s[len - i + begin] - 48;
		}
	}
	void operator=(const char s[max_size]) { this->init(s); }
	void print()
	{
		eat_zero();
		if (!len)
			putchar(48);
		else
			for (lv_WXT int i = len; i >= 1; i--)
				putchar(val[i] + 48);
		putchar('\n');
	}
};

inline void read(int &a)
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
	a = x * f;
}

void qsort(int *array, int low, int high)   //快排
{
	if (low >= high)
		return;
	int i = low;
	int j = high;
	int key = array[low];
	
	while (i < j)
	{
		while (array[j] >= key && i < j)
			j--;
		array[i] = array[j];
		while (array[i] <= key && i < j)
			i++;
		array[j] = array[i];
	}
	array[i] = key;
	
	qsort(array, low, i - 1);
	qsort(array, i + 1, high);
}

int main()
{
	Int f[2][N];   //要定义成高精度的类型
	
	read(n), read(jk);

	for (lv_WXT int i = 1; i <= n; i++)
		read(m[i]);
	for (lv_WXT int i = 1; i <= n; i++)
		read(w[i]);
	qsort(m, 1, n);
	qsort(w, 1, n);

	f[0][0] = 1;

	for (lv_WXT int i = 1; i <= n; i++)
	{
		int k = 0, sg = i & 1; // k 就是上面分析的位置数
		// sg 是找一下奇数和偶数，因为如果当前为奇数，在进行本次转移前会清空奇数的数组，而那个奇数的数组是i - 2位置的值。偶数的是i - 1的值，如果sg为偶数则反之

		for (lv_WXT int j = 0; j <= i; j++) // 采取滚动数组优化，所以每次使用前都要初始化一遍（j一定要从0开始，否则会没有完全初始化,第一次提交的时候就因为这个挂了）
		{
			f[sg][j] = 0;
		}

		if (m[i] >= w[i]) // 男生不低于女生的情况
		{
			for (lv_WXT int j = 1; j <= i; j++)
			{
				if (m[j] >= w[i]) // 处理一下k
					k++;
			}

			for (lv_WXT int j = 0; j <= i; j++) // 处理前半部分
			{
				f[sg][j] = f[!sg][j] * (j + k);
			}

			for (lv_WXT int j = 1; j <= i - k; j++) // 直接加上与它奇偶性不同的那个数组即可
			{
				f[sg][j] = f[sg][j] + f[!sg][j - 1] * (i - k - j + 1);
			}
		}

		else // 男生没有女生高的情况
		{
			for (lv_WXT int j = 1; j < i; j++)
			{
				if (w[j] > m[i])
					k++;
			}

			for (lv_WXT int j = 1; j <= i; j++) // 处理前半部分
			{
				f[sg][j] = f[!sg][j - 1] * (i - j + 1 + k);
			}

			for(lv_WXT int j = k ; j <= i ; j ++)
			{
				f[sg][j] = f[sg][j] + f[!sg][j] * (j - k);
			}
		}
	}

	Int res = 0;   //res要用高精度的

	for(lv_WXT int i = 0 ; i <= jk ; i ++ )  //最后统计答案，因为奇数的存在了sg为奇数的位置，偶数则反之
	{
		res = res + f[n & 1][i];
	}

	res.print();

	return 0;
}
```
2023.5.22 是洛谷上的第七个最优解。orz

撒花完结，✿✿ヽ(°▽°)ノ✿