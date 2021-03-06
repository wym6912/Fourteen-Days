## 第 6 天 线段树\(II\) 区间修改 线段树的问题转换

今天依旧是 V8 老师。一去上课，他就提问我：“这个数列有什么规律吗？”我：“???”

[提高班第 6 天练习题](/vjudge.net/contest/177295)

[入门班第 8 天练习题](https://vjudge.net/contest/177627)

[入门班第 9 天练习题](https://vjudge.net/contest/177956)

今天开始接着昨天的论题，区间修改、问题转换。

#### 1. 线段树的区间修改

按照昨天的说法，线段树的区间修改的复杂度是$$O( \log {N})$$，但是更新的区间如果退化成了一个点，那么更新的复杂度期望为$$O(N \log {N})$$，甚至不及数组。

所以我们需要一个东西：标记下传。

例题：[POJ 3468](http://poj.org/problem?id=3468)，这道题我们需要去从更新区间，查询区间和。

按照样例：

```
10 2
1 2 3 4 5 6 7 8 9 10
C 2 9 3
Q 2 4
```

对于修改操作，类比询问操作的思想，也是沿着两条链下去修改每一段子区间，不过这里出现了一个问题：如右图如果修改区间$$[2,9]$$，但是修改到$$[6,8]$$时就已经结束了，这样的话，$$[6,7], [8,8], [6,6], [7,7]$$这四段区间并没有修改。

![](/PIC_Day6_1.png)

lazy 思想可以解决这个问题。再更新的地方如下所示：（仅展示了$$[6, 10]$$向$$[6, 8]$$和$$[9, 10]$$传递标记的过程）

![](/PIC_Day6_2.png)

我们对于线段树上的每个结点再维护一个$$lazy$$值，如图，假设$$[6, 8]$$区间的编号为$$rt$$。当我们给$$[6,8]$$加$$c$$时，同时`lazy[rt] += c`。表示有一个$$+ c$$的修改还没有传下去，这样的话，当我们查询$$[6,7] , [8,8], [6,6], [7,7]$$时，再将修改传下去就行了。

我们需要定义一个结构体：

```cpp
typedef struct
{
    ll data, tag;
}node;
```

开设一个数组：

```cpp
node tree[maxn];
```

首先，我们需要把数字读入。

```cpp
void pre(int l, int r, int p)
{
    tree[p].l = l;
    tree[p].r = r;
    if(l == r)
    {
        scan_d(tree[p].tag);
        pushup(p, l, r);//注意在计算的位置 
        return ;
    }
    int mid = midf(l, r);
    pre(l, mid, DXA(p));
    pre(mid + 1, r, DXB(p));
    pushup(p, l, r);
}
```

完成上传结果函数：

```cpp
void pushup(int p, int l, int r)
{
    tree[p].data = 0;
    if(r > l) //注意这个是根节点
    {
        tree[p].data += tree[DXA(p)].data + tree[DXB(p)].data;
    }
    tree[p].data += tree[p].tag * (r - l + 1);//自己已经更新
}
```

开始完成更新函数：

```cpp
void update(int l, int r, int nl, int nr, long long d, int p)
{
    if(l <= nl && nr <= r)
    {
        tree[p].tag += d; //注意只累加了一个 d, tag 表示的是 [l, r] 的累加数为 tree[p].tag
        //在这里，标记并没有下传
        pushup(p, nl, nr);
        return ;
    }
    int mid = midf(nl, nr);
    if(l <= mid)
        update(l, r, nl, mid, d, DXA(p));
    if(mid < r)
        update(l, r, mid + 1, nr, d, DXB(p));
    pushup(p, nl, nr);
}
```

完成查询函数：

```cpp
long long query(int l, int r, int nl, int nr, int p, long long tags)
//查询[l, r] 区间的值, 在他的祖先结点有 tags 需要累加进结果
{
    if(l <= nl && nr <= r)
    {
        return tree[p].data + (tags) * (nr - nl + 1);
        //为什么这里没有累加自己的 tree[p].tag？
        //因为 tree[p].data 就是自己的 tag * 自己的长度了 
    }
    else
    {
        long long ans = 0ll;
        int mid = midf(nl, nr);
        if(l <= mid)
            ans += query(l, r, nl, mid, DXA(p), tags + tree[p].tag);
        if(mid < r)
            ans += query(l, r, mid + 1, nr, DXB(p), tags + tree[p].tag);
        return ans;
    }
}
```

这道题可以将模型抽象如下：

```
1. 将 [a, b] 的每个值累加 c;
2. 查询 [a, b] 的总和/最值。
```

[POJ 3468 答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/POJ/3468.cpp)

例题：火车售票预订系统。每个售票系统申请包含三个参数，分别用$$(O, D, N)$$表示，$$O$$为起点，$$D$$为目的地点，$$N$$为车票张数。售票系统对该售票申请作出受理或不受理的决定，只有在$$[O, D]$$的区段内列车上都有$$N$$个以上的空座位时才可以买票。

分析一下：我们应该每次先查询$$[O, D]$$的最小值，如果最小值大于等于$$N$$即可将它更新。

那么这道题可以将模型抽象为上面所提及的模型。

还有一种模型，可以看下面的例题：

例题：[HDU 1698](http://acm.hdu.edu.cn/showproblem.php?pid=1698)，这道题是区间查询、将一个区间内的所有数替换成另一个数。

那么我们应该怎么办呢？依然使用 lazy 思想。

比如说还是要更新$$[6, 9]$$的值，我们应该怎么修改线段树的域呢？（仅展示了$$[6, 10]$$向$$[6, 8]$$和$$[9, 10]$$传递标记的过程）![](/PIC_Day6_3.png)我们需要把上面的一些函数进行~~修改~~重构，如下所示：

首先是 pushdown 函数，将标记进行下传：

```cpp
void pushdown(int p, int l) //位置为 p 的结点，表示区间为[l, r]
{
    if(tree[p].lazy)//有标记才需要下传
    {
        tree[DXA(p)].lazy = tree[DXB(p)].lazy = tree[p].lazy; 
        tree[DXA(p)].data = (l - (l >> 1)) * tree[p].lazy;  
        //一定要注意这里：如果在这里更新过孩子结点的信息，就不要调用 pushup() 函数
        tree[DXB(p)].data = (l >> 1) * tree[p].lazy;  
        tree[p].lazy = 0;
    }
}
```

update 函数：

```cpp
void update(int l, int r, int nl, int nr, int d, int p)
{
    if(l <= nl && nr <= r)
    {
        tree[p].lazy = d;
        tree[p].data = tree[p].lazy * (nr - nl + 1);
        return ;
    }
    else 
    {
        pushdown(p, nr - nl + 1);//标记下传
        int mid = midf(nl, nr);
        if(l <= mid)update(l, r, nl, mid, d, DXA(p));
        if(mid < r)update(l, r, mid + 1, nr, d, DXB(p));
    }
    pushup(p, nl, nr);
}
```

注意，这里 pushdown 和 update 还可以这样写：

```cpp
void pushdown(int p)
{
    if(tree[p].lazy)
    {
        tree[DXA(p)].lazy = tree[DXB(p)].lazy = tree[p].lazy;
        tree[p].lazy = 0;
        //不重算结点的信息
    }
}

void update(int l, int r, int nl, int nr, int d, int p)
{
    if(l <= nl && nr <= r)
    {
        tree[p].lazy = d;
        pushup(p, nl, nr);
        return ;
    }
    pushdown(p);
    int mid = midf(nl, nr);
    if(l <= mid)update(l, r, nl, mid, d, DXA(p));
    else pushup(DXA(p), nl, mid); // 在标记下传以后，需要重新计算结点的信息
    if(mid < r)update(l, r, mid + 1, nr, d, DXB(p));
    else pushup(DXB(p), mid + 1, nr);
    pushup(p, nl, nr);
}
```

现在我们需要写一下 pushup 函数：

```cpp
void pushup(int p, int l, int r)
{
    tree[p].data = 0;
    if(r > l)tree[p].data = tree[DXA(p)].data + tree[DXB(p)].data;
    tree[p].data += tree[p].lazy * (r - l + 1);
}
```

query 函数可以这样写：

```cpp
int query(int l, int r, int nl, int nr, int p)
{
    if(tree[p].lazy)
        return tree[p].lazy * (min(nr, r) - max(nl, l) + 1);
    else if(l <= nl && nr <= r)
        return tree[p].data;
    else
    {
        int mid = midf(nl, nr);
        int ans = 0;
        if(l <= mid)ans += query(l, r, nl, mid, DXA(p));
        if(mid < r) ans += query(l, r, mid + 1, nr, DXB(p));
        return ans;
    }
}
```

也可以这样完成：

```cpp
int query(int l, int r, int nl, int nr, int p)
{
    if(l <= nl && nr <= r)
        return tree[p].data;
    else
    {
        pushdown(p);
        int mid = midf(nl, nr);
        int ans = 0;
        if(l <= mid)ans += query(l, r, nl, mid, DXA(p));
        else pushup(DXA(p), nl, mid);
        if(mid < r) ans += query(l, r, mid + 1, nr, DXB(p));
        else pushup(DXB(p), mid + 1, nr);
        pushup(p, nl, nr);
        return ans;
    }
}
```

不管怎么说，一定要保证在查询结果之前，**区间信息**都是**正确**表达结果的。

我们于是就这样完成了这道题。

这道题可以将模型抽象如下：

```
1. 将 [a, b] 的每个值改为 c;
2. 查询 [a, b] 的总和/最值。
```

练习题：[POJ 2777](http://poj.org/problem?id=2777)，可以利用位运算表示一个区间的颜色 [答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/POJ/2777.cpp)

练习题：[ZOJ 1610](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=1610)，注意是在$$[0, 8000]$$区间里面染$$(l, r)$$，询问的是这个区域有多少颜色，各个颜色长度为多少。[答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/ZOJ/1610.cpp)

#### 1'. 线段树的区间修改 - 续

在看 [kuangbin 专题 7](https://vjudge.net/contest/166945) 的时候，发现一类题不能完全利用上面的方法去完成。所以特地将它总结如下：

例题：[HDU 4027](http://acm.hdu.edu.cn/showproblem.php?pid=4027)，将$$[l, r]$$中所有数都开根号（向下取整），并求和。例如样例：

```
10
1 2 3 4 5 6 7 8 9 10
5
0 1 10
1 1 10
1 1 5
0 5 8
1 4 8
```

对$$[1, 10]$$开根号以后，序列变为

```
1 1 1 2 2 2 2 2 3 3
```

发现什么了吗？如果对单个数开根号的话，一定会**收敛**到`1`的。

所以我们在更新这个序列的时候，判断如果区间$$[l, r]$$到了收敛值，不再更新子区间。

这个怎么实现呢？

修改 update 函数：

```cpp
void update(int l, int r, int nl, int nr, int data, int p)
{
    if(l <= nl && nr <= r)
    {
        if(tree[p].data == nr - nl + 1) //在这里优化了 一个 ll 型的数开方次数有限 
            return ;
    }
    if(nl == nr)//注意区间的表示 
    {
        tree[p].data = (ll)sqrt(tree[p].data * 1.0);
        return ;
    }
    int mid = midf(nl, nr);
    if(l <= mid)update(l, r, nl, mid, data, DXA(p));
    if(mid < r) update(l, r, mid + 1, nr, data, DXB(p));
    pushup(p);
}
```

[答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/HDU/4027.cpp)

（穿越时空的一道题）例题：[CodeForces 920F](http://codeforces.com/contest/920/problem/F)，将$$[l, r]$$更新为$$f(l), f(l + 1), ..., f(r)$$，其中$$f(x) = $$能整除$$x$$的个数。例如样例：

```
7 6
6 4 1 10 3 2 4
1 1 7
2 4 5
1 3 5
2 4 4
1 5 7
2 1 7
```

经历了第 1 个变换后，得到

```
4 3 1 4 2 2 3
```

发现什么了吗？如果对单个数做$$f$$变换的话，一定会**收敛**到`1`或者`2`的。注意这里有两个收敛值！

跟上一道题一样，如果$$[l, r]$$到了收敛值，不再更新子区间。

针对这种情况，我们需要这样修改 update 函数：

```cpp
void update(int l, int r, int nl, int nr, int p)
{
    if(tree[p].max_d <= 2)return ; //需要统计最大值，修改 pushup 函数实现
    if(nl == nr)
    {
        tree[p].tag ++;
        while(tree[p].tag > 0)
        {
            tree[p].sum = trans[tree[p].sum];
            tree[p].tag --;
            if(tree[p].sum <= 2)break;
        }
        pushup_2(p);
        return ;
    }
    pushdown(p);
    int mid = midf(nl, nr);
    if(l <= mid)update(l, r, nl, mid, DXA(p));
    else pushup(DXA(p), nl, mid);
    if(mid < r) update(l, r, mid + 1, nr, DXB(p));
    else pushup(DXB(p), mid + 1, nr);
    pushup(p, nl, nr);
}
```

[答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/CodeForces/920F.cpp)

这两道题可以将模型抽象如下：

```
1. 将 [a, b] 的每个值改为 f(a), f(a + 1), ..., f(b), 其中 f(x) 满足收敛性质；
2. 查询 [a, b] 的总和。
```

#### 2. 线段树的多重标记问题

线段树如果它需要更新多重标记，我们应该怎么办？本节我们来探讨一下这个问题。

例题：[BZOJ 1798](http://www.lydsy.com/JudgeOnline/problem.php?id=1798)，我们需要更新这个数列，需要以下几种操作：

* 把数列中的一段数全部乘一个值；

* 把数列中的一段数全部加一个值；

* 询问数列中的一段数的和，对`mod`取模。

在这里，我们需要知道的是，如果对于一段区间的所有数乘法的话，会影响这个数列的加的值。所以，我们的线段树中使用乘法和加法两种标记。加法操作和询问操作不变，乘法操作每次要把加法标记和区间和都乘上相应的数，下传标记时先下传乘法标记再下传加法标记。

证明可以参考下面图片：![](/PIC_Day6_4.png)

现在的 update 函数需要修改，query 函数修改的地方不多，这里不再赘述。

这是我写的 update 和 pushdown 函数：

```cpp
void pushdown(int p, int l, int r)
{
    if(tree[p].add == 0 && tree[p].mul == 1)return ;
    int mid = midf(l, r);
    tree[DXA(p)].add = (tree[DXA(p)].add * tree[p].mul + tree[p].add) % mod;
    tree[DXB(p)].add = (tree[DXB(p)].add * tree[p].mul + tree[p].add) % mod;
    tree[DXA(p)].mul = (tree[p].mul * tree[DXA(p)].mul) % mod;
    tree[DXB(p)].mul = (tree[p].mul * tree[DXB(p)].mul) % mod;
    tree[DXA(p)].sum = (tree[DXA(p)].sum * tree[p].mul + (mid - l + 1) * tree[p].add) % mod;
    tree[DXB(p)].sum = (tree[DXB(p)].sum * tree[p].mul + (r - mid) * tree[p].add) % mod;
    tree[p].add = 0;
    tree[p].mul = 1;
}

void update_add(int l, int r, int nl, int nr, ll a, int p)
{
    if(l <= nl && nr <= r)
    {
        tree[p].add = (tree[p].add + a) % mod; //注意与 update_mul 的比较
        tree[p].sum = ((nr - nl + 1) * a + tree[p].sum) % mod;
        return ;
    }
    pushdown(p, nl, nr);
    int mid = midf(nl, nr);
    if(l <= mid)update_add(l, r, nl, mid, a, DXA(p));
    if(mid < r) update_add(l, r, mid + 1, nr, a, DXB(p));
    pushup(p, nl, nr);
}


void update_mul(int l, int r, int nl, int nr, ll m, int p)
{
    if(l <= nl && nr <= r)
    {
        tree[p].mul = (tree[p].mul * m) % mod;//注意与 update_mul 的比较
        tree[p].add = (tree[p].add * m) % mod;
        tree[p].sum = (tree[p].sum * m) % mod;
        return ;
    }
    pushdown(p, nl, nr);
    int mid = midf(nl, nr);
    if(l <= mid)update_mul(l, r, nl, mid, m, DXA(p));
    if(mid < r) update_mul(l, r, mid + 1, nr, m, DXB(p));
    pushup(p, nl, nr);
}
```

练习题：[ZOJ 3998](http://acm.zju.edu.cn/onlinejudge/showProblem.do?problemCode=3998) 需要使用 Euler 降幂公式：$${a^{b^c}} {\rm {MOD}} m = a^{{b^c} {\rm {MOD}}{phi}(m)} {\rm {MOD}} {m}$$

如果出现了三重标记怎么办？[HDU 4578](http://acm.hdu.edu.cn/showproblem.php?pid=4578)

#### 3. 线段树区间求最大

我们将这个问题先简化一下，我们只需要求一个区间的连续和最大。

例题：[SPOJ GSS1](http://www.spoj.com/problems/GSS1/)，我们看一下样例：

```
3 
-1 2 3
1
1 2
```

对区间$$[1, 2]$$而言：$$[1, 1]$$的区间和为`-1`，$$[1, 2]$$的区间和为`1`，$$[2, 2]$$的区间和为`2`，样例的答案显然是`2`。

我们需要维护左区间连续和，右区间连续和，区间和，区间和最大值：

```cpp
typedef struct
{
	ll lsum, rsum, sum, maxsum;
}node;
```

如何维护结点呢？看看下面的 pushup 函数就明白了：

```cpp
void pushup(int p, int l, int r)
{
	if(l < r)
	{
		tree[p].sum = tree[DXA(p)].sum + tree[DXB(p)].sum;
		tree[p].lsum = max(tree[DXA(p)].lsum, tree[DXA(p)].sum + tree[DXB(p)].lsum);
		tree[p].rsum = max(tree[DXB(p)].rsum, tree[DXB(p)].sum + tree[DXA(p)].rsum);
		tree[p].maxsum = max(max(tree[DXA(p)].maxsum, tree[DXB(p)].maxsum), 
		                  tree[DXA(p)].rsum + tree[DXB(p)].lsum);
	}
}
```

区间和最大必须需要返回一个`node`型，要不然会出错：

```cpp
node query(int l, int r, int nl, int nr, int p)
{
	if(l <= nl && nr <= r)
	{
		return tree[p];
	}
	int mid = midf(nl, nr);
	if(mid < l)
		return query(l, r, mid + 1, nr, DXB(p));
	if(r <= mid)
		return query(l, r, nl, mid, DXA(p));
	node a, b, c;
	c = query(l, r, mid + 1, nr, DXB(p));
	b = query(l, r, nl, mid, DXA(p));
	a.maxsum = max(max(b.maxsum, c.maxsum), b.rsum + c.lsum);
	a.lsum = max(b.lsum, b.sum + c.lsum);
	a.rsum = max(c.rsum, c.sum + b.rsum);
	return a;
}
```

[SPOJ GSS1 答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/781d6f0d04cd12a7ae1e6f4cf1534d5232276c04/SPOJ/Gss1.cpp)

例题：[SPOJ GSS3](http://www.spoj.com/problems/GSS3/)，这道题就是需要在 GSS1 的基础上补充一个单点更新函数：

```cpp
void update(int x, int nl, int nr, ll val, int p)
{
	if(nl == nr)
	{
		tree[p].sum = tree[p].rsum = tree[p].lsum = tree[p].maxsum = val;
		return ;
	}
	int mid = midf(nl, nr);
	if(x <= mid)update(x, nl, mid, val, DXA(p));
	else if(mid < x)update(x, mid + 1, nr, val, DXB(p));
	pushup(p, nl, nr);
}
```

[SPOJ GSS3 答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/ecb05a6d5b70d8dbefe388b90fd2534674e3b140/SPOJ/Gss3.cpp)

练习：[BZOJ 1756\(权限题\)](http://www.lydsy.com/JudgeOnline/problem.php?id=1756)

#### 4. 线段树的一些骨骼清奇题目

以为 ZOJ 3998 和 ZOJ 4009 一样...结果一写才发现不太对劲...

