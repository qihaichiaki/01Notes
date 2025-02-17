---
link: https://blog.csdn.net/weixin_61508423/article/details/126296698
title: 【C++】从使用string类到模拟实现string类
description: 文章浏览阅读789次，点赞3次，收藏2次。本篇文章我会初步介绍string类的相关使用（结合文档的使用笔记），以及到后面如何去简单的模拟实现string类。_n/,*哦没
keywords: n/,*哦没
author: 柒海啦 Csdn认证博客专家 Csdn认证企业博客 码龄3年 暂无认证
date: 2022-08-20T08:12:19.000Z
publisher: null
stats: paragraph=61 sentences=20, words=138
---
## 前言

hi~大家好呀！(〃'▽'〃) 欢迎大家点击我的文章！

本篇文章我会初步介绍 **string**类的相关使用（ **结合文档**的使用笔记），以及到后面如何去简单的 **模拟实现string类**。

我所使用的C++帮助文档：[cplusplus.com - The C++ Resources Network](https://www.cplusplus.com/ "cplusplus.com - The C++ Resources Network")

string类虽然是实现在标准库内的，但是符合 **STL**（ **标准模板库**（也在标准库内））的规则的，学完string也能方便的为我们打开 **STL**学习的大门。

![](https://img-blog.csdnimg.cn/b6c38d48f6da4c1885527b0db3835a6b.png) 准备好了吗？❥(ゝω・✿ฺ) 启程！

---------------------------------------------------------------------------------------------------------------------------------

**目录**

[一、string类引入和使用](#%e4%b8%80%e3%80%81string%e7%b1%bb%e5%bc%95%e5%85%a5%e5%92%8c%e4%bd%bf%e7%94%a8)

[1.string类引入](#1string%e7%b1%bb%e5%bc%95%e5%85%a5)

[2.string类使用](#2string%e7%b1%bb%e4%bd%bf%e7%94%a8)

[1构造函数](#1%e6%9e%84%e9%80%a0%e5%87%bd%e6%95%b0)

[2运算符重载](#2%e8%bf%90%e7%ae%97%e7%ac%a6%e9%87%8d%e8%bd%bd)

[赋值重载](#%e8%b5%8b%e5%80%bc%e9%87%8d%e8%bd%bd)

[[]重载](#%5b%5d%e9%87%8d%e8%bd%bd)

[+=重载](#%2b%3d%e9%87%8d%e8%bd%bd%c2%a0)

[3push_back、append尾插](#3push-back%e3%80%81append%e5%b0%be%e6%8f%92)

[4isalpha检查是否为字符](#4isalpha%e6%a3%80%e6%9f%a5%e6%98%af%e5%90%a6%e4%b8%ba%e5%ad%97%e7%ac%a6)

[5iterator迭代器](#5iterator%e8%bf%ad%e4%bb%a3%e5%99%a8)

[范围for](#%c2%a0%e8%8c%83%e5%9b%b4for)

[6指定位置插入insert](#6%e6%8c%87%e5%ae%9a%e4%bd%8d%e7%bd%ae%e6%8f%92%e5%85%a5insert)

[7删除字符erase](#7%e5%88%a0%e9%99%a4%e5%ad%97%e7%ac%a6erase)

[​编辑](#%e2%80%8b%e7%bc%96%e8%be%91)

[8string的相关增容](#8string%e7%9a%84%e7%9b%b8%e5%85%b3%e5%a2%9e%e5%ae%b9)

[reserve开空间](#reserve%e5%bc%80%e7%a9%ba%e9%97%b4)

[resize开空间也可初始化](#resize%e5%bc%80%e7%a9%ba%e9%97%b4%e4%b9%9f%e5%8f%af%e5%88%9d%e5%a7%8b%e5%8c%96%c2%a0)

[9兼容c的字符串c_str()](#9%e5%85%bc%e5%ae%b9c%e7%9a%84%e5%ad%97%e7%ac%a6%e4%b8%b2c-str%28%29)

[10查找一个字符、字符串、string对象find\rfind](#10%e6%9f%a5%e6%89%be%e4%b8%80%e4%b8%aa%e5%ad%97%e7%ac%a6%e3%80%81%e5%ad%97%e7%ac%a6%e4%b8%b2%e3%80%81string%e5%af%b9%e8%b1%a1find%5crfind%c2%a0)

[substr取子字符串构造一个新的string对象](#substr%e5%8f%96%e5%ad%90%e5%ad%97%e7%ac%a6%e4%b8%b2%e6%9e%84%e9%80%a0%e4%b8%80%e4%b8%aa%e6%96%b0%e7%9a%84string%e5%af%b9%e8%b1%a1)

[11getline输入一行字符串](#11getline%e8%be%93%e5%85%a5%e4%b8%80%e8%a1%8c%e5%ad%97%e7%ac%a6%e4%b8%b2)

[12string下的类型转化](#12string%e4%b8%8b%e7%9a%84%e7%b1%bb%e5%9e%8b%e8%bd%ac%e5%8c%96)

[二、string类的模拟实现](#%e4%ba%8c%e3%80%81string%e7%b1%bb%e7%9a%84%e6%a8%a1%e6%8b%9f%e5%ae%9e%e7%8e%b0)

[1.明确实现思路](#1%e6%98%8e%e7%a1%ae%e5%ae%9e%e7%8e%b0%e6%80%9d%e8%b7%af)

[2.代码实现](#2%e4%bb%a3%e7%a0%81%e5%ae%9e%e7%8e%b0)

## 一、string类引入和使用

### 1.string类引入

![](https://img-blog.csdnimg.cn/693aba4ff6a1465b94c6d6601fc7dff1.png)

首先，上面就是库里面实现的类模板名，而从下面的string开始才是真正的类。

没错，类模板经过指定模板类型才能真正的成为自定义类型，在通过typedef进行了一下重命名，如string的来源：

![](https://img-blog.csdnimg.cn/c7ac4dd21fe6482a809f66922cfcc5cd.png)

所以这些类的本质就是通过类模板进行分配不同的模板类型而来的。

那么上面类中不同的区分又是怎么一回事呢？

strings就是字符串的意思，字符串也就是要代表文字语言呀。文字语言要用计算机里存储的信号表示出来的话，就必须要有一张 **参考图**。由于最早是美国研究出来的计算机，所以最早的一个参考图就是大家耳熟能详的ASCII码表，由于英文表示起来简单（26大小写字母组合起来而已），自然这张表也就很简单了。

但是计算机流行全球后，全球这么多国家自然需要不同的参考图。针对于此，世界上就有专门针对全球每个不同的语言进行转变为计算机识别的参考图 -UTF就诞生了。（详细可参考百度哦~）

不同的编码又和英文出现了很多区别，比如就我们中国汉字而言，仅仅一个字节储存的数字信息是概括不了中国基础汉字的，所以就有了用两个字节来储存汉字的信息，为了和ASCII码值配合，就有了UTF - 8。

![](https://img-blog.csdnimg.cn/afd9e4cc545f4447bbea656b75c8a7fd.png)

那么其他的类也就可以表示更多的语言了，自然就有了其他类的区分。我们这里就重点学习string类，可以发现是使用的 **char**类型进行类模板显示实例化的。

### 2.string类使用

#### 1构造函数

![](https://img-blog.csdnimg.cn/85842540b3f44992996fffdb1bcb2c95.png) 首先是默认构造函数： **string()**;

```cpp
#include
using namespace std;
//using std::string; //自然只展开string也是可以的

//string类存放在标准库中 std是标准库的命名空间 展开std就可以使用string类了

int main()
{
	string a;
	return 0;
}
```

默认即没有给任何参数就实例化一个此类的对象，就调用此类的默认构造函数，通过调试，我们可以发现：

![](https://img-blog.csdnimg.cn/a56112c4baa34a348cdb047f83f2df09.png)

此类对象实际有着三个成员变量，一个是类似于字符数组，用于存储字符串，还有就是size和capacity，一个存储当前字符个数，一个存储当前字符容量。可以看到，默认构造会给字符串"",即一个空的字符串，但是观察其原始视图不难发现，里面为了兼容c最后一个字符存储着'\0'作为字符串结束标志，但是注意size并没有显示，即size和capacity只显示 **有效字符个数**。

然后就是为了能让c字符串也能存入string里面，并且也能更加方便的进行隐式类型转化，这里便就有来自c字符串的构造函数： **string(const char* s)**;

![](https://img-blog.csdnimg.cn/66c8ec51428e4511ada0acfc15fa0758.png)![](https://img-blog.csdnimg.cn/65d8b203dd2544388b2ee58b14a0eb18.png)

这里也可以说明size只是表示 **有效字符**个数。并且观察原始视图，可以发现最后一个字符储存的就是'\0'。也就是说，实际储存的size是要比有效字符多一个的。

然后的那个局部拷贝构造函数实际上就是截取字符串上的一部分。 **npos**在类里定义为-1。由于类型是size_t即无符号的整形，所以就变为了整形的最大值。pos表示下标位置，即从字符串的哪个位置开始拷贝。 **综上，即传从哪里开始拷贝的位置，然后拷贝几个字符，如果len大于本身字符串的长度就全拷贝完，因为给的有缺省值npos，即如果不默认传拷贝几个字符的话，也就默认相当于从pos位置开始拷贝完。**

```cpp
int main()
{
	string a;
	string b = "hello world";
	string c("Tech otakus save the world!", 17, 3);
	string d("Tech otakus save the world!", 17, 12);
	string e("Tech otakus save the world!", 17);//注意这个会和string(const char* s, size_t n)重合，所以使用string类
	a = "Tech otakus save the world!";
	string f(a, 17);
	cout << c << endl;
	cout << d << endl;
	cout << e << endl;
	cout << f << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/b57cbfc53c944c5cba609bee16d48744.png)

这里的e对象由于传入的是两个参数，刚好和构造函数中的 **string(const char* s, size_t n)**完美匹配，所以e的意思就是将传入的字符串拷贝n个字节。上面的字符串也会隐式转换为string类型的哦，所以没有问题。

注意这里的string b = "hello world";这个可不是赋值重载，这个实际也是直接的显示构造哦~别忘了隐式类型转换，字符串会转化为对应的一个string类型，然后发生拷贝构造。即原本时构造+拷贝构造。但是经过编译器优化后，也就变成了单纯的构造咯。

析构函数实现的是深拷贝，实现时在细谈，目前不说。

那么这里的 **=** 就可以涉及到运算符重载的相关成员函数了：

#### 2运算符重载

#### 赋值重载

如上，a = "字符串";就是一个 **赋值重载**。

![](https://img-blog.csdnimg.cn/1a6f2382d64047d5bb39bdfbd102e3fa.png)

传入的参数有三类：即 **string对象，常量字符串，字符**。也就是常规的这些：

```cpp
int main()
{
	string a;
	a = 'a';
	cout << a << endl;
	a = "abc";
	cout << a << endl;
	string b("abcd");
	a = b;
	cout << a << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/64d8e582ab9442458f444a2a3f91e35e.png)

#### **[]重载**

除此之外，最重要的也是能够让string好用的成员函数就时 **[]重载**了，让我们能够像数组一样方便的去使用：

![](https://img-blog.csdnimg.cn/1569693bb2a8440bb7881f2b064b6ecd.png)

一个就是正常常规用的，另一个就是针对于 **const类型**使用的。由于其实现在类里，所以有默认的 **string* const this**为第一个参数，也就是[]的第一个操作符了：

```cpp
int main()
{
	string a = "abcde";
	for (size_t i = 0; i < a.length(); i++)
	{
		cout << a[i] << endl;
	}
	return 0;
}
```

![](https://img-blog.csdnimg.cn/8d3a9d9590d848c395aba1649bf8e7fe.png)

比如使用此就可以实现最常用的遍历数组，这里的 **length**也是其中的一个成员函数，能输出当前的size个数，当然，通用的是 **成员函数size**，因为其他 **STL容器里面可就没有length**这一说法了：

```cpp
	cout << a.size() << endl;
```

![](https://img-blog.csdnimg.cn/62e2b66826f54fb79d0772cd5f2870da.png)

当然，满足数组也就可以修改里面的字符数据啦。即[]重载就可以实现对string对象存储的字符串某处进行读写操作。

针对于const类型，重载了一个[]重载函数，便于const对象读：

```cpp
int main()
{
	const string a = "hi~";
	for (size_t i = 0; i < a.size(); i++)
		cout << a[i] << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/79e01f8e70cc4edd9de036367078639e.png)

需要注意的是数组里可不要越界哦，里面是有严格的越界检查的（assert()，断言检查）。

**at()成员函数**：

![](https://img-blog.csdnimg.cn/821f1d7d939448ad96800bcfa4358591.png)

此函数和[]的使用类似，会返回pos位置的字符哦，并且非const对象也可以进行修改：

![](https://img-blog.csdnimg.cn/8a4b67dcda674f32a20d1e0566508f89.png)

但是和[]的区别就是失败抛异常（越界）（捕获异常 程序继续运行），[]就直接断言了。2

#### +=重载

当你想在string对象后面在加点字符或者字符串的时候，+=重载极其好用：

![](https://img-blog.csdnimg.cn/71056fd5e3234715a6f720e2672253bf.png)

一个就是添加string对象，一个就是常量字符串，一个就是字符。

```cpp
int main()
{
	string a = "h";
	a += 'e';
	a += "ll";
	string b(1, 'o');
	a += b;
	cout << a << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/c455e8c2623f4d2080a65c64ca26da5c.png)

注意， **>、<**就表示string是支持比大小的，利用的是 **ASCII码值**进行的比大小。

#### 3push_back、append尾插

上面使用了+=重载，其实这个成员函数底层调用的就是push_back和append这两个成员函数。其中： **push_back尾插字符，append尾插字符串**：

![](https://img-blog.csdnimg.cn/c4bc370bd53f44b4a829c2f659394d96.png)![](https://img-blog.csdnimg.cn/ad1e93f136db41cca26028be7840a081.png)

比如，将上面的+=程序修改为用append和push_back进行实现：

```cpp
int main()
{
	string a = "h";
	a.push_back('e');
	a.append("ll");
	string b(1, 'o');
	a.append(b);
	cout << a << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/f8aa586d47a34ca79a7668e7bede7fb1.png)

append第二个实际上就是选取要准备追加的字符串的一部分，第一个参数表示起始位置，第二个表示追加几个字符过去，如果第二个参数没传的话会默认从pos位置开始将直到末尾的字符均追加哦~

```cpp
int main()
{
	string a = "hello";
	a.append(a, 3);
	cout << a << endl;
	a.append(a, 0, 2);
	cout << a << endl;
	a.append(a, 1, 5);
	cout << a << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/ca775746a45b46189a2465f615ac24f7.png)

自然，如果第二个参数过大还是默认追加到末尾哦~

append第6个是 **append迭代器**，传入两个string迭代器进去就可以实现对该string对象的一部分进行追加哦~迭代器后面会讲，比如如下传入一个迭代器区间（注意此迭代器区间可以理解为是一个指针区间，只不过里面满足[begin, end)）

![](https://img-blog.csdnimg.cn/53b3017026b6417f975569a689f83eca.png)

**扩展**：

#### 4isalpha检查是否为字符

检查字符是不是字母有一个库函数可以检查哦~

![](https://img-blog.csdnimg.cn/c029cc872e6c4b5b8fd0a1104f6af61d.png)

是字母就返回非零，不是就返回零：

![](https://img-blog.csdnimg.cn/737f4f276b5749459d118668258bc388.png)

#### 5iterator迭代器

迭代器可以理解是指向储存数据的一个前后指针。 **只不过可能是指针，也可能不是**。但是用法和指针类似。（ **实际上string的迭代器实现底层就是指针**）

迭代器iterator类似指针，那么它就有一个返回string对象存的字符串的最开始的位置： **begin**

![](https://img-blog.csdnimg.cn/584f15cc0ba64a56a2c7068ef09f0f56.png)

**const**自然也就是指的是const对象，方便其进行只读操作：

既然有一个指向开 始的位置，那么是否有指向结尾的位置呢？有的，指向储存数据真正的结尾（字符串中除开有效字符，实际最后一个字符就是\0）： **end**

![](https://img-blog.csdnimg.cn/ba0aad1409aa4bb1882750559c7c5786.png)

有了头和尾的指向，我们就可以像数组那样进行遍历：

```cpp
int main()
{
	string a = "hello world";
	string::iterator it = a.begin();
	while (it != a.end())
	{
		cout << *it << endl;
		++it;
	}

	return 0;
}
```

![](https://img-blog.csdnimg.cn/d46c54e5452140d3a7d66661817dcab9.png)

因为我们只需要编译有效字符部分，所以当不等于'\0'指向的位置的时候，就遍历完了。需要注意此时类型需要指定是 **string类域中的迭代器iterator**。

迭代器非const对象的迭代器同样可以修改数据：

```cpp
int main()
{
	string a = "hello world";
	string::iterator it = a.begin();
	while (it != a.end())
	{
		cout << (*it)++ << endl;//每打印数据一次此字符就++
		++it;
	}
	cout << a << endl;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/a1be690e358c46c7bf3dc75ecb8dbc61.png)

当然，当const对象使用迭代器的时候就要注意类型就是对应的 **const_iterator**了：

![](https://img-blog.csdnimg.cn/63d5d105ac95414a880fecee467ce018.png)

当然，迭代器也提供了 **反向迭代**：

和begin类似，只不过此时是指向有效字符的最后一个：

![](https://img-blog.csdnimg.cn/65ffd8b99a394d60ba362c42f028706a.png)

和end类似，只不过此时是指向第一个字符的前一个。即和正向迭代的区间[**begin, end)**反了过来： **(rbegin, rend]**。

![](https://img-blog.csdnimg.cn/c13fa04c5f1e4e1098d185b73530f326.png)

注意，此时的类型也就是 **reverse_iterator**了，const对应的在前面加上const。比如如下我们反向遍历一下string：

```cpp
void test2()
{
	string a = "hello world";
	string::reverse_iterator rit = a.rbegin();//必须一一匹配哦
	while (rit != a.rend())
	{
		cout << (*rit)++;//同样可以修改数据
		++rit;
	}
	cout << endl;
	cout << a << endl;
}
```

![](https://img-blog.csdnimg.cn/05a1dd28471a4ac980949500d8f323f9.png)

> 综上，可以总结一下string类中的迭代器（iterator）：
一共四种类型：
**iterator**
**const_iterator**
**reverse_iterator**
**const_reverse_iterator**(当类型长的时候可以使用auto进行自动推导类型，会非常方便)
类似指针一样的作用，但是在其他容器里的迭代器就不一定底层是指针实现的了。

测试一下 **auto**的方便：

```cpp
void test3()
{
	const string a = "hello world";
	//string::const_reverse_iterator crit = a.rbegin();//正常写法  -- 但是类型太长了，可以简写：
	auto crit = a.rbegin();
	while (crit != a.rend())
	{
		cout << *crit;
		++crit;
	}
	cout << endl;
}
```

![](https://img-blog.csdnimg.cn/c05e541bd9ee46ab8eac866cb11b3223.png)

#### **范围for**

既然提到了auto，那么就不得不提一下范围for，在之前，我们认为范围for挺高级的，因为它可以自动迭代和自动判断结束，非常方便，但是实际上范围for的底层就是调用的迭代器：（后序模拟实现string类后，实现了迭代器就可以直接用范围for来自动遍历我们的string类了）

```cpp
void test4()
{
	string a = "hello world";
	for (auto ch: a)
	{
		cout << ch;
	}
	cout << endl;
}
```

![](https://img-blog.csdnimg.cn/bfc68cab9dd94f13bb98e8f70c18a898.png)

实际上每次就是从a那里拷贝给ch，想要改变里面的值的时候就传auto&引用就好了。

#### 6指定位置插入insert

![](https://img-blog.csdnimg.cn/415fb22db08143dfbefbb0e899cb69fb.png)

指定位置同样有很多的重载，第一个参数pos实际上指的就是从当前string对象的字符串哪个位置开始插入，然后就传入字符串，或者几个字符，又或者是传入string对象的哪一部分：

```cpp
void test5()
{
	string b = "world";
	b.insert(0, "hello", 2);//对于常量字符串，后面n表示传几个字符进去
	cout << b << endl;
	b = "world";
	b.insert(0, "hello");//正常头插
	cout << b << endl;
	b = "world";
	b.insert(0, "hello", 0, 3);//局部对应位置插入
	cout << b << endl;
}
```

![](https://img-blog.csdnimg.cn/b49b051ff64f41bf8950393f9deffc9a.png)

insert一直使用的话效率低下，尽量少用。

#### 7删除字符erase

#### ![](https://img-blog.csdnimg.cn/01a5b97edf0844a1b8bb8ecdaf0619e5.png)

pos指定开始位置下标，len表示删几个，没给默认使用缺省值，即删除从pos位置开始到结尾的字符。

```cpp
void test6()
{
	string a = "hello";
	a.erase(1, 1);
	cout << a << endl;
	a.erase(1);
	cout << a << endl;
}
```

![](https://img-blog.csdnimg.cn/e2177eb6601b4e2abaff777c7ab9e5e6.png)

#### 8string的相关增容

首先是max_size:

![](https://img-blog.csdnimg.cn/b7bd44912b644ad1880095fbd49f1bed.png)

查看其字符串长度的最大值--写死的（整形最大值）

![](https://img-blog.csdnimg.cn/aa8f1c76db2748fb96ca2d974ed38bc4.png)

在string对应底层，capacity就是用来储存容量大小的，可以用capacity()查看当前容量大小

![](https://img-blog.csdnimg.cn/291f7965862945c9995c41415fb0ca34.png)

比如在vs（当前我使用的编译器vs2022）底下扩容机制是这样的（使用如下代码进行测试）：

```cpp
void test8()
{
	string a;
	size_t capacity = a.capacity();
	cout << capacity << endl;
	for (int i = 0; i < 120; i++)
	{
		a += 'a';
		if (capacity != a.capacity())
		{
			capacity = a.capacity();
			cout << capacity << endl;
		}
	}
}
```

![](https://img-blog.csdnimg.cn/3b2c835b603e4c228c060ba4d2f0b304.png)

在Linux下似乎呈现：0 1 2 4 8 16 32 64....增长，这里不在演示。

#### reserve开空间

因为扩容会消耗，为了避免频繁的扩容的话，可以使用成员函数 **reserve**（注意此单词有保留的意思，和逆转reverse区分开）：

![](https://img-blog.csdnimg.cn/8011be28bbbb4a9aaaf34d85e6a4b74a.png)

这个就可以提前开好对应的空间大小：（在vs底下为了更好的符合内存对齐，有时编译器会对你指定的大小进行优化）

![](https://img-blog.csdnimg.cn/1b6632b0aa2f43c8ad8ba1a8e069eefa.png)

#### resize开空间也可初始化

当然，如果想要在开空间的同时进行初始化的话，成员函数 **resize()**自然避免不了：

![](https://img-blog.csdnimg.cn/e2166e0a94d54ee39e4957ab56dfd3ba.png)

n就是开的空间大小并且数据个数，后面加上参数char c就表示初始化为对应的字符。

```cpp
void test10()
{
	string a;
	a.resize(16, 'a');
	cout << a.capacity() << endl;
	cout << a << endl;
	//注意是开空间或者初始化赋值，即再开比原本长空间时，会对没有值的空间进行赋值
	a.resize(32);
	cout << a.capacity() << endl;
	cout << a << endl;
	a.resize(64, 'x');
	cout << a.capacity() << endl;
	cout << a << endl;
	//但是当开的数据比现有小时，不会缩减空间，但是对于开空间的n之后的字符就会被清除  注意，赋值初始化只会对没有值的地方
	a.resize(32);
	cout << a.capacity() << endl;
	cout << a << endl;
	a.resize(16, 'e');
	cout << a.capacity() << endl;
	cout << a << endl;
}
```

![](https://img-blog.csdnimg.cn/f38fbabb9e09402ab4f267c37d93dbbf.png)

> 总结reserve和resize：
首先会开辟空间，但是不会缩小空间。
resize初始化只会对没有值的位置进行赋值，有字符的地方不会。
resize当给的n小于当前字符串的长度的时候，会删掉n之后的字符哦（存在内存对齐现象）
resize会同时将size提升上去，默认填'\0'。

#### 9兼容c的字符串c_str()

![](https://img-blog.csdnimg.cn/ceb40797569c48c1b41d7b8cb07366e3.png)

此时这个就可以帮助我们返回 **存储这个字符串的一个字符数组地址**，只不过我们不能进行修改（const）。在进行c和c++语法混合使用的时候会非常有用（读取当前源代码文件，然后打印到控制台上）：

```cpp
void test11()
{
	string str = "teststring8_13.cpp";
	FILE* fout = fopen(str.c_str(), "r");
	char ch = fgetc(fout);//提取字符串
	while (ch != EOF)//没有返回EOF
	{
		cout << ch;//打印到控制台上
		ch = fgetc(fout);
	}
}
```

注意，新的vs编译器已经弃用fopen这个函数，仍要使用的话可以在第一行加上宏：

```cpp
#define _CRT_SECURE_NO_WARNINGS 1;
```

![](https://img-blog.csdnimg.cn/be6a83227cc34a978a62ffd5abc32235.png)

既然知晓了c_str字符串，那么我们就可以区分一下其在输出的时候的区别了：（string以size结束，char*以\0结束）

```cpp
void test12()
{
	string a = "hello world";
	a[5] = '\0';//将下标5处的空格修改为\0
	cout << a << endl;
	cout << a.c_str() << endl;
}
```

![](https://img-blog.csdnimg.cn/0efcb5bca4bf45bfa50db574e01db0c9.png)

可以发现截然不同，cout在遇到\0会默认没有，所以就对\0什么都没打印：

![](https://img-blog.csdnimg.cn/ad822832b0fd4132a54b60cd8085be6d.png)

#### 10查找一个字符、字符串、string对象find\rfind

![](https://img-blog.csdnimg.cn/3f15d78b585847f483d16f16cd5787b2.png) ![](https://img-blog.csdnimg.cn/50fccd4b93104e8ebed3e807915db456.png)

同理，find就是顺着找，rfind就是逆着找，找到对应字符串或者字符出现的第一处下标然后返回，没有找到就返回整形最大值npos（注意到这里如果要匹配必须给入的什么就要 **完全匹配**）：

```cpp
void test13()
{
	string a = "hello";
	cout << a.find('l') << endl;//正向找，默认从a的0开始--找得到 2
	cout << a.find('h', 0) << endl;//字符--找得到 0
	cout << a.find('h', 1) << endl;//字符--找不到 最大值
	cout << a.find("ell", 1) << endl;//常量字符串--找得到 1
	cout << a.find(a, 0) << endl;//string对象 0
	cout << a.find("ell", 1, 1) << endl;//常量字符串局部，第一参数为常量字符串的初始位置，n为几个字符--找得到 2
	cout << a.rfind('l');//反向找，默认从a的最后开始（缺省值为npos）--找得到 3
	//...

}
```

![](https://img-blog.csdnimg.cn/6f93dd3f992d41eda2ddde2555fc67d5.png)

#### substr取子字符串构造一个新的string对象

![](https://img-blog.csdnimg.cn/16dfb09927b84ad29662e8dd9de1451f.png)

同样的，从哪一处下标开始，拷贝多少个过去。默认缺省值是npos：

![](https://img-blog.csdnimg.cn/07319ba26e41409793e27fbf93b099d2.png)

#### 11getline输入一行字符串

我们知道，scanf在读取键盘输入的字符的时候，遇到空格和换行(\n)就会停止读入，将前面的字符从缓冲区给对应的变量，然后空格和\n就不会，cin类似。但是，字符串中也还是存在空格的啊，那么我们如何在输入的时候也能同时将空格也给string呢？

此时就有这么一个成员函数就可以用了，getline():注意包含 **头文件**

![](https://img-blog.csdnimg.cn/0f8011f0214c40e0acce8398f808fee8.png)

第一个getline第三个参数即就是分隔符，在写入时会将其第一个分隔符开始，后面的均不写入str中：

```cpp
void test15()
{
	string a;
	getline(cin, a);
	cout << a << endl;
	getline(cin, a, ' ');//第三个参数为分隔符，即会将从第一个分隔符开始后面的就不会写入a中
	cout << a << endl;
```

![](https://img-blog.csdnimg.cn/89f2ed485aea484d87fa0205f79e4b4d.png)

#### 12string下的类型转化

各种类型转string：

![](https://img-blog.csdnimg.cn/d65c5e7e2e1448ecbfeadd0b69433ab8.png)

字符串转整形：

![](https://img-blog.csdnimg.cn/162004a800294be7a32c10b89648c1d0.png)

字符串转双精度浮点型：

![](https://img-blog.csdnimg.cn/8cacd5c3e1ff4a5d9846df2b7da4bd2f.png)

（注意浮点数是无法精确存储的，会精度丢失）

介绍string的使用差不多啦，让我们来看看如何自己简单实现一个string类吧~

## 二、string类的模拟实现

基于上面我们所了解的内容，为了能够更好的理解string类的底层实现，我们这里就对string类进行一个简单的模拟实现：

### **1.明确实现思路**

基于对库内string实现的理解，我有如下草图作为参考：

![](https://img-blog.csdnimg.cn/3873df0e7ff442d0953188bd472de61b.png)

下面是对上图的一定解释，可以先看看自己实现实现，在参照下面的代码实现哦~

> 1. **成员**：
在构造函数开始之前，我们首先要了解此类中封装的私有成员：1. **char* str**（用来储存数据的指针）2. **size_t size**（存储当前数据个数） 3. **size_t capacity**（存储当前容量大小）4. **const size_t npos = -1**用来指定整形最大值的，加上const可以直接在类里进行初始化，可以发现，这不刚好就是一个顺序表的实现，只不过这次是专门 **针对于char类型**的。
2. **构造函数**：
构造函数是一个 **类实例化成对象**的基石。构造函数又有默认构造string()，拷贝构造string(const string& s)（注意要实现里面的 **深拷贝**），以及相对的析构~string()（析构和构造函数一起来说）。
**3.迭代器实现**：
迭代器是STL中重要的一个部分，是通用的遍历所有容器的方式，在顺序存储中（即非树、链表结构，像数组那样地址按照顺序进行储存数据的）的底层实现就是指针。通常针对于 **正常对象**和 **const对象**有 **begin()**(指向第一个数据),和 **end()**(指向最后一个数据的下一个空间)。
4. **操作数据**：
操作数据的内容就很多了，实际上也就是string类中的需要实现的重要一部分的功能：首先是根据c++的语法实现的非常好用的运算符重载： **operator[]**(能够像数组一样的进行访问)， **push_back()**(对字符能够进行尾插)， **append()**(对字符串能够进行尾插)， **operator+=**(能够更加方便的实现前面两个对字符和字符串进行尾插的功能，重载的同时底层对其复用即可)， **insert()**(对任意位置进行插入字符和字符串)， **erase()**(对任意位置的数据进行清除)，find()(寻找字符或者字符串，更具下标来进行寻找第一个符合要求的子串)， **substr()**(能够根据位置和个数进行构造出一个子串)...

5. **容量**：
关于容量的相关操作实际就是查看当前数据个数 **size()**，查看当前容量个数 **capacity()**，查看当前数据是否为0 **empty()**,(注意前面三个操作要把const对象考虑进去哦~)，然后就是相对重要的 **reserve**(注意和reverse逆转这个单词进行区分)，此函数用于扩充容量，进行拷贝数据，别忘了将原来的数据空间释放掉，防止内存泄漏。其余不做处理。但是还有一个多管闲事的 **resize**就来啦，它不仅仅是扩容，还要对给它的个数里没有初始化的数据根据给入的数据进行初始化，如果给入其个数比原本size还低的话就要注意会发生数据的截断了，超过其个数的就会没有哦~
6. **比较**：
比较相对好说，字符串和字符串比较就按照c语言那一套来定义就好了，比较的是ASCII码值，起始只要实现 **operator<**和 **operator==**其余的重载符号就都可以进行复用了。<
7. **流插入和流提取**：
实际上也就是方便打印string和对string类进行输入。 **<<**打印的时候注意是按照size()的量去打印的，而不是直接把字符串打印出来哦~要一个字符一个字符的去打印。 **>>**的话就需要注意输入' '和'\n'即空格和enter键，键盘输入这两个的时候scanf和cin都会自动认为其为分割符，就不会输入具体的变量中去存储，所有就可以使用in.get()进行输入，方便判断字符的停止条件，当然在这个里面由于要不断调用+=即尾插，也可以先插入一段数组，然后一部分一部分的去插入字符串，这样功耗就会相对少一些哦~

### 2.代码实现

综上，string.h的实现如下代码：命名空间只是作为一个区分，可以自己起名字哦~

```cpp
#pragma once
#define _CRT_SECURE_NO_WARNINGS 1;
#include
#include
using namespace std;

namespace Yushen
{
	class string
	{
	public:
		typedef char* iterator;//迭代器
		typedef const char* const_iterator;

		//string()//这样初始化空对象是不行的  需要一个\0的空间
		//	:_str(nullptr)
		//	,size(0)
		//	,capacity(0)
		//{

		//}
		//构造函数
		string(const char* str = "")//给一个缺省值，这样就可以默认给一个\0了。
		{
			_size = strlen(str);
			_capacity = _size;
			_str = new char[_capacity + 1];//+1是无效字符个数--即最后的\0

			strcpy(_str, str);//会同时将\0拷贝过去
		}

		// 传统写法

		//拷贝构造	需要自己实现深拷贝
		//string(const string& s)
		//{
		//	_size = s._size;
		//	_capacity = s._capacity;
		//	_str = new char[_capacity + 1];
		//	strcpy(_str, s._str);
		//}

		//string(const string& s)
		//	:_str(new char[s._capacity + 1])
		//	,_size(s._size)
		//	,_capacity(s._capacity)
		//{
		//	strcpy(_str, s._str);
		//}

		//赋值重载
		//string& operator=(const string& s)
		//{
		//	if (this != &s)
		//	{
		//		_size = s._size;
		//		_capacity = s._capacity;

		//		delete[] _str;//别忘了原来的那份空间  防止发生内存泄漏
		//		_str = new char[_capacity + 1];
		//		strcpy(_str, s._str);
		//	}
		//	return *this;
		//}

		//类交换
		void swap(string& s)//直接交换成员 就好 -- 如果不实现此类的交换函数外界的交换的话就会发生拷贝构造 -- 代价高
		{
			::swap(_str, s._str);
			::swap(_size, s._size);
			::swap(_capacity, s._capacity);
		}

		//现代写法

		//拷贝构造函数
		string(const string& s)
			:_str(nullptr)
			,_size(0)
			,_capacity(0)
		{
			//请一个帮工的
			string temp(s._str);
			swap(temp);
		}

		//赋值重载函数
		string& operator=(string s)
		{
			//s形参为打工人
			swap(s);
			return *this;
		}

		//析构函数
		~string()
		{
			delete[] _str;
			_str = nullptr;
			_size = _capacity = 0;
		}

		//迭代器
		iterator begin()//返回储存字符串的数组第一个的位置
		{
			return _str;
		}

		iterator end()//返回\0的位置
		{
			return _str + _size;
		}

		const_iterator begin() const//重载begin
		{
			return _str;
		}

		const_iterator end() const//重载end
		{
			return _str + _size;
		}

		//扩容
		void reserve(size_t n)
		{
			if (n > _capacity)//比原来的大才扩容
			{
				char* temp = new char[n + 1];//始终预留好最后一个\0的位置
				_capacity = n;
				strcpy(temp, _str);
				delete[] _str;
				_str = temp;
			}
		}

		void resize(size_t n, char c = '\0')//给一个缺省值 '\0'
		{
			//注意如果比原来的小就要进行删除：
			if (n > _size)
			{
				reserve(n);//一上来先扩容 不管n，由reverse里面的去管
				for (size_t i = _size; i < n; i++)
				{
					_str[i] = c;
				}
				_str[n] = '\0';
			}
			else
			{
				//不进行扩容，但是大于n位置的元素会被删除
				_str[n] = '\0';
			}
			_size = n;
		}

		//输出当前数据个数
		size_t size() const
		{
			return _size;
		}

		//输出当前容量
		size_t capacity() const
		{
			return _capacity;
		}

		//判断当前数据是否为0
		bool empty() const
		{
			return _size == 0;
		}

		//重载< _capacity)
			//{
			//	reserve(_size + n);//大了就进行扩容
			//}
			//strcpy(_str + _size, s);
			//_size += n;
			//可以套用任意位置插入
			insert(_size, s);
		}

		//此时就可以重载+=了，非常好用
		string& operator+=(char c)
		{
			push_back(c);
			return *this;
		}

		string& operator+=(const char* s)
		{
			//复用append即可
			append(s);
			return *this;
		}

		//任意位置插入  位置 插入字符、字符串
		string& insert(size_t pos, char c)
		{
			assert(pos  _capacity)
			{
				reserve(_size + len);
			}
			//移动位置
			size_t n = _size + 1;
			while (n != pos)
			{
				_str[n + len - 1] = _str[n - 1];
				--n;
			}

			//strncpy(_str + pos, s, len);
			memcpy(_str + pos, s, sizeof(char) * len);
			_size += len;
			return *this;
		}

		//删除从pos位置开始的n个字符
		void erase(size_t pos, size_t len = npos)//缺省值为全部
		{
			assert(pos < _size);

			if (len == npos || pos + len >= _size)//从pos开始往后删完
			{
				_str[pos] = '\0';
				_size = pos;
			}
			else
			{
				size_t n = pos + len;
				while (n != _size + 1)
				{
					_str[n - len] = _str[n];
					n++;
				}
				_size = _size - len;
			}
		}

		//寻找子串
		size_t find(char c, size_t pos = 0)//默认从第一个开始找
		{
			assert(pos < _size);
			for (size_t i = pos; i < _size; i++)
			{
				if (_str[i] == c)
					return i;
			}
			return npos;
		}

		size_t find(const char* s, size_t pos = 0)
		{
			assert(pos < _size);
			char* tmp = strstr(_str + pos, s);
			if (tmp == nullptr)
			{
				return npos;
			}
			return tmp - _str;
		}

		//构造子串
		string substr(size_t pos, size_t len = npos) const
		{
			assert(pos < _size);

			size_t n = pos + len;
			//第一种情况，超出容量或者默认npos
			if (len == npos || pos + len >= _size)
			{
				n = _size;
			}

			string tmp;
			for (size_t i = pos; i < n; i++)
			{
				tmp += _str[i];
			}

			return tmp;
		}

		//大于小于等于 -- 根据ASCII码值进行比较
		//重载 < == 其余均可以直接复用
		bool operator(const string& s) const
		{
			return !(*this =(const string& s) const
		{
			return !(*this < s);
		}

		bool operator!=(const string& s) const
		{
			return !(*this == s);
		}

		//clear
		void clear()
		{
			_str[0] = '\0';
			_size = 0;
		}

	private:
		char* _str;//存放字符串
		size_t _size;//当前个数
		size_t _capacity;//容量
		const static size_t npos = -1;
	};

	ostream& operator<>(istream& in, string& s)
	{
		//防止不断的扩容造成太多的消耗
		//首先将原来的数据清零
		s.clear();
		char arr[16] = { 0 };
		char ch = in.get();
		int i = 0;
		while (ch != ' ' && ch != '\n')
		{
			arr[i++] = ch;
			if (i == 15)
			{
				arr[15] = '\0';
				s += arr;
				i = 0;
			}
			ch = in.get();
		}
		arr[i] = '\0';
		s += arr;

		return in;
	}
}
```

ps：其中注释掉的部分是为了突出问题，可酌情观看~
