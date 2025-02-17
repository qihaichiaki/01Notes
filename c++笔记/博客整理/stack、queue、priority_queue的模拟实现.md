---
link: https://blog.csdn.net/weixin_61508423/article/details/127064176
title: 【C++】stack、queue、priority_queue的模拟实现
description: 文章浏览阅读361次。本篇主要简单介绍一下stack、queue、priority_queue的作用以及用法，然后就快速到具体的模拟实现。本篇会引入两个新的概念：适配器和仿函数_c++ stack是先进先出
keywords: c++ stack是先进先出
author: 柒海啦 Csdn认证博客专家 Csdn认证企业博客 码龄3年 暂无认证
date: 2024-04-02T08:45:05.000Z
publisher: null
stats: paragraph=23 sentences=10, words=93
---
## 前言

hello~大家好呀！欢迎大家进入我的C++系列笔记！上一篇的传送点在这里哦：[【C++】vector从理解到深入_柒海啦的博客-CSDN博客](https://blog.csdn.net/weixin_61508423/article/details/126917530?spm=1001.2014.3001.5502 "【C++】vector从理解到深入_柒海啦的博客-CSDN博客")

本篇主要简单介绍一下stack、queue、priority_queue的作用以及用法，然后就快速到具体的模拟实现。本篇会引入两个新的概念：适配器和仿函数，模拟实现的时候会具体介绍，我们开始吧~

![](https://img-blog.csdnimg.cn/c29ff8fa0c50455fb0023c3a893900b9.png)

（红蓝赛高~）

**目录**

[一、stack、queue、priority_queue的简单引入](#%e4%b8%80%e3%80%81stack%e3%80%81queue%e3%80%81priority-queue%e7%9a%84%e7%ae%80%e5%8d%95%e5%bc%95%e5%85%a5)

[1.验证特性](#1%e9%aa%8c%e8%af%81%e7%89%b9%e6%80%a7%c2%a0)

[2.适配器与仿函数](#2%e9%80%82%e9%85%8d%e5%99%a8%e4%b8%8e%e4%bb%bf%e5%87%bd%e6%95%b0)

[二、stack、queue、priority_queue的模拟实现](#%e4%ba%8c%e3%80%81stack%e3%80%81queue%e3%80%81priority-queue%e7%9a%84%e6%a8%a1%e6%8b%9f%e5%ae%9e%e7%8e%b0)

[1.stack&queue的模拟实现](#1stack%26queue%e7%9a%84%e6%a8%a1%e6%8b%9f%e5%ae%9e%e7%8e%b0)

[总体思路：](#%e6%80%bb%e4%bd%93%e6%80%9d%e8%b7%af%ef%bc%9a)

[完整代码：](#%e5%ae%8c%e6%95%b4%e4%bb%a3%e7%a0%81%ef%bc%9a)

[2.priority_queue的模拟实现](#2priority-queue%e7%9a%84%e6%a8%a1%e6%8b%9f%e5%ae%9e%e7%8e%b0)

[总体思路：](#%e6%80%bb%e4%bd%93%e6%80%9d%e8%b7%af%ef%bc%9a)

[完整代码：](#%e5%ae%8c%e6%95%b4%e4%bb%a3%e7%a0%81%ef%bc%9a)

## 一、stack、queue、priority_queue的简单引入

### 1.验证特性

> stack
*栈结构 特性： **先进后出，后进先出**
queue
*队列结构 特性： **先进先出，后进后出**
priority_queue
*优先级队列 特性： **堆结构存储（默认大根堆），每次取出值为当前存储值的最大值**（最小值）

比如使用一下程序来验证上述的特性：

```cpp
	void test()
	{
		// 均为模板类 需要给定类型
		stack s;
		queue q;
		priority_queue p;

		// 1 -9 4 9 3 0 -2 60
		int arr[] = { 1, -9, 4, 9, 3, 0, -2, 60 };
		int len = sizeof arr / sizeof arr[0];
		for (int i = 0; i < len; ++i)
		{
			s.push(arr[i]);
			q.push(arr[i]);
			p.push(arr[i]);
		}

		cout << "stack:\n";
		while (!s.empty())
		{
			cout << s.top() << " ";  // 60 -2 0 3 9 4 -9 1
			s.pop();
		}
		cout << endl;
		cout << "queue:\n";
		while (!q.empty())
		{
			cout << q.front() << " ";  // 1, -9, 4, 9, 3, 0, -2, 60
			q.pop();
		}
		cout << endl;
		cout << "priority_queue:\n";
		while (!p.empty())
		{
			cout << p.top() << " ";  // 60  9 4 3 1 0 -2 -9
			p.pop();
		}
		cout << endl;
	}
```

![](https://img-blog.csdnimg.cn/ba5e0d25901d479ebee7ea664ca4e8b9.png)

可以发现是可以完美符合性质的哦。

可是，我们查看文档，发现stack、queue、以及priority_queue都有一个：Container的模板参数，并且stack和queue给的都是deque，priority_queue给的是vector。另外priority_queue还有一个模板参数Compare，给的是less。

![](https://img-blog.csdnimg.cn/4da2f5277aee43c0ac9df22ad6fdd945.png)

![](https://img-blog.csdnimg.cn/b3fa494b0b8b4a2e9250f82bd07d223b.png)

![](https://img-blog.csdnimg.cn/6d20a5b034374d62bcd30f36ad726cae.png) 这就是我们马上要开始的适配器和仿函数。

### 2.适配器与仿函数

适配器，顾名思义，就是适配的一种模式。

> 在计算机编程中，适配器模式（有时候也称包装样式或者包装）将一个类的接口适配成用户所期待的。一个适配允许通常因为接口不兼容而不能在一起工作的类工作在一起，做法是将类自己的接口包裹在一个已存在的类中。（百度百科）

而这里就是容器适配器。可以看到给的缺省参数是deque，那么相比底层代码操作的也就是deque（deque实际上是综合了vector和list的特点，适合作为stack和queue的容器适配器）。如果是利用C语言进行编程，那么就无法使用到如此便捷的容器，也只能像之前实现vector那样或者说实现线性表那样，自己开空间，赋值，扩容，深浅拷贝......

既然有这个参数，也就是说明了我们使用别的容器也可以咯， 可以试下vector和list：

```cpp
	void test2()
	{
		// 更改容器适配器
		stack> sv;
		stack> sl;
		//queue> qv;
		queue> ql;

		int arr[] = { 1, 2, 3, 4, 5};
		int len = sizeof arr / sizeof arr[0];
		for (int i = 0; i < len; ++i)
		{
			sv.push(arr[i]);
			sl.push(arr[i]);
			//qv.push(arr[i]);
			ql.push(arr[i]);
		}

		while (!sv.empty())
		{
			cout << sv.top() << " ";
			sv.pop();
		}
		cout << endl;

		while (!sl.empty())
		{
			cout << sl.top() << " ";
			sl.pop();
		}
		cout << endl;

		//while (!qv.empty())
		//{
		//	cout << qv.front() << " ";
		//	qv.pop();
		//}
		//cout << endl;

		while (!ql.empty())
		{
			cout << ql.front() << " ";
			ql.pop();
		}
		cout << endl;
	}
```

![](https://img-blog.csdnimg.cn/86673221d90743f1a580f11948b1dc54.png)

实际上，在queue的传递容器适配器的时候，底层实现了pop_front。而vector容器是不支持头插头删的。所以如果报错就说明此容器不适合。

![](https://img-blog.csdnimg.cn/769f19fddccd402796887653b28545bc.png)

既然容器适配器就是我们实现此结构的一个容器，那么仿函数（compare）又是什么呢？

> 仿函数(functor)，就是使一个类的使用看上去像一个函数。其实现就是类中实现一个[operator](https://baike.baidu.com/item/operator/2387244?fromModule=lemma_inlink "operator")()，这个类就有了类似函数的行为，就是一个仿函数类了。

是的，仿函数用起来就像一个函数一样，实际上是通过类进行实现的，然后在类中实现()运算符重载：

```cpp
	// 比较仿函数
	template
	struct myless
	{
		// <
		bool operator()(const T& val1, const T& val2)
		{
			return val1 < val2;
		}
	};

	void test4()
	{
		int a = 2, b = 3;
		myless less;
		if (less(a, b))
			cout << "a < b" << endl;
		else cout << "a >= b" << endl;
	}
```

![](https://img-blog.csdnimg.cn/046d71249cee4f7f938f14a7151a844c.png)

compare是比较的意思，在优先级队列里面第三个模板参数也就意思在于我们传入的比较函数用来 **控制优先级是大根堆还是小根堆**。而在算法库

下面简单测试一下：

```cpp
	void test3()
	{
		// 测试仿函数类型传入
		priority_queue q;  // 默认大根堆
		priority_queue, greater> p;  // 指定小根堆

		int arr[] = { 1, 2, 3, 4, 5 };
		int len = sizeof arr / sizeof arr[0];
		for (int i = 0; i < len; ++i)
		{
			q.push(arr[i]);
			p.push(arr[i]);
		}

		while (!q.empty())
		{
			cout << q.top() << " ";  // 5 4 3 2 1
			q.pop();
		}
		cout << endl;

		while (!p.empty())
		{
			cout << p.top() << " ";  // 1 2 3 4 5
			p.pop();
		}
		cout << endl;
	}
```

![](https://img-blog.csdnimg.cn/db0940077c464826aa8d258944023a51.png)

可以发现果然如此，这样我们就能控制优先级的存储顺序了。

## 二、stack、queue、priority_queue的模拟实现

### 1.stack&queue的模拟实现

#### 总体思路：

通过上面的引入，我们发现了，stack以及queue的底层均是用deque容器实现的，那么我们利用其结构，进行简单的 **push、pop、top、empty、size**即可。

stack是 **先进后出，后进先出**。所以进即就是简单线性表的尾插，所以需要container适配器有push_back结构，出的话也就是每次都是尾删，也就是需要有pop_back结构。top就是简单的最后一个元素而已。

queue同理，只不过保持结构 **先进先出，后进后出**，需要push_back、pop_front....

上述大体可用一下思路图解释：

![](https://img-blog.csdnimg.cn/43f217565ec142aeb043657bbfb0e032.png)![](https://img-blog.csdnimg.cn/1f7147ad3750404fb147ccd317a8bf3e.png)

#### 完整代码：

// stack.h

```cpp
// 引入新内容 空间适配器 不再原生自己造，而是用其他封装好的空间
	// 保持结构 先进后出 后进先出即可
	template>
	class stack
	{
	public:
		void push(const T& val)
		{
			v.push_back(val);  // 套用容器模板
		}

		void pop()
		{
			v.pop_back();
		}

		T& top()
		{
			return v.back();
		}

		T top() const
		{
			return v.back();
		}

		bool empty() const
		{
			return v.empty();
		}

		size_t size() const
		{
			return v.size();
		}

	private:
		Containers v;
	};
```

// queue.h

```cpp
	template>
	class queue
	{
	public:
		void push(const T& val)
		{
			q.push_back(val);
		}

		void pop()
		{
			q.pop_front();
		}

		T& top()
		{
			return q.front();
		}

		T top() const
		{
			return q.front();
		}

		bool empty() const
		{
			return q.empty();
		}

		size_t size()
		{
			return q.size();
		}
	private:
		Container q;
	};
```

### 2.priority_queue的模拟实现

#### 总体思路：

通过引入我们知道了，此优先级队列实际上也就是一个堆结构。在堆结构的时候利用数组结构构造一个之前数据结构以及详细讲过了，感兴趣的可以去看一下哦~：[堆的c语言实现以及简单应用_柒海啦的博客-CSDN博客_堆在c语言](https://blog.csdn.net/weixin_61508423/article/details/124881991 "堆的c语言实现以及简单应用_柒海啦的博客-CSDN博客_堆在c语言")

以下实现使用适配器vector容器实现，首先是数组的随机访问（父子结点计算）、其次就是尾插效率也高，所以没有选择默认适配器deque。其次就是加入了仿函数代替原本的< >符号，这样在向上调整和向下调整的时候就可以控制优先级顺序了（大根堆和小根堆）。根据如上，我们也可以得到一个大致思路：

![](https://img-blog.csdnimg.cn/0319e7cdc48c443580cbc8b93abe977f.png)

#### 完整代码：

```cpp
// 默认大根堆
	template, class Compare = std::less>  // 默认给的仿函数是 < 符号
	class priority_queue
	{
	public:
		priority_queue()
		{}

		// 迭代器区间构造函数
		template
		priority_queue(InputIterator frist, InputIterator end)
		{
			// 先插入
			while (frist != end)
			{
				v.push_back(*frist);
				++frist;
			}

			// 利用向下调整  利用从最后开始的父节点开始
			for (int i = (v.size() - 1 - 1) / 2; i >= 0; --i)
			{
				adjust_down(i);
			}
		}

		void adjust_up(size_t child)
		{
			// 向上调整  logn
			size_t parent = (child - 1) / 2;
			while (child > 0)
			{
				// v[parent] < v[child]
				if (com(v[parent], v[child]))
				{
					std::swap(v[parent], v[child]); // 交换数据
					child = parent;
					parent = (child - 1) / 2;
				}
				else
					break;
			}
		}

		void push(const T& val)
		{
			// 插入，保持堆结构，进行向上调整
			v.push_back(val);
			adjust_up(v.size() - 1);
		}

		void adjust_down(size_t parent)
		{
			// 向下调整 logn
			size_t child = parent * 2 + 1;
			while (child < v.size())
			{
				// 防止越界 v[child] < v[child + 1]
				if (child + 1 < v.size() && com(v[child], v[child + 1]))
					++child;
				if (com(v[parent], v[child]))
				{
					std::swap(v[parent], v[child]);
					parent = child;
					child = parent * 2 + 1;
				}
				else
					break;
			}
		}

		void pop()
		{
			// 删除，删除堆顶元素，和尾元素交换，删除，向下调整
			std::swap(v[0], v[v.size() - 1]);
			v.pop_back();

			adjust_down(0);  // 向下调整  保持下方为堆结构
		}

		const T& top() const
		{
			return v[0];
		}

		bool empty() const
		{
			return v.empty();
		}

		size_t size() const
		{
			return v.size();
		}
	private:
		Container v;
		Compare com;
	};
```

欢迎大佬补充~
