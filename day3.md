## 第 3 天 搜索

第 3 天讲授的是搜索和模拟。今天毛老师讲的并不是很多，11 点就下课了。

模拟由于专题过大，就不太想写了（其实是不会...）。

[提高班第 3 天练习题](https://vjudge.net/contest/176280)

[入门班第 4 天练习题](https://cn.vjudge.net/contest/176664)

[入门班第 5 天练习题](https://cn.vjudge.net/contest/176912)

[入门班第 6 天练习题](https://vjudge.net/contest/177296)

那么，我们从搜索开始吧。

#### 0. 搜索概论

搜索，就是暴力枚举各个**可达**结果，选择符合条件的结果，并将这些结果**按一定规则**保存起来。

结构是这样的：

1. 选择某一种可能情况向前探索。
   1. 探索过程中，若发现原来的选择是错误的，就退回一步重新选择，继续向前探索。
2. 反复进行1，直至得到解或证明无解。

如果这些话比较难理解的话，举个形象的例子：

* 迷宫问题：进入迷宫后，先随意选择一个前进方向，一步步向前试探前进,如果碰到死胡同，说明前进方向已无路可走，这时，首先看其它方向是否还有路可走，如果有路可走，则沿该方向再向前试探；如果已无路可走，则返回一步，再看其它方向是否还有路可走；如果有路可走，则沿该方向再向前试探。按此原则不断搜索回溯再搜索，直到找到新的出路或从原路返回入口处无解为止。

搜索的分类：

1. 盲目搜索：   \(按预定的控制策略进行搜索，在搜索过程中获得的中间信息不用来改进控制策略。\)
   1. 广度优先搜索（Breadth First Search）    
   2. 深度优先搜索（Depth First Search）   
   3. 纯随机搜索、重复式搜索、迭代加深搜索、迭代加宽搜索、柱型搜索 
2. 启发式搜索：    \(在搜索中加入了与问题有关的启发性信息，用以指导搜索朝着最有希望的方向发展，加速问题的求解过程并找到最优解。\)
3. 1. A\*算法
   2. IDA\*算法

本文重点讲述 1.i 和 1.ii。

#### 1. 深度优先搜索（Depth First Search，简称 DFS）

定义：在搜索时以深度为优先，进行扩展答案的一种方法。（拿了以前的框架。。。）

DFS算法框架\(I\)

```cpp
int DFS(int k) 　
{ 　
    for (i = 1;i <= 算符种数;i ++) 　　
        if (满足条件) 　　   
        { 　　　　
            保存结果 　　　　
            if (到目的地) 输出解; 　　　           
            else DFS(k + 1); 　　　　
            恢复：保存结果之前的状态{回溯一步} 　 　  
        } 　
}
```

DFS算法框架\(II\)

```cpp
int DFS(int k) 　
{ 　  
    if  (到目的地) 输出解;
    else 　　　　
        for (i = 1;i <= 算符种数;i ++) 　　　　　
            if  (满足条件)  　　　　　　　
            {
                保存结果;
                DFS(k + 1); 　　　　　　　　
                恢复：保存结果之前的状态{回溯一步} 　　　　　　　
            } 　
}
```

当然，如果这样的话，它的时间复杂度是$$O(Status ^ n)$$，复杂度虽说比$$O(n!)$$稍小一点，但是这个复杂度仍然让人无法接受。

例题：[HDU 1016](http://acm.hdu.edu.cn/showproblem.php?pid=1016)，素数环：从`1`到`20`这`20`个数摆成一个环，要求相邻的两个数的和是一个素数。递归填数：判断第$$i$$个数填入是否合法。

这个时候，就需要**剪枝**了。

剪枝大致分为 可行性剪枝 和 最优化剪枝。

可行性剪枝 就是尽早将不可能到达的点剪去，以在**保证正确性**的前提下，快速完成搜索的目的。

例题：[HDU 2553](http://acm.hdu.edu.cn/showproblem.php?pid=2553) ，多皇后问题：皇后能吃同一行、同一列、同一对角线的任意棋子，如何放置才能让他们不互相吃？

以 8 皇后为例，显然问题的关键在于如何判定某个皇后所在的行、列、斜线上是否有别的皇后；可以从矩阵的特点上找到规律，如果在同一行，则行号相同；如果在同一列上，则列号相同；如果同在／ 斜线上的行列值之和相同；如果同在＼ 斜线上的行列值之差相同，利用下图可以验证。

![](/PIC_Day3_1.jpg)

考虑每行有且仅有一个皇后，设一维数组A\[8\]\[8\]表示皇后的放置：第i行皇后放在第j列，用A\[i\]=j来表示，即下标是行数，内容是列数。例如：A\[3\]=5就表示第3个皇后在第3行第5列上。

[答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/HDU/2553.cpp)

最优化剪枝就是如果当前结果已经无法达到已存储的结果，可以剪枝的一种方法。

例题：[PPOJ 12  ](http://ppoj.ac.cn/problem/12)，初看是个图论题。但是没想到该怎么做，于是gg。但是看到数据范围，窝开心了：爆搜就行了！

~~先打了个完全的爆搜， T 了。~~

然后。。。可行性剪枝来了！如果这个人已经有敌人了，我就不加这个点，这样一定不会加错！（如果加了是错的，还不如在一开始就不加，你说对吧？）这样是不能过的。但是如何更好地优化？

下来。。。最优化剪枝！直接 x + n - p &lt;= maxp，这样就可以了。（[Submission](http://ppoj.ac.cn/submission/961)）

但是可行性剪枝有更好的方法。有一种巧妙的方法进行剪枝：如果加入了一个人，利用`int`型的判敌数组 isenemy\[101\]，如果加入了一个人，就把他的敌人的 isenemy 值 + 1。为什么这样做呢？因为这样在 恢复 过程中，可以很容易进行撤销： isenemy 值 - 1 即可。（[Submission](http://ppoj.ac.cn/submission/926)）

所以，巧妙的可行性剪枝和最优化剪枝可以将一些无法过的爆搜改过。（~~又称 aqx 搜索~~）

有技巧的枚举 可以将问题简化。

例题1：[POJ 1564 ](http://poj.org/problem?id=1564)，首先将数字排序，然后按数字从大到小强行枚举。（发现 ACM 选修老师竟然讲过2333333）

![](/PIC_Day3_2.jpg)

结点记录当前剩余\(根等于所输入的$$x = y$$\)。第$$i-1$$层结点\(根是0层\)的左孩子表示取第$$i$$个数，右孩子示不取；左孩子记录\($$t_i - 1 - {num} _ i$$\)。若某结点值等于0，从根到这个结点的路径就表示了一个解。

（当时想到要按照倒序枚举，没想到可以这样简化状态的描述233333）

例题2：[POJ 1011](http://poj.org/problem?id=1011) ，（感谢雨姐姐2333），如果第$$i$$个棍子不能拼成假设的长度，则和第$$i$$个棍子相同长度的棍子也是不可能的，所以可以直接跳过去的！将所有题目给的棍子的长度按照从大到小的顺序排列，然后按照此顺序进行深搜。

**更新**：这一道题也是很好的一道爆搜题：[CodeForces 915C](http://codeforces.com/contest/915/problem/C)，枚举一个数（利用上面的每个数位）在比下面的数小的前提下，尽可能最大。需要估算这个数，如果当前的数已经比要求的数大了，就剪枝。

练习：数独问题 [POJ 2676 ](http://poj.org/problem?id=2676), [~~POJ 3074~~](http://poj.org/problem?id=3074)（Dancing Links）

#### 2.广度优先搜索（Breadth First Search，简称 BFS）

定义：在扩展状态时，以优先扩展浅层节点，逐渐深入  
进行扩展的过程。

BFS算法框架

```cpp
BFS()
{
    初始化队列
    while(队列不为空且未找到目标节点)
    {
        取队首节点扩展，并将扩展出的节点放入队尾；
        必要时要记住每个节点的父节点；
    }
}
```

注意将已经过的结点进行判重。怎么判重？这里可以提供一些比较好的方法。

* 合理编码，减小存储代价
  * 对于全排列的搜索，可以使用 Cantor 展开，可以实现一一对应。
  * 直接编码。
  * 针对字符串的一些hash算法，比如ELFHash和BKDRHash。

例题1：八数码难题 [POJ 1077](http://poj.org/problem?id=1077) & [HDU 1043](http://acm.hdu.edu.cn/showproblem.php?pid=1043)，不得不佩服那个写了 8 个境界的人。

这道题需要利用 Cantor 展开，还需要将搜索进行一定的优化。如果你每次都要搜的话，POJ 1077 只有一组数据，但是 HDU 1043 有多组数据，一定会超时的。所以，你可以先预处理所有可达结点，然后直接输出路径即可。（为什么这样可以？）

例题2：迷宫问题，直接扩展结点即可。

#### 3.分层图搜索

在一些迷宫问题中，有时会有一些锁，我们必须去用钥匙打开，这个时候可以使用分层图搜索来解决这一类问题。

原理就是以下图形：

![](/PIC_Day3_3.jpg)

但是在实际处理时，我们需要对结点进行更多的标记。

```cpp
struct Node
{
    int x, y;
    Status s1, s2, ...;
};
```

还需要对状态判重（为什么？因为在一张图中一个点需要走多次，然而 `Status`却不一定一样，有可能你走了很多遍但是钥匙状态不一样）。

需要一个

```cpp
bool Stat[maxn][maxm][State1_Number][State2_Number]...;
```

进行状态判重。有几种状态就加几维。

例题：[HDU 4845](http://acm.hdu.edu.cn/showproblem.php?pid=4845)，是网络流 24 题中的一道很经典的题。

将所有钥匙存成一个`int`型的整数，利用位运算将每把钥匙取出。核心代码如下：

```cpp
int bfs(int fx, int fy)
{
    point temp, init;
    queue <point> qqq;
    init.x = fx;
    init.y = fy;
    init.keys = keys[fx][fy];
    init.step = 0;
    qqq.push(init);
    while(! qqq.empty())
    {
        init = qqq.front();
        if(init.x == n && init.y == m)return init.step;
        for(int i = 0; i <= 3; i ++)
        {
            if(init.x + dx[i] > 0 && init.x + dx[i] <= n 
            && init.y + dy[i] > 0 && init.y + dy[i] <= m)
            {
                temp.x = init.x + dx[i];
                temp.y = init.y + dy[i];
                temp.keys = init.keys | keys[temp.x][temp.y];
                temp.step = init.step + 1;
                if(walls[init.x][init.y][temp.x][temp.y] == -1)
                    continue;
                if(walls[init.x][init.y][temp.x][temp.y] == 0 //这里可以直接通过的！！ 
                ||(temp.keys >> walls[init.x][init.y][temp.x][temp.y]) & 1)
                    if(! stat[temp.x][temp.y][temp.keys]) //必须优化掉这里，会爆空间的（重复走这个状态） 
                    {
                        qqq.push(temp);
                        stat[temp.x][temp.y][temp.keys] = true;
                    }
            }
        }
        qqq.pop();
    }
    return -1;
}
```

[答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/HDU/4845.cpp)

习题：[PPOJ 3](http://ppoj.ac.cn/problem/3)，（答案：[Submission](http://ppoj.ac.cn/submission/992)）；[HDU 4771](http://acm.hdu.edu.cn/showproblem.php?pid=4771)（[答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/HDU/4771.cpp)），[HDU 5025](http://acm.hdu.edu.cn/showproblem.php?pid=5025)（[答案](https://github.com/wym6912/ACM-ICPC_wym6912/blob/master/HDU/5025.cpp)）

#### 4.双向搜索

emmm...不太会（毛老师窝对不起你）

今天的搜索旅程就到这里。明天我们继续！

