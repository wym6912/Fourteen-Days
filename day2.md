## 第 2 天

作为一名提高班的学员，听了毛老师讲的东西，嗯...还和昨天一样，好晕啊。

[提高班第 2 天练习题](https://vjudge.net/contest/176016)

[入门班第 3 天练习题](https://vjudge.net/contest/176369)

今天讲的是分治。下面就是正题了。

#### 1.二分

二分是什么？二分就是对于一个问题，抽象一个函数$$f(x)$$ ， 利用 "折半" 的方法，查找**需要的值。**

![](/PIC_Day2_1.jpg)如上图，Ans\_Place 是需要的值，l 是区间左端，r 是区间右端，每次都是**比较**两边的值，然后再不断缩小区间，这样一定可以得到结果的。

例题：[PPOJ 46 题](http://115.159.222.170/problem/46) ，这道题是一个最基础的二分题了，构造函数$$f(x)$$ = 割下的长度，那么这就是一道裸的二分题了。

[练习题1](https://vjudge.net/contest/194743)

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

![](/PIC_Day2_3.jpg)性质1：一定选择奇数个数。若选择偶数个数则可将中间两个数较大的一 个删除使得答案更优

性质2：枚举中位数，按照这样的方法枚举可以得到唯一结果

这道题需要注意的问题是：需要限制枚举个数\(Time Limit Exceed On Test 7\), 用 double 枚举会错\(Wrong Answer On Test 12\), 最好用前缀和计算上式中的 sigma 的值\(可能也会 T 吧\)

