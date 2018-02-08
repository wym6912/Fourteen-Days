## 第 4 天 并查集

第 4 天毛老师讲的是图论。我一想，这个东西不是我搞的，所以就没有去听。

[提高班第 3 天练习题\(增加\)](https://vjudge.net/contest/176236)

[提高班第 4 天练习题](https://vjudge.net/contest/176665)

[提高班第 4 天练习题\(增加\)](https://vjudge.net/contest/209149) 六个月后增加

那么就开始今天的正题了。

#### 1. 基础的并查集

并查集是一种用于分离集合操作的抽象数据类型。它所处理的是**集合**之间的关系，即动态地维护和处理集合元素之间复杂的关系，当给出两个元素的一个无序对\(a, b\)时，需要快速**合并 **a 和 b 分别所在的集合，这其间需要反复**查找**某元素所在的集合。“并”、“查”和“集”三字由此而来。在这种数据类型中，n 个不同的元素被分为若干组。每组是一个集合，这种集合叫做分离集合\(disjoint set\)。

并查集支持查找一个元素所属的集合以及两个元素各自所属的集合的合并。

例题：亲戚。给出最多`1000000`条信息 \($$m$$\)和询问\($$q$$\)，包括`20000`个人\($$n$$\)的信息。请判断各对人是否有关系。

用传统的思路，可以马上反应过来，对于输入的$$n$$个点$$m$$条边，找出连通块，然后进行判断。但这种实现思路首先必须保存$$m$$条边，然后再进行普通的遍历算法，效率显然不高。

在下图情况应该怎么处理？"？"边是否需要处理？

![](/Pic_Day4_1.png)

用集合的思路，对于每个人建立一个集合，开始的时候集合元素是这个人本身，表示开始时不知道任何人是他的亲戚。以后每次给出一个亲戚关系时，就将两个集合合并。这样实时地得到了在当前状态下的集合关系。如果有提问，即在当前得到的结果中看两元素是否属于同一集合。

![](/PIC_Day4_2.png)

下图展示了并查集的基本操作。

![](/PIC_Day4_3.png)

![](/PIC_Day4_4.png)

由上图所说，这种算法的时间复杂度是$$O(n \alpha (n, n))$$，其中$$\alpha (n, n)$$是 Ackerman 函数的反函数，近似为常数。

并查集的实现分为以下几个过程：

类型定义：

```cpp
int father[10000];
```

初始化过程：

```cpp
for(int i = 1; i <= n; i ++)
    father[i] = i;
```

find\(x\) 过程，查找父亲节点的编号：

```cpp
int find(int x)
{
    return x == father[x] ? x : find(father[x]);
}
```

Union\(x, y\) 过程，表示合并 x 和 y 所在的集合：

```cpp
void unionn(int x, int y)
{
    x = find(x);
    y = find(y);
    //假定 x 和 y 不在一个集合
    father[x] = y;
}
```

并查集只是一个模型，我们需要解决一些其他问题。

例题：[POJ 1308](http://poj.org/problem?id=1308) & [HDU 1272](http://acm.hdu.edu.cn/showproblem.php?pid=1272)，我们需要判断这些边能不能组成树。这里需要利用并查集了。如果有一组边\(a, b\), a和b有边间接相连还要再连一下的话，就不满足树的定义了。见下图：

![](/PIC_Day4_5.png)

我们需要知道虚线边是否属于这个树。于是可以用并查集求解。

例题：[HDU 1213](http://acm.hdu.edu.cn/showproblem.php?pid=1213)，求一个无向图内有多少个连通块。也可以用并查集求解。

#### 2. 带权并查集

在一些并查集里，元素与元素之间存在一定的关系，它们不能简单地像上文所述那样进行合并。我们需要一个`rank`数组表示它们之间的关系。时间复杂度为 $$O(n \log {n})$$

例题：[POJ 1182](http://poj.org/problem?id=1182)，我们需要一个`rank`数组表示动物与动物之间的关系。rank\[x\] == 0 表示 father\[x\] 与 x 同类，1 表示 father\[x\] 吃 x, 2 表示 x 吃 father\[x\]。

判断假话的一个条件： \(rank\[y\] - rank\[x\] + 3\) % 3 != D - 1

注意这里的 find 函数和 unionn 函数

find\(x\)：

```cpp
int find(int x)
{
    if(x != father[x])
    {
        int t = father[x];
        father[x] = find(father[x]);
        rank[x] = (rank[x] + rank[t]) % 3; // 反映了 rank 数组的本质
    }
    return father[x];
}
```

unionn\(x, y\) ：

```cpp
void unionn(int x, int y, int d)
{
    //不能路径压缩！
    int xf = find(x), yf = find(y);
    father[xf] = yf;
    rank[xf] = (rank[y] - rank[x] + 3 + d) % 3;
}
```

练习题：[POJ 2492 ](http://poj.org/problem?id=2492)

[kuangbin 专题 5](https://vjudge.net/contest/166939)

今天的并查集旅程就到这里。我们明天再见！

