---
link: https://blog.csdn.net/weixin_61508423/article/details/126917530
title: 【C++】vector从理解到深入
description: 文章浏览阅读567次，点赞2次，收藏2次。这篇文章我会从会用vector的容器开始，理解到最后的模拟实现vector的基本框架，以便达到我们对此容器的一个熟练操作的目的。_vector v4(v3.begin(),v3.end())
keywords: vector<int> v4(v3.begin(),v3.end())
author: 柒海啦 Csdn认证博客专家 Csdn认证企业博客 码龄3年 暂无认证
date: 2022-09-21T05:33:13.000Z
publisher: null
stats: paragraph=53 sentences=28, words=107
---
## 前言

大家好呀！欢迎来到我的C++系列学习笔记！~上一篇是有关STL标准模板库的string的理解到深入，传送门在这里哦：[【C++】从使用string类到模拟实现string类_柒海啦的博客-CSDN博客_c++引入string](https://blog.csdn.net/weixin_61508423/article/details/126296698?spm=1001.2014.3001.5502 "【C++】从使用string类到模拟实现string类_柒海啦的博客-CSDN博客_c++引入string")

vector同样是STL里的一个容器，其功能就是存放任意数据的一个动态数组。这篇文章我会从会用vector的容器开始，理解到最后的模拟实现vector的基本框架，以便达到我们对此容器的一个熟练操作的目的。

![](https://img-blog.csdnimg.cn/14ee4a77148940eeb564c3178a32f037.png)

**目录**

[一、熟练使用vector](#%e4%b8%80%e3%80%81%e7%86%9f%e7%bb%83%e4%bd%bf%e7%94%a8vector%c2%a0)

[1.大致框架](#1%e5%a4%a7%e8%87%b4%e6%a1%86%e6%9e%b6)

[2.代码演示](#2%e4%bb%a3%e7%a0%81%e6%bc%94%e7%a4%ba)

[构造函数：](#%e6%9e%84%e9%80%a0%e5%87%bd%e6%95%b0%ef%bc%9a)

[插入、遍历操作：](#%e6%8f%92%e5%85%a5%e3%80%81%e9%81%8d%e5%8e%86%e6%93%8d%e4%bd%9c%ef%bc%9a)

[迭代器失效：](#%e8%bf%ad%e4%bb%a3%e5%99%a8%e5%a4%b1%e6%95%88%ef%bc%9a)

[容量](#%e5%ae%b9%e9%87%8f%c2%a0)

[二、简单模拟实现vector](#%e4%ba%8c%e3%80%81%e7%ae%80%e5%8d%95%e6%a8%a1%e6%8b%9f%e5%ae%9e%e7%8e%b0vector%c2%a0)

[1.代码分析](#1%e4%bb%a3%e7%a0%81%e5%88%86%e6%9e%90)

[成员变量：](#%c2%a0%e6%88%90%e5%91%98%e5%8f%98%e9%87%8f%ef%bc%9a)

[插入成员变量变化演示：](#%e6%8f%92%e5%85%a5%e6%88%90%e5%91%98%e5%8f%98%e9%87%8f%e5%8f%98%e5%8c%96%e6%bc%94%e7%a4%ba%ef%bc%9a%c2%a0)

[迭代器相关实现：](#%e8%bf%ad%e4%bb%a3%e5%99%a8%e7%9b%b8%e5%85%b3%e5%ae%9e%e7%8e%b0%ef%bc%9a)

[无参构造函数：](#%c2%a0%e6%97%a0%e5%8f%82%e6%9e%84%e9%80%a0%e5%87%bd%e6%95%b0%ef%bc%9a)

[迭代器区间构造函数：](#%e8%bf%ad%e4%bb%a3%e5%99%a8%e5%8c%ba%e9%97%b4%e6%9e%84%e9%80%a0%e5%87%bd%e6%95%b0%ef%bc%9a)

[交换&拷贝构造：](#%e4%ba%a4%e6%8d%a2%26%e6%8b%b7%e8%b4%9d%e6%9e%84%e9%80%a0%ef%bc%9a)

[析构函数：](#%e6%9e%90%e6%9e%84%e5%87%bd%e6%95%b0%ef%bc%9a)

[数据个数和容量：](#%e6%95%b0%e6%8d%ae%e4%b8%aa%e6%95%b0%e5%92%8c%e5%ae%b9%e9%87%8f%ef%bc%9a)

[赋值重载：](#%e8%b5%8b%e5%80%bc%e9%87%8d%e8%bd%bd%ef%bc%9a)

[[]重载：](#%5b%5d%e9%87%8d%e8%bd%bd%ef%bc%9a)

[开空间或者初始化reserve&resize:](#%e5%bc%80%e7%a9%ba%e9%97%b4%e6%88%96%e8%80%85%e5%88%9d%e5%a7%8b%e5%8c%96reserve%26resize%3a)

[front、back、empty：](#front%e3%80%81back%e3%80%81empty%ef%bc%9a)

[操作：插入、删除](#%e6%93%8d%e4%bd%9c%ef%bc%9a%e6%8f%92%e5%85%a5%e3%80%81%e5%88%a0%e9%99%a4)

[2.综合代码：](#2%e7%bb%bc%e5%90%88%e4%bb%a3%e7%a0%81%ef%bc%9a)

## 一、熟练使用vector

因为是STL，所以里面的容器操作还是熟悉的操作，我们只需要明白其容器的大致框架，然后具体的对这些进行模拟实现即可，所以我们针对这些框架去熟练使用vector。

### 1.大致框架

![](https://img-blog.csdnimg.cn/5fad6f8327b248e0b5a51c6738ed332e.png)

vector的理解流程大致如上图所示。首先自然了解其构造函数，明确内部所拥有的成员变量，其次也要明确析构。

然后容量来说就是对整个空间进行申请或者初始化操作了，运算符重载要么就是方便使用，要么就是重载一些比较常用且重要的（比如赋值重载）。操作就是一个动态线性表一般的操作，随机插入，随机删除，尾插，尾删.......

接下来我会用具体代码来对对应的模块进行演示，帮助我们一起理解vector这个容器。

比较好的C++文档在这里：[https://cplusplus.com/reference/vector/vector/](https://cplusplus.com/reference/vector/vector/ "https://cplusplus.com/reference/vector/vector/")

### 2.代码演示

#### 构造函数：

空数组、几个相同的数构造、拷贝构造、迭代器构造。

![](https://img-blog.csdnimg.cn/db5a9d924f1b4a0b8b8bc45d0473a1eb.png)

参考代码：

```cpp
void test1()
{
	vector v1;										// 空
	vector v2(6, 6);                               // 初始化6个6
	vector v3(v2); // 或者 vector v3 = v2     // 拷贝构造
	vector v4(v3.begin(), v3.end());			    // 迭代器构造
}
```

![](https://img-blog.csdnimg.cn/2662b6557e8543f5964bb1120653f0b7.png)

![](https://img-blog.csdnimg.cn/4d5b136ba17e4f3b8ed64dff6395c9fa.png) ![](https://img-blog.csdnimg.cn/93bc818d48374b74bf3953a6aaba4453.png)

可以发现v1 v2 v3 v4对象均构造成功，v2对应的就是构造6个6，v3就是拷贝构造，v4用了两个迭代器进行构造。（一般线性顺序表的迭代器的底层实际上就是指针）

既然构造成功了，那么我们就需要对其进行操作了。

#### 插入、遍历操作：

首先，可以简单的看一下我们耳熟能详的 **push_back**:(push_back实际上底层也就是调用的insert())。

![](https://img-blog.csdnimg.cn/98c714b26e06484e954ae3a141b70275.png)

同样的，在vector里面也支持 **[]**运算符重载，所以我们就可以通过下标的方式去遍历vector，自然同样存在 **size()**是统计当前数据的个数的。

![](https://img-blog.csdnimg.cn/6d4d7a5e6575450f8842010a6f1706d0.png)![](https://img-blog.csdnimg.cn/e4a81ffe72364fca90b9e3fcd6cb4ebf.png)

参考代码：

```cpp
void test2()
{
	// 插入 遍历
	vector v1(6, 1);
	v1.push_back(2); // 尾插
	// 利用下标去遍历数据
	for (size_t i = 0; i < v1.size(); ++i)
	{
		cout << v1[i] << " ";
	}
	cout << endl;

}
```

![](https://img-blog.csdnimg.cn/61144d34293248cd8e1a2bb2ab644592.png)

上述取出数据我们使用的是[]重载，实际上，我们取出数据的方式有很多种，这里介绍两种：一个取出头的数据 **front()**，一个取出尾的数据 **back()**。

![](https://img-blog.csdnimg.cn/2080c09e605b446caf18dfb7f77fb486.png) ![](https://img-blog.csdnimg.cn/01279b2a99784b21a4ff39a3833de4be.png)

插入数据的话自然也就是有 **insert**之类的函数：

![](https://img-blog.csdnimg.cn/a5b77ef089b04972ba387b36dd300598.png)

当然，和插入对应的就是删除了， **erase()**即随机删除：

![](https://img-blog.csdnimg.cn/11c683bc4be541478ac5c30e54b91a6d.png)

也存在尾删pop_back()：

![](https://img-blog.csdnimg.cn/8137439607fd4ff79f558ada6b80402f.png)

需要注意的是，上述传入的参数并不是size_t或者int的下标，而是迭代器iterator哦~

提到迭代器自然就不会忘记end(),begin(),rend(),rbegin()...... 并且实际上此迭代器的实现底层也就是简单的指针，且类型应该是：std::vector

那么我们可以利用其进行遍历--同时就可以使用范围for，然后进行一些基本的随机插入删除等操作：

```cpp
void test3()
{
	// 插入遍历
	// 迭代器及其迭代器遍历：
	vector v(3, 2);
	vector::iterator it = v.begin();
	while (it != v.end())  // end为最后的元素的下一个
	{
		cout << *it << " ";  // 像指针一样的进行访问
		++(*it);  // 同样的可以进行修改
		++it;
	}
	cout << endl;

	// 支持迭代器，也就支持范围for
	for (auto& e : v)
	{
		cout << e << " ";
	}
	cout << endl;

	v[0] = 1;
	v[2] = 0;
	// 打印头尾，不用[]重载
	cout << "头：" << v.front() << endl;
	cout << "尾：" << v.back() << endl;

	// 随机插入 先寻找
	it = find(v.begin(), v.end(), 3);
	// 在3前插入 2
	v.insert(it, 2);

	for (auto& e : v)
	{
		cout << e << " ";
	}
	cout << endl;

	// 随机删除
	//v.erase(it);  // 在把此位置删除  直接这样会出现失效的问题，所以必须重新寻找或者接收insert的返回值
	it = find(v.begin(), v.end(), 2);
	v.erase(it);
	for (auto& e : v)
	{
		cout << e << " ";
	}
	cout << endl;

}
```

![](https://img-blog.csdnimg.cn/7edd369127d84f32b90e333afa5df728.png)

这里提到了迭代器失效的情况，那么这个是什么情况呢？

#### 迭代器失效：

所谓迭代器失效，也就是在进行insert或者erase，即移动空间的时候，就可能会产生当前位置地址被释放掉，转移到其他地方去了，那么此时地址也就是 **野指针**了，这就是 **迭代器失效**。

```cpp
void test4()
{
	vector v(2, 2);
	auto it = v.begin();  // 头结点值地址
	printf("%p\n", it);
	v.erase(it);
	//验证it是否失效
	printf("%p\n", it);
	printf("%p\n", v.begin());
	for (auto& e : v)
	{
		cout << e << " ";
	}
	cout << endl;
}
```

![](https://img-blog.csdnimg.cn/2a535882200040d6af4bef29f09fe028.png)

此时可以发现三者的地址完全不一样，说明发生了变化，也就是所谓的迭代器失效了。

其实，在不同的平台下，编译器对此迭代器失效的方法也存在不一样的地方：

比如，我们测试程序12345，对此数组中的偶数进行删除：在linuxg++和Windows下的vs2022分别进行测试：

```cpp
void test5()
{
	// 迭代器遍历随机删除
	vector v;
	v.push_back(1);
	v.push_back(2);
	v.push_back(3);
	v.push_back(4);
	v.push_back(5);

	for (auto& e : v)
	{
		cout << e << " ";
	}
	cout << endl;

	vector::iterator it = v.begin();
	// 迭代器失效
	while (it != v.end())
	{
		if (*it % 2 == 0)
		{
			// 偶数删除
			v.erase(it);
		}
		++it;
	}
	for (auto& e : v)
	{
		cout << e << " ";
	}
	cout << endl;
}
```

![](https://img-blog.csdnimg.cn/23e61f4458cf47cf81f1b38e13a8c394.png) vs2022 直接崩溃

![](https://img-blog.csdnimg.cn/221ec61bd6c749e2aac960dfa4beed01.png)

Linux下的g++却没有。

上述情况也就说明了：

> VS下和Linux下可能会不一样： **STL只是一个规范-->不同的平台下面实现是不同的**。
VS下对迭代器会进行强制检查，如果原本指向的数据被清除了，下次在对此处指向（存在数据，不是野指针），但是会检查报错
Linux下：12345没有问题（删去偶数，不正确使用）1235
-- 数据排列的偶然性（最后一个不是偶数，没有连续的偶数）

但是当我们遍历插入或者删除的时候就会产生很多不便，那么insert以及erase要如何处理才能优化好呢？对返回值进行操作：

![](https://img-blog.csdnimg.cn/08454125d7744d9b98d045cb82153420.png) ![](https://img-blog.csdnimg.cn/cff8f55639a441ddbd63f192458852f1.png)

insert返回的就是新插入的那个元素位置的迭代器，而erase返回的就是被删除元素的下一个元素的迭代器。

知晓这些之后，我们就可以稍微修改一下上面的代码，使其能在Windows平台下也能够跑起来：

```cpp
	while (it != v.end())
	{
		if (*it % 2 == 0)
		{
			// 偶数删除
			it = v.erase(it);
		}
		else
			++it;
	}
```

![](https://img-blog.csdnimg.cn/0661a8d17ced4648a16551a9814f0807.png)

此时就能够在Windows平台上跑了。

#### 容量

跟string这个类类似，实际上在STL内基本都有这个扩容以及初始化的 **reserve**和 **resize**，用法都基本一致，一个就是一开始申请多大空间，另一个就是申请的同时也初始化多少数据。

![](https://img-blog.csdnimg.cn/7695d99e29854437aab662cebc16b40f.png)

![](https://img-blog.csdnimg.cn/1e08251b4e4344af87fe97f76ed1925b.png)

其实我们发现resize默认给的是value_type(),其实有的时候存储的数据并不是内置类型，而是自定义类型，所以就会如此设计，这在之后的模拟实现也会涉及。

参考代码：

```cpp
void test6()
{
	// reserve
	vector v1;
	v1.reserve(10);
	cout << v1.capacity() << endl;

	// resize
	vector v2;
	v2.resize(6, 2);  // 初始化6个数据，均为2
	for (auto e : v2)
	{
		cout << e << " ";
	}
	cout << endl;
	v2.resize(10, 1);  // 当申请空间比原本数据个数大，那么会将没填充的数据填为对应数据
	for (auto e : v2)
	{
		cout << e << " ";
	}
	cout << endl;
	v2.resize(2, 0);  // 如果比原本数据小，会发生截断，不会填充数据
	for (auto e : v2)
	{
		cout << e << " ";
	}
	cout << endl;
}
```

![](https://img-blog.csdnimg.cn/373aa4b894fe4234a78618262199e8fa.png)

在了解一些基本并且熟练的时候，就可以对vector进行更加深入的模拟实现了：

## 二、简单模拟实现vector

大致了解后，我们想要进一步深入理解vector的话，就要对其进行模拟实现。既然只是简单的去模拟实现，那么我们只需要实现一个大致框架即可：

### 1.代码分析

**准备工作：**

我们首先可以将其实现代码放在一个命名空间内，这样就可以不用和std内官方的vector相互冲突。当然，此类可以定义在一个头文件内，比如叫做vector.h，这样方便其他程序进行调用此头文件。

![](https://img-blog.csdnimg.cn/ca5aa71c47a546ba9684bdc40d82002b.png)

然后，此类是一个模板类，因为里面的存储的数据可不只有int一种，自然使用template

```cpp
namespace QiHai
{
	// 类模板
	template
	class vector
	{
        //...

    }
//...

}
```

#### 成员变量：

首先看我们的私有成员。因为这是一个顺序结构，所以内存地址是连续的，也就是普通的在堆上申请的动态数组。结合std官方的实现， **我们可以定义如下三个指针**：（同时将指针重命名为 **迭代器类型**）

```cpp
	public:
		typedef T* iterator;  // 定义迭代器类型为iterator
		typedef const T* const_iterator;
```

```cpp
	private:
		iterator _start;          // 存储数据的开始位置
		iterator _finish;         // 存储数据的最后一个位置的后一个位置
		iterator _end_of_storage; // 存储数据的空间最后一个地址
```

三者在某一时候关系可以如图所示：

![](https://img-blog.csdnimg.cn/31969db3f2e54f748f20356dcc9a2833.png)

#### 插入成员变量变化演示：

那么此时你的内心是否存在一个疑惑：那么这些是如何进行控制的呢？其实也就是保证开空间和插入的时候控制其在对应的位置即可，这些后面均会实现，现在这里可以进行一个简单的演示：（假设第一次插入开8个空间）

![](https://img-blog.csdnimg.cn/d940fb65748748fca090d78268c69289.gif)

#### 迭代器相关实现：

和string的实现不一样，我们对外提供的接口也是迭代器，所以首先提供一些基本的迭代器接口：比如begin()、end()等。

```cpp
		iterator begin()
		{
			return _start;
		}

		iterator end()
		{
			return _finish;
		}

		const_iterator begin() const
		{
			return _start;
		}

		const_iterator end() const
		{
			return _finish;
		}
```

#### 无参构造函数：

然后就是最基本的构造函数也是最常用的构造函数。如上述演示一样，我们只需控制三个变量初始化为 **nullptr**即可。

```cpp
        vector()
			:_start(nullptr)
			,_finish(nullptr)
			,_end_of_storage(nullptr)
		{}
```

#### 迭代器区间构造函数：

为了方便后序的拷贝构造函数方便调用，并且也是一种构造函数方式，即对传入的类似于地址使用（即*解应用就是数据），要使用模板函数，结构如下：（push_back还未写，下面会进行实现）--（传入的迭代器，就有点类似于迭代器遍历）

```cpp
		template
		vector(InputIterator first, InputIterator last)
			:_start(nullptr)
			,_finish(nullptr)
			,_end_of_storage(nullptr)
		{
			while (first != last)
			{
				push_back(*first);
				++first;
			}
		}
```

#### 交换&拷贝构造：

拷贝构造也分为现代写法和原始写法，原始写法就是重新建立一个空间，然后一一把传入的对象的数据拷贝到新空间上去，否则默认的拷贝构造实现的是浅拷贝（注意此深拷贝不能使用memcpy进行数据替换，因为当储存对象本身也是一个数组的时候，即内部存的同样也是地址，那么此时就会出现问题，这就是深度拷贝，后面也会具体讲，需要一个一个进行赋值）

当然，除了原始写法，现代写法就高明的多， 会让一个具体的对象帮我们做事，然后将地址进行交换即可。这个对象也就是我们上面的迭代器区间构造，而地址交换就需要我们自己实现一个。

交换函数很简单，只需要执行外部的交换函数将我们每一个变量所存的地址交换即可。

（注意传入的参数必须是一个类型，vector可不是一个具体的类型，而是一个模板，所以需要

```cpp
		void swap(vector& v)
		{
			::swap(_start, v._start);
			::swap(_finish, v._finish);
			::swap(_end_of_storage, v._end_of_storage);
		}
```

拷贝构造现代写法：

```cpp
		vector(const vector& v)
			:_start(nullptr)
			, _finish(nullptr)
			, _end_of_storage(nullptr)
		{
			// 现代做法，请打工人
			vector tmp(v.begin(), v.end());  // 请到的打工人
			swap(tmp);  // 两者交换，它的就是我的了
		}
```

#### 析构函数：

析构函数很简单，因为我们变量的申请的空间由堆而来（new/malloc），所以我们将头指针释放掉，然后将其所有变量置为空即可。

```cpp
		~vector()
		{
			delete[] _start;
			_start = _finish = _end_of_storage = nullptr;
		}
```

#### 数据个数和容量：

为了统计当前数据个数以及容量大小，结合之前创建变量的介绍，这里可以轻松的解决：

```cpp
		size_t size() const
		{
			return _finish - _start;
		}

		size_t capacity() const
		{
			return _end_of_storage - _start;
		}
```

#### 赋值重载：

赋值重载和拷贝构造函数有点类似，只不过一个是新创建一个对象，而赋值是两个以及存在的对象让另一个的值拷贝给另一个。

也有原始写法和现代写法，原始写法同样注意深度拷贝的问题，而现代写法就只需要让形参做工具人，因为此时传参就会发生拷贝构造，然后与其形参交换数据即可。同时注意返回对象的本身。

```cpp
		vector& operator=(vector v)
		{
			swap(v);  // 把形参当做打工人
			return *this;
		}
```

#### []重载：

此重载也就是方便像数组一样的对下标进行访问，方便快捷。实际上也就是根据传入的下标，根据首元素地址加减获得，因为可以修改数据，那么就可传出引用，const对象加上const即可：

```cpp
		T& operator[](size_t n)
		{
			assert(n < size());
			return _start[n];
		}

		const T& operator[](size_t n) const
		{
			assert(n < size());
			return _start[n];
		}
```

#### 开空间或者初始化reserve&resize:

开空间是插入的基础。首先就需要存在空间才能进行插入操作，也才能真正的构造一个数组。如上面的动画演示一样，我们可以对这个函数传入n表示我们要开n个对应数据类型的空间。

所谓开空间，也就是原来开的的一串地址空间不够了或者没有，然后需要开辟一个比原来大的空间供数据进行操作。既然是新的空间，那么数据就要进行搬家。数据搬家也就是要进行拷贝。在拷贝的同时我们要注意到数据类型不再是像string那样简单的char类型了，而是 **T**。这个T可以是内置类型，当然也可以是vector

```cpp
		void reserve(size_t n)
		{
			if (n > capacity())
			{
				iterator new_start = new T[n];
				size_t len = size();

				if (_start)  // 有可能是空数据进行
				{
					// 普通的进行内存拷贝的话，存在更深层次无法拷贝
					//memcpy(new_start, _start, sizeof(T) * capacity());
					// 深层次拷贝  一个一个进行拷贝 方便调用其赋值进行深拷贝
					for (size_t i = 0; i < len; i++)
						new_start[i] = _start[i];
					delete[] _start;
				}

				_start = new_start;
				_finish = _start + len;
				_end_of_storage = _start + n;
			}
		}
```

开空间并且初始化实际上分为两个步骤，一个进行开空间，另一个负责初始化数据。注意缺省参数是一个匿名构造，C++为了迎合自定义类型初始数据的问题，也给内置类型定义了一个匿名对象，所以使用匿名对象对内置类型同样适合。

```cpp
		void resize(size_t n, const T& val = T())  // 缺省值给T的一个匿名对象 （C++为了迎合自定义类型，也给内置类型增加了无参构造）
		{
			// 1. n > capacity()
			if (n > capacity())
			{
				reserve(n);  // 扩容
			}
			for (size_t i = size(); i < n; i++)
				_start[i] = val;

			_finish = _start + n;
		}
```

#### **front、back、empty：**

front获得头数据，back获得尾数据，只需要像数组那样访问即可，记得传出引用即可：

```cpp
		T& front()
		{
			assert(size() > 0);
			return *(_start);
		}

		T& back()
		{
			assert(size() > 0);
			return *(_finish - 1);
		}

		bool empty()
		{
			return _start == _finish;
		}
```

#### 操作：插入、删除

只有插入我们才能对数据进行管理。随机插入可以复用于尾插，随机删除同样可以。需要注意的是在挪动空间，原本传入的迭代器就可能出现失效的问题，那么我们只需控制insert返回插入后新数据所在的位置指针，删除数据的下一个数据位置指针即可。（插入空间满申请空间，然后进行移动，删除进行移动操作）--注意检查边界问题哦~

```cpp
		// 随机插入
		iterator insert(iterator pos, const T& val)
		{
			assert(pos >= _start);
			assert(pos = pos)
			{
				*(temp + 1) = *temp;
				--temp;
			}
			*pos = val;
			++_finish;

			return pos;  // 返回改变地址后的迭代器位置
		}

		// 随机删除  返回删除位置的下一个位置的迭代器
		iterator erase(iterator pos)
		{
			assert(pos < _finish);
			assert(pos >= _start);

			iterator temp = pos + 1;
			while (temp
```

此时尾插尾删复用即可。

### 2.综合代码：

仅供参考，有误请大佬指正！

```cpp
// vector.h
#pragma once
#include
#include
#include
#include

using namespace std;

// 模拟实现vector

namespace QiHai
{
	// 类模板
	template
	class vector
	{
	public:
		typedef T* iterator;  // 定义迭代器类型为iterator
		typedef const T* const_iterator;

		// 迭代器
		iterator begin()
		{
			return _start;
		}

		iterator end()
		{
			return _finish;
		}

		const_iterator begin() const
		{
			return _start;
		}

		const_iterator end() const
		{
			return _finish;
		}

		// 类中交换函数
		void swap(vector& v)
		{
			::swap(_start, v._start);
			::swap(_finish, v._finish);
			::swap(_end_of_storage, v._end_of_storage);
		}

		// 构造函数
		vector()
			:_start(nullptr)
			,_finish(nullptr)
			,_end_of_storage(nullptr)
		{}

		// 迭代器区间进行初始化构造
		template
		vector(InputIterator first, InputIterator last)
			:_start(nullptr)
			,_finish(nullptr)
			,_end_of_storage(nullptr)
		{
			while (first != last)
			{
				push_back(*first);
				++first;
			}
		}

		// 拷贝构造
		vector(const vector& v)
			:_start(nullptr)
			, _finish(nullptr)
			, _end_of_storage(nullptr)
		{
			// 现代做法，请打工人
			vector tmp(v.begin(), v.end());  // 请到的打工人
			swap(tmp);  // 两者交换，它的就是我的了
		}

		// 析构函数
		~vector()
		{
			delete[] _start;
			_start = _finish = _end_of_storage = nullptr;
		}

		// 数据个数
		size_t size() const
		{
			return _finish - _start;
		}

		size_t capacity() const
		{
			return _end_of_storage - _start;
		}

		// 重载
		// 赋值重载
		vector& operator=(vector v)
		{
			swap(v);  // 把形参当做打工人
			return *this;
		}

		// []重载
		T& operator[](size_t n)
		{
			assert(n < size());
			return _start[n];
		}

		const T& operator[](size_t n) const
		{
			assert(n < size());
			return _start[n];
		}

		// 开空间
		void reserve(size_t n)
		{
			if (n > capacity())
			{
				iterator new_start = new T[n];
				size_t len = size();

				if (_start)  // 有可能是空数据进行
				{
					// 普通的进行内存拷贝的话，存在更深层次无法拷贝
					//memcpy(new_start, _start, sizeof(T) * capacity());
					// 深层次拷贝  一个一个进行拷贝 方便调用其赋值进行深拷贝
					for (size_t i = 0; i < len; i++)
						new_start[i] = _start[i];
					delete[] _start;
				}

				_start = new_start;
				_finish = _start + len;
				_end_of_storage = _start + n;
			}
		}

		// 开空间并且初始化
		void resize(size_t n, const T& val = T())  // 缺省值给T的一个匿名对象 （C++为了迎合自定义类型，也给内置类型增加了无参构造）
		{
			// 1. n > capacity()
			if (n > capacity())
			{
				reserve(n);  // 扩容
			}
			for (size_t i = size(); i < n; i++)
				_start[i] = val;

			_finish = _start + n;
		}

		T& front()
		{
			assert(size() > 0);
			return *(_start);
		}

		T& back()
		{
			assert(size() > 0);
			return *(_finish - 1);
		}

		bool empty()
		{
			return _start == _finish;
		}

		void pop_back()
		{
			assert(_finish > _start);
			--_finish;
		}

		// 操作
		// 尾插
		void push_back(const T& val)
		{
			// 直接复用insert()即可
			insert(end(), val);
		}

		// 随机插入
		iterator insert(iterator pos, const T& val)
		{
			assert(pos >= _start);
			assert(pos = pos)
			{
				*(temp + 1) = *temp;
				--temp;
			}
			*pos = val;
			++_finish;

			return pos;  // 返回改变地址后的迭代器位置
		}

		// 随机删除  返回删除位置的下一个位置的迭代器
		iterator erase(iterator pos)
		{
			assert(pos < _finish);
			assert(pos >= _start);

			iterator temp = pos + 1;
			while (temp
```

谢谢观看~
