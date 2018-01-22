## 第 1 天 绪论 C++11 新特性

2017 年 7 月 15 日。终于考完了所有课程。那个让我喜欢的女生，和别的男生在一起了。心好痛。

2017 年 7 月 20 日。出了最后一门成绩。果然还是很砸。

那几天，虽然身在学校，但是心里感觉很空很空。

直到 7 月 31 号的 CF... 不说了，太惨了。

2017 年 8 月 1 日，我们正式开始了集训。

第一天毛老师讲了两个东西：

* 代码规范
* 思维导论

但是，我对 C++11 的新特性感兴趣。（hhhh...）

下面进入正题。

### 0. 编译前提

使用`g++`编译的时候，需要使用以下命令：`g++ -std=c++11`

在提交 OJ 上的时候，需要将语言选为 C++11 。

### 1. 类型推导

主要是在 Ar 老师的代码里经常看到。（但是这样省事啊）

例如以下代码：

```cpp
vector <int> vec;
vector <int> :: iterator it = vec.begin();
```

可以利用`auto`关键字简化为以下代码：

```cpp
vector <int> vec;
auto it = vec.begin();
```

这样做的好处是将遍历一个 STL 集合变得简便：

例如以下代码：

```cpp
vector <int> vec;
for(vector <int> :: iterator it = vec.begin(); it != vec.end(); it ++)
{
    *it += 2;
    cout << *it << endl;
}
```

可以简化成如下代码：

```cpp
vector <int> vec;
for(auto & p: vec)
{
    p += 2;
    cout << p << endl;
}
```

### 2. 空指针值

利用`nullptr`关键字，将指针和数组的`NULL`区分开来。

### 3. STL 的初始化语法

在 C++11  中，将 `vector`和`map`等容器的初始化简化为数组形式的初始化。

例子：

```cpp
vector <string> vec{"1", "2", "3"};
map <string, string> dict{ {"ABC", "123"}, {"BCD", "234"} };
```

### 4. Hash 容器

利用以下容器可以减少时间复杂度的常数。它们都是利用 Hash 算法实现的。

`unordered_map`, `unordered_set`, `unordered_multimap`, `unordered_multiset`

注意它只能在不需要排序的时候可以提高查找性能。

