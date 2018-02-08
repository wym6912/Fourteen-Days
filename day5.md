## 第 5 天 线段树\(I\) 线段树绪论 点修改 区间修改

今天是来自 XDU 的 V8 老师，他讲的是博弈论和单调队列。能听懂简单的博弈论，搞不懂单调队列。真的气。

[提高班第 5 天练习题](https://vjudge.net/contest/176855)

[入门班第 7 天练习题](/vjudge.net/contest/177478)

于是我...就去看了看线段树了。

开始今天的正题吧。

#### 1. 线段树概论

线段树是一种可以修改或者查询\(离散的\)区间上的一些信息，进行查询。

在一类问题中，我们需要经常处理可以映射在一个坐标轴上的一些固定线段，例如说映射在OX轴上的线段。由于线段是可以互相覆盖的，有时需要动态地取线段的并，例如取得并区间的总长度，或者并区间的个数等等。一个线段是对应于一个区间的，因此线段树也可以叫做区间树。

线段树是一棵二叉树，树中的每一个结点表示了一个区间$$[a, b)$$。每一个叶子节点表示了一个单位区间。对于每一个非叶结点所表示的结点$$[a, b)$$，其左儿子表示的区间为$$[a, \frac {a+b} {2})$$，右儿子表示的区间为$$[ \frac {a+b} {2}, b)$$。 见下图所示。

![](/PIC_Day5_1.png)

注意我的实现是表示一个$$[a, b]$$的区间，左儿子是$$[a, \frac {a + b} {2}]$$，右儿子是$$(\frac {a + b} {2}, b]$$。

可以证明任意区间 \(1到N范围内\) 最多只需要$$O(\log {N})$$个线段树的节点区间拼接而成，加上线段树的深度是$$O(\log {N})$$,所以上述以及类似区间型操作可以在对数时间内完成。对于上图，我们需要 17 个结点保存$$[1, 10]$$内的信息。可以证明，我们最多需要$$4 \times N$$个结点保存。

线段树的每个节点上往往都增加了一些其他的域。在这些域中保存了某种动态维护的信息，视不同情况而定。这些域使得线段树具有极大的灵活性，可以适应不同的需求。

在第 5 天和第 6 天中，我们将线段树按照查询和修改的情况，分为  点修改区间查询 、区间修改点查询、区间修改区间查询、RMQ与LCA、线段树问题转换 等方面。

那么，我们开始走进线段树的旅程吧。

#### 2. 线段树的单点修改、区间查询

问一个很简单的问题：为什么线段树不能做单点修改单点查询？\(数组对于这种操作只需要$$O(1)$$，而线段树则需要$$O(\log {N})$$，太不划算了233333\)

而在单点修改以后，如果要查询区间的和，数组就显得非常不便了。\(数组需要$$O(N)$$，而线段树则需要$$O(\log{N})$$\)

所以我们采用线段树这个结构作为这一类型题的解答。

话不多说，我们先看一道例题。

例题：[HDU 1166](http://acm.hdu.edu.cn/showproblem.php?pid=1166)，这一道题是线段树的单点修改和区间查询。这一道题询问的是区间的和。那么问题来了：怎么去实现这个结构？

首先，我们需要对问题进行分析：

在询问一段区间$$[4, 7]$$的时候，如下图：

![](/PIC_Day5_2.png)

可以不需要访问 4, 5, 6, 7 即可知道结果。

如果修改 7 怎么办？注意，这些题的实现可能与图不符。

![](/PIC_Day5_3.png)

现在，我们来实现这个结构吧！

在我的代码中，有以下定义宏：

```cpp
# define midf(a, b) (((a) + (b)) >> 1)
# define DXA(_) ((_) << 1)
# define DXB(_) (((_) << 1) | 1)
```

定义一个结构体：

```cpp
typedef struct
{
    int sum; //表示一段区间的和 [l, r]
}node;
```

我们需要开出比范围要大 4 倍的数组：

```cpp
node tree[max_node * 4]; //max_node 表示最多需要维护多少个结点
```

我们可以发现一个性质：如果`l == r`，递归结束，这个结点表示这一段区间的和。则预处理$$n$$个数按照这样写：

```cpp
void pre(int l, int r, int p) //表示[l, r] 区间，在线段树上的位置是 p
{
    if(l == r){scanf("%d", &tree[p].sum);return ;}
    else
    {
        int mid = midf(l, r);
        pre(l, mid, DXA(p));     //在我的代码里，左儿子表示[l, (l + r) >> 1]
        pre(mid + 1, r, DXB(p)); //右儿子表示[((l + r) >> 1) + 1, r]
        pushup(p, l, r);
    }
}
```

pushup 的功能在后面会提及。

现在我们需要查询一段区间的和，我们这样写 query 函数：

```cpp
long long query(int l, int r, int nl, int nr, int p) // 在 [l, r] 查询区间和，当前区间是 [nl, nr]，在线段树第 p 位
{
    if(l <= nl && nr <= r)
        return tree[p].sum;
    long long ans = 0;
    int mid = midf(nl, nr);
    if(l <= mid)ans += query(l, r, nl, mid, DXA(p));
    if(mid < r) ans += query(l, r, mid + 1, nr, DXB(p));
    return ans;
}
```

现在我们来完成单点修改 update 函数：（并没有把标记修改到这些区间上，只修改了点）

```cpp
void update(int nl, int nr, int px, int ans, int p)
{
    if(px == nr && nl == px)
    {
        tree[p].sum += ans;
        return ;
    }
    else
    {
        int mid = midf(nl, nr);
        if(px <= mid)update(nl, mid, px, ans, DXA(p));
        if(mid < px) update(mid + 1, nr, px, ans, DXB(p));
        pushup(p, nl, nr); //一定要注意更新的区域
    }
}
```

pushup 函数， 它的作用是把子树上的和更新到这个点：

```cpp
void pushup(int p, int l, int r)
{
    if(l < r)
        tree[p].sum = tree[DXA(p)].sum + tree[DXB(p)].sum;
}
```

再把主函数一些，这道题就完成了。

[HDU 1166 答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/12f32f6d399d4a88fed9c5b31e353bf558d67804/HDU/1166.cpp)

例题：[HDU 1754](http://acm.hdu.edu.cn/showproblem.php?pid=1754)，这次我们需要维护的是最大值了。我们只需要修改 pushup 函数和 query 函数即可。

```cpp
void pushup(int p)
{
    tree[p].sum = max(tree[DXA(p)].sum, tree[DXB(p)].sum);
}
```

query 函数应该返回的是最大值。

```cpp
int query(int l, int r, int nl, int nr, int p)
{
    if(l <= nl && nr <= r)
        return tree[p].sum;
    else 
    {
        int ans = -INF; //需要将 ans 赋初值
        int mid = midf(nl, nr);
        if(l <= mid)
            ans = max(ans, query(l, r, nl, mid, DXA(p)));
        if(mid < r)
            ans = max(ans, query(l, r, mid + 1, nr, DXB(p)));
        return ans;
    }
}
```

[HDU 1754 答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/12f32f6d399d4a88fed9c5b31e353bf558d67804/HDU/1754.cpp)

练习题：[PPOJ 135](http://ppoj.ac.cn/problem/135)，我们需要维护$$[1, 60000]$$内的信息。`a = 1`表示能力值为$$p$$的人数加1，`a = 2`表示能力值为$$p$$的人数减1，`a = 3`表示查询$$(p, 60000]$$的人数。可以利用线段树维护。[答案](http://ppoj.ac.cn/submission/1117)

练习题：[UVA12299](https://uva.onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&page=show_problem&problem=3720)



