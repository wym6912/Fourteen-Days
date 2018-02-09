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

lazy 思想可以解决这个问题。再更新的地方如下所示：

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



