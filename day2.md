## 第 2 天

作为一名提高班的学员，听了毛老师讲的东西，嗯...还和昨天一样，好晕啊。

[提高班第 2 天练习题](https://vjudge.net/contest/176016)

[入门班第 3 天练习题](https://vjudge.net/contest/176369)

今天讲的是分治。

#### 0.分治概论

分而治之，将原问题不断划分为若干个子问题，直到子问题规模足够小可以**直接**解决

子问题间互相独立且与原问题**形式相同**，递归求解这些子问题，然后将各子问题的解合并得到原问题的解。

分治三步：

1. 划分
2. 解决
3. 合并

下面就是正题了。

#### 1.二分

二分是什么？二分就是对于一个问题，抽象一个函数$$f(x)$$ ， 利用 "折半" 的方法，查找**需要的值。**

![](/PIC_Day2_1.jpg)如上图，Ans\_Place 是需要的值，l 是区间左端，r 是区间右端，每次都是**比较**两边的值，然后再不断缩小区间，这样一定可以得到结果的。

例题：[PPOJ 46 题](http://115.159.222.170/problem/46) ，这道题是一个最基础的二分题了，构造函数$$f(x)$$ = 割下的长度，那么这就是一道裸的二分题了。

[练习题1](https://vjudge.net/contest/194743)

xjb整理的...C 和 D 都需要用主席树， gg

A 题是一道可以抽象成函数$$f(x) = num(\exists i, j, |data_i - data_j| = x)$$，$$num()$$表示满足条件的对数。我需要什么值呢？一共有$$\begin {pmatrix} n \\ 2 \end {pmatrix}$$对，然而根据题的性质，我们只需比较$$\frac{\begin {pmatrix} n \\ 2 \end {pmatrix}}{2}$$与$$f(x)$$的关系。

B 题也是一样的，可以抽象一个函数$$f(x) = \sum \frac{r[i] ^  2} {x}$$，这道题的问题在于精度问题，4位小数，还有注意$$\pi$$和$$eps$$的取值，$$\pi$$必须在外面乘（感觉是因为$$\pi$$的精度不够），$$eps$$应该取$$1e-6$$

C D 两道题使用前面的方法是不行的，复杂度是$$O(qn log n)$$，肯定过不了的（感谢 jlx 指出）

#### 2.三分

三分的原理其实和二分一样，我们需要去找什么呢？这次是要找函数的**极值**了。

![](/PIC_Day2_2.jpg)为什么是极值？因为三分可以通过两个点反映这个函数的增减性。

注意这一次怎么缩小边界呢？

```cpp
if(f[mid] > f[mmid])
    R = mmid;
if(f[mid] <= f[mmid])
    L = mid;                //函数是凸函数，也就是求极大值
```

怎么求极小值呢？

```cpp
if(f[mid] <= f[mmid])   
     R = mmid;
if(f[mid] > f[mmid])
    L = mid;                //函数是凹函数，也就是求极小值
```

例题1：[HihoCoder 1142](https://hihocoder.com/problemset/problem/1142)

真·三分裸题

$$f(x) = ax ^ 2 + bx + c$$

利用三分就可以直接做了。

例题2：[CodeForces 578C](http://codeforces.com/problemset/problem/578/C)

给定一个序列$$A$$，一个区间的$$poorness$$定义为这个区间内和的绝对值 ，$$weakness$$等于所有区间最大的$$poorness$$

求一个$$x$$使得，序列$$A$$全部减$$x$$后$$weakness$$最小

这道题我们应该怎样抽象一个函数呢？

$$f(x) = \sum {(data[i] - x)}$$

但是在计算区间的时候，我发现了一个问题：你必须要将正负的所有值记录下来，每次都要比较（因为可能单点会比较大），否则会 gg

例题3：[CodeForces 626E](http://codeforces.com/problemset/problem/626/E)

抽象一个函数

$$f(x, i) = \sum_{x - i <= k <= x + i, n - x + 1 <= k <= n}{(data[k])} - data[x]$$

这个函数的性质有哪些？

![](/PIC_Day2_3.jpg)性质1：一定选择奇数个数。若选择偶数个数则可将中间两个数较大的一个删除使得答案更优

性质2：枚举中位数，按照这样的方法枚举可以得到唯一结果

这道题需要注意的问题是：需要限制枚举个数\(Time Limit Exceeded On Test 7\), 用 double 枚举会错\(Wrong Answer On Test 12\), 最好用前缀和计算上式中的 sigma 的值\(可能也会 T 吧\)

（反正不会告诉你我错了 14 遍）

#### 3.归并排序

为什么要把归并排序放到这里来说呢？因为归并排序每次把待排序区间一分为二，将两个子区间排序，然后将两个已经排好序的序列合并，符合二分的性质。它的复杂度稳定的$$O(n log n)$$

归并排序的代码如下：

```cpp
long long __merge__(int l, int mid, int r)  // return 的值表示[l, r]逆序对的个数
{
    ll ans = 0; 
    int p1 = l, p2 = mid + 1;
    for(int i = l;i <= r;i ++)
    {
        if(p1 <= mid && (p2 > r || data[p1] <= data[p2]))
            tmp[i] = data[p1], p1 ++;
        else tmp[i] = data[p2], p2 ++, ans += (mid - p1 + 1);
    }
    for(int i = l;i <= r;i ++)data[i] = tmp[i];
    return ans;
}

long long erfen(int l, int r) // return 的值表示[a, b]逆序对的个数
{
    long long ans = 0;
    int mid = (l + r) >> 1;
    if(l < r)
    {
        ans += erfen(l, mid);
        ans += erfen(mid + 1, r);
    }
    ans += __merge__(l, mid, r);
    return ans;
}
```

模板题：[HihoCoder 1141](https://hihocoder.com/problemset/problem/1141)

简单变形题：[HDU 4911](http://acm.hdu.edu.cn/showproblem.php?pid=4911)

#### 4.点分治

emmm...不太会（毛老师窝对不起你）

#### 5.树分治

emmm...不太会（毛老师窝对不起你）

#### 6.cdq 分治

这是一个比较高级的数据结构（emm...还没看...）

今天的分治旅程就到这里。我们明天见！

