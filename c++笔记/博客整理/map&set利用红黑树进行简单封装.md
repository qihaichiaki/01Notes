---
link: https://blog.csdn.net/weixin_61508423/article/details/127883440
title: 【C++】map&set利用红黑树进行简单封装
description: 文章浏览阅读436次，点赞2次，收藏2次。基于红黑树实现简单封装map和set。_c++ map碎片化
keywords: c++ map碎片化
author: 柒海啦 Csdn认证博客专家 Csdn认证企业博客 码龄3年 暂无认证
date: 2022-12-04T02:27:45.000Z
publisher: null
stats: paragraph=59 sentences=63, words=356
---
## 前言

大家好~~~~呀！很荣幸你能点击这篇文章。本篇也是我的一份学习笔记，让我们一起共同成长吧~ing......

> C++红黑树的简单插入实现博客~
[【C++】红黑树的插入实现_柒海啦的博客-CSDN博客](https://blog.csdn.net/weixin_61508423/article/details/127814584?spm=1001.2014.3001.5501 "【C++】红黑树的插入实现_柒海啦的博客-CSDN博客")
二叉搜索树的基本结构和实现博客~
[【C++】二叉搜索树_柒海啦的博客-CSDN博客_c++二叉搜索树](https://blog.csdn.net/weixin_61508423/article/details/127699544?spm=1001.2014.3001.5501 "【C++】二叉搜索树_柒海啦的博客-CSDN博客_c++二叉搜索树")

本篇笔记基于红黑树的基本插入实现，实现简单的map和set的包装，掌握底层的基本原理即可~

![](https://img-blog.csdnimg.cn/5912c5c24be24c9ba95c5d3a2efc9cad.png)

（嘿嘿玉子阔爱~~~）

**目录**

[一、map和set的基本使用](#%e4%b8%80%e3%80%81map%e5%92%8cset%e7%9a%84%e5%9f%ba%e6%9c%ac%e4%bd%bf%e7%94%a8)

[1.set的使用](#1set%e7%9a%84%e4%bd%bf%e7%94%a8)

[1.1set的模板参数：](#11set%e7%9a%84%e6%a8%a1%e6%9d%bf%e5%8f%82%e6%95%b0%ef%bc%9a)

[1.2set的迭代器：](#12set%e7%9a%84%e8%bf%ad%e4%bb%a3%e5%99%a8%ef%bc%9a)

[1.3set的修改：](#13set%e7%9a%84%e4%bf%ae%e6%94%b9%ef%bc%9a)

[1.4set的操作：](#14set%e7%9a%84%e6%93%8d%e4%bd%9c%ef%bc%9a)

[1.5代码测试：](#15%e4%bb%a3%e7%a0%81%e6%b5%8b%e8%af%95%ef%bc%9a)

[2.map的使用](#2map%e7%9a%84%e4%bd%bf%e7%94%a8)

[2.1map的模板参数：](#21map%e7%9a%84%e6%a8%a1%e6%9d%bf%e5%8f%82%e6%95%b0%ef%bc%9a)

[2.2map的迭代器：](#22map%e7%9a%84%e8%bf%ad%e4%bb%a3%e5%99%a8%ef%bc%9a)

[2.3map的修改：](#23map%e7%9a%84%e4%bf%ae%e6%94%b9%ef%bc%9a)

[2.4map的操作：](#24map%e7%9a%84%e6%93%8d%e4%bd%9c%ef%bc%9a%c2%a0)

[2.5代码操作：](#25%e4%bb%a3%e7%a0%81%e6%93%8d%e4%bd%9c%ef%bc%9a)

[二、基于插入实现的红黑树进行封装](#%e4%ba%8c%e3%80%81%e5%9f%ba%e4%ba%8e%e6%8f%92%e5%85%a5%e5%ae%9e%e7%8e%b0%e7%9a%84%e7%ba%a2%e9%bb%91%e6%a0%91%e8%bf%9b%e8%a1%8c%e5%b0%81%e8%a3%85)

[1.优化红黑树的模板参数](#1%e4%bc%98%e5%8c%96%e7%ba%a2%e9%bb%91%e6%a0%91%e7%9a%84%e6%a8%a1%e6%9d%bf%e5%8f%82%e6%95%b0)

[上层代码初步实现：](#%e4%b8%8a%e5%b1%82%e4%bb%a3%e7%a0%81%e5%88%9d%e6%ad%a5%e5%ae%9e%e7%8e%b0%ef%bc%9a)

[下层红黑树代码：](#%e4%b8%8b%e5%b1%82%e7%ba%a2%e9%bb%91%e6%a0%91%e4%bb%a3%e7%a0%81%ef%bc%9a)

[2.红黑树的迭代器实现](#%c2%a02%e7%ba%a2%e9%bb%91%e6%a0%91%e7%9a%84%e8%bf%ad%e4%bb%a3%e5%99%a8%e5%ae%9e%e7%8e%b0)

[重载++/--运算符：](#%e9%87%8d%e8%bd%bd%2b%2b%2f--%e8%bf%90%e7%ae%97%e7%ac%a6%ef%bc%9a)

[3.map、set上层对红黑树迭代器的相关封装：](#3map%e3%80%81set%e4%b8%8a%e5%b1%82%e5%af%b9%e7%ba%a2%e9%bb%91%e6%a0%91%e8%bf%ad%e4%bb%a3%e5%99%a8%e7%9a%84%e7%9b%b8%e5%85%b3%e5%b0%81%e8%a3%85%ef%bc%9a)

[4.mapset综合简单封装代码：](#4mapset%e7%bb%bc%e5%90%88%e7%ae%80%e5%8d%95%e5%b0%81%e8%a3%85%e4%bb%a3%e7%a0%81%ef%bc%9a)

## 一、map和set的基本使用

在一开始，我们接触的C++STL容器比如：vector、list、stack、deque....这些容器的底层都是为线性序列的数据结构，存储的就是元素本身。这些就是 **序列式容器**。

除了序列式容器外，还有 **关联式容器**。它也是存储数据的，只不过和序列式容器不同的是内部存储的是 **key-value**结构的键值对，并且 **数据结构也是非线性**的（二叉树）。而现在我们要介绍的map和set均为关联式容器。

### 1.set的使用

> set文档：[set - C++ Reference (cplusplus.com)](https://legacy.cplusplus.com/reference/set/set/ "set - C++ Reference (cplusplus.com)")
multiset文档：[multiset - C++ Reference (cplusplus.com)](https://legacy.cplusplus.com/reference/set/multiset/ "multiset - C++ Reference (cplusplus.com)")
1.set是按照一定顺序进行存储的容器。
2.set/multiset 底层数据均是 ****。插入只需要一个key类型即可。
3.set的元素不可重复，multiset的元素可重复。
4.使用set的迭代器进行遍历可以得到有序序列。
5.set的元素默认按照小于来比较，查找某个元素时间复杂度为 **log_2 N**，并且元素 **不可修改**。
6.底层使用二叉搜索树-红黑树进行实现。
（注：下述以set为准，multiset使用接口类似）

#### 1.1set的模板参数：

![](https://img-blog.csdnimg.cn/c49f7b258d0a49ddabe0dd954187354c.png)

> T - 插入元素的数据类型。
Compare - 比较仿函数。默认给的less为库中实现的小于比较。
Alloc - 内存分配
（上述模板在模拟实现中目前只关注T）

#### 1.2set的迭代器：

![](https://img-blog.csdnimg.cn/3d1744e8b0564343a04f9c3adcda2973.png)

（解引用返回的就是T这个元素类型）

#### 1.3set的修改：

![](https://img-blog.csdnimg.cn/060acd4309ad4ca7ba36a46b32586d38.png)

> insert - 插入T类型的元素，或者在一段迭代器区间进行插入（模拟实现只实现第一种）![](https://img-blog.csdnimg.cn/fc141164b90248488a3e93c0c8749bc4.png) erase - 删除对应迭代器指向元素，或者一段迭代器区间的元素（模拟实现只实现第一种）
![](https://img-blog.csdnimg.cn/73aa24997e7b4cba911075e3de537042.png)

#### 1.4set的操作：

![](https://img-blog.csdnimg.cn/b465f4a806c14cc7986f021b570753a5.png)

> find - 根据传入的元素返回一个迭代器。没有找到返回的就是end()。
count - 根据传入的元素返回其出现的个数（对于multiset有效）

#### 1.5代码测试：

```cpp
void test2()
{
	//set的使用 - 底层使用的是红黑树进行封装的
	set s;
	int arr[] = { 4, 2, 1, 1, 4, 3, 9, 101, 87, -3 };
	// 特性：去重，迭代器访问是有序的
	for (auto e : arr) s.insert(e);
	set::iterator si = s.begin();
	while (si != s.end())
	{
		cout << *si << " ";
		//*si = 2;  set不允许修改值
		si++;
	}
	cout << endl;
	set> s2;  // 第二个是配置的仿函数 - 给其底层对应key的比较大小
	for (auto e : arr) s2.insert(e);
	si = s2.begin();
	while (si != s2.end())
	{
		cout << *si << " ";  // 101 87 9 4 3 2 1 -3
		si++;
	}
	cout << endl;
	// 可删除
	s2.erase(-3);
	s2.erase(s2.begin());
	si = s2.begin();
	while (si != s2.end())
	{
		cout << *si << " ";  // 87 9 4 3 2 1
		si++;
	}
	cout << endl;
	cout << s2.count(4) << endl;  // 返回对应个数 -- 对于不可冗余的set没啥用
	// 查找，返回迭代器 set具有const，无法修改
	cout << *s2.find(87) << endl;
	//cout << *s2.find(10) << endl;  // 返回的是end
	multiset ms(arr, arr + (sizeof(arr) / sizeof(arr[0])));  // 可以使用迭代器构造 - 使用的是multi 可以重复的使用元素哦~
	cout << ms.count(4) << endl;  // 2
}
```

运行结果：

![](https://img-blog.csdnimg.cn/6d21416fbd5d4ff6bb88db78b4b99b29.png)

### 2.map的使用

> map文档：[map - C++ Reference (cplusplus.com)](https://legacy.cplusplus.com/reference/map/map/ "map - C++ Reference (cplusplus.com)")
multimap文档：[multimap - C++ Reference (cplusplus.com)](https://legacy.cplusplus.com/reference/map/multimap/ "multimap - C++ Reference (cplusplus.com)")
1. map是关联容器，它按照特定的次序(按照 **key来比较**)存储由键值key和值value组合而成的元素（ **pair**）。
2.同样的默认小于进行比较，比较的元素也 **key**类型的元素。查找类型同样是key，时间复杂度也是log_2 N。并且key不可修改，但是对应的 **value**可以进行修改。
3.针对于key元素，利用迭 **代器遍历**可以得到有序序列。
4.支持[]进行访问。
5.map的key不可重复，multimap的key允许重复。
6.底层使用二叉搜索树-红黑树进行实现。
（注：下述以map为准，multimap使用接口类似）

#### 2.1map的模板参数：

![](https://img-blog.csdnimg.cn/509b10f137f94a3f8eafe9af12f40c0a.png)

> Key、T - 对应键值对中的key-value。之后插入比较均以key类型元素为基准。
Compare - 比较仿函数。默认给的less为库中实现的小于比较。
Alloc - 内存分配

#### 2.2map的迭代器：

![](https://img-blog.csdnimg.cn/61f688ec1d784f73aae376b4e5350f30.png) （此时返回的迭代器类型解用引用后是个pair

#### 2.3map的修改：

![](https://img-blog.csdnimg.cn/65725b35000e46f88d4a6e848f06023d.png)

> operator[] -
访问对应key下标的value值。如果key不存在，那么就进行新的插入，返回值为以T类型调用的默认构造。
at - 访问key下标对应的value值。如果key不存在，则抛异常。
insert -
![](https://img-blog.csdnimg.cn/4f119e9c5a9b43db9cc3b53cd12ba3e0.png) erase -
![](https://img-blog.csdnimg.cn/380a2f1319d44b62bb7f8e975e0fb70a.png)

#### 2.4map的操作：

![](https://img-blog.csdnimg.cn/c6c6838262ee40dd88e680b4af25dc61.png)

> find - 根据传入的元素返回一个迭代器。没有找到返回的就是end()。
count - 根据传入的key返回其出现的对数（对于multimap有效）

#### 2.5代码操作：

```cpp
void test3()
{
	// map的使用
	map m;
	m.insert(pair("China", "中国"));
	cout << m["China"] << endl;  // 重载的有[] - []功能强大
	m["Japan"] = "日本";  // 还能直接插入
	cout << m["Japan"] << endl;

	string ar[] = {"西瓜", "苹果", "香蕉", "香蕉", "苹果", "西瓜", "苹果", "香蕉", "西瓜", "苹果", "苹果", "苹果", "西瓜", "苹果", "香蕉"};
	map m2;
	for (auto e : ar)
	{
		m2[e]++;  // int() = 0 第一次就是0 + 1 = 1， 存在就++
	}
	auto it = m2.begin();
	while (it != m2.end())
	{
		cout << (*it).first << ":" << (*it).second << endl;
		it++;
	}
	cout << (*m2.find("西瓜")).second << endl;  // 同样的返回迭代器
	cout << m2.find("西瓜")->second << endl;  // 同样的返回迭代器
}
```

运行结果：

![](https://img-blog.csdnimg.cn/44783d6264bb48bbb91f53220c7376a6.png)

## 二、基于插入实现的红黑树进行封装

在了解了基本map和set的基本使用后，我们可以利用之前实现的简单插入实现的红黑树进行封装，文章链接在前言哦~

### 1.优化红黑树的模板参数

首先，因为我们实现的红黑树利用的K-Value模型。为了兼容set和map的存储结构（一个只是value、一个是key - value），我们就保留K这个模板参数，利用T这个模板参数一个存储value、一个存储pair

那么此时插入数据就是T类型的数据。由于是上层封装，所以传到红黑树这一层，就不知道T类型究竟是value还是一个键值对。此时同样需要上层传入一个仿函数，每次控制对应类型取出对应的key值。set key就是本身，map pair

![](https://img-blog.csdnimg.cn/2ab9d1e6dd0144439cabb19deaea88b2.png)

#### 上层代码初步实现：

map:

```cpp
namespace YuShen
{
	template
	class map
	{
		struct MapKeyOfT
		{
			const K& operator()(const pair& kv)  // 重载()
			{
				return kv.first;
			}
		};
	public:
    // ......
	private:
		RBTree, MapKeyOfT> _t; // 红黑树结构
	};
}
```

set：

```cpp
namespace YuShen
{
	template
	class set
	{
		struct SetKeyOfT
		{
			const K& operator()(const K& key)
			{
				return key;
			}
		};

	public:
    // ......
	private:
		RBTree _t;
	};
}
```

#### 下层红黑树代码：

```cpp
template
class RBTree
{
	typedef RBTreeNode Node;
    // .......

};

// 红黑树结点结构
template
struct RBTreeNode
{
    //......

};
```

此时原本的红黑树key和value模型就混在一起了，需要将所有需要取出key的地方（比如比较的地方）都要利用传入的仿函数对象进行比较，比如：

![](https://img-blog.csdnimg.cn/ee7b69097ac54a3e95e73cd800f98d58.png)

### 2.红黑树的迭代器实现

在上述的使用中，我们发现map和set都是可以控制迭代器的。实际上，它们调用的就是底层红黑树自己实现的迭代器。

此时迭代器也就需要自己实现了。首先红黑树是一个二叉树结构，非线性结构，和list的模拟实现类似。

其次红黑树是一个二叉搜索树。因为控制了所以结点值大于对应结点的左子树的值，小于对应结点的右子树的值的性质，所以对二叉搜索树来说，中序遍历是有序的。那么在接下来的迭代器实现中++或者--就要按照中序的性质进行实现：

> 红黑树迭代器简单实现：-模板类
模板参数：（STL迭代器三样）class T, class Ref, class Ptr （T T& T*）
重载运算符：
***解引用 ->箭头访问（指针） ==/!= ++ --**

解引用、箭头访问、== !=的简单实现：

```cpp
template  // T T& T*
struct __RBTreeIterator
{
	RBTreeNode* node;
	typedef __RBTreeIterator Self;

	__RBTreeIterator(RBTreeNode* root)
		:node(root)
	{}
	Ref operator*()
	{
		return node->_data;
	}

	Ptr operator->()
	{
		return &node->_data;
	}

	bool operator==(const Self& it) const
	{
		return it.node == node;
	}

	bool operator!=(const Self& it) const
	{
		return it.node != node;
	}
    // ..... ..

}
```

#### 重载++/--运算符：

由于实现红黑树的迭代器，当前迭代器存储的结点，要让它指向++后也就是中序的下一个。我们想想中序的时候是如何进行遍历的？左子树，当前结点，右子树。所以，当到此结点后，表示此结点的左子树已经遍历过了，我们就遍历到右子树即可。

到 **右子树就要找到右子树中的最左的结点**。但是，如果右子树为空呢？按照之前的中序递归方式，此时就会返回到上一层也就是父节点的地方。但是以下面的的一张图为例，有着两种不同的情况：

![](https://img-blog.csdnimg.cn/38cd31fe0ae244f8bef0568a7e572636.png)

如果当前结点为1，那么++后就是3结点。 但是如果当前结点为4的话，那么++后的结点就应该是5结点。此时找的 **条件就是孩子不是父右子树的节点**。一直找到空表示此时中序遍历到空了。

-- 是完全反过来的，首先当前结点，那么右子树已经遍历过了，此时找到 **左子树的最右结点**。如果为空，那么就找到 **孩子不是父左子树**的结点即可。

**简单代码实现**：

```cpp
	Self& operator++()
	{
		// 中序访问-只不过是碎片化的进行访问
		if (node->_right)
		{
			// 找到右子树最左结点 --
			node = node->_right;
			while (node->_left) node = node->_left;
		}
		else
		{
			// 右子树为空，直接访问父节点吗？不一定哦，需要向上找孩子不是父亲的右结点的那个祖先
			RBTreeNode* parent = node->_parent;
			while (parent && parent->_right == node)
			{
				node = parent;
				parent = node->_parent;
			}
			node = parent;
		}
		return *this;
	}

	Self& operator--()  // 反过来了
	{
		if (node->_left)
		{
			node = node->_left;
			while (node->_right) node = node->_right;
		}
		else
		{
			// 右子树为空，直接访问父节点吗？不一定哦，需要向上找孩子不是父亲的右结点的那个祖先
			RBTreeNode* parent = node->_parent;
			while (parent && parent->_left == node)
			{
				node = parent;
				parent = node->_parent;
			}
			node = parent;
		}
		return *this;
	}
```

### 3.map、set上层对红黑树迭代器的相关封装：

在实现了红黑树的迭代器后，就可以具体写出begin和end了，begin就当前红黑树存储的根结点。end只需要给一个空结点过去即可。

![](https://img-blog.csdnimg.cn/02be9b824ee64430b9ea9152c9bbb77b.png)

map和set首先对此红黑树类型中的iterator类型进行重命名。由于是通过域访问符::进行访问的，所以编译器编译的时候不好判断其究竟是静态变量还是类型，所以类型的前面就需要加上typename表示这是一个类型以保证编译过的去：

```cpp
// map
    typedef typename RBTree, MapKeyOfT>::iterator iterator;
// set
    typedef typename RBTree::iterator iterator;
```

那么对于插入insert，我们的返回值可以是一个pair，里面存放迭代器和一个bool值。插入失败就返回对应元素的迭代器和false，插入成功就返回插入新结点的迭代器和true值。

```cpp
		pairinsert(const K& key)
		{
			return _t.Insert(key);
		}
```

对于map来说，此结构还可以方便我们实现[]重载。因为在之前的使用中，我们发现对于map的[]，如果对应key数据存在就返回其对应的value值，如果不存在就会创建新的键值对。实际上就是在调用[]调用红黑树的inset，传入key和利用其V的默认构造进行插入，传入失败就说明存在元素，然后返回对应的迭代器的value值即可，插入成功同样返回迭代器的value值即可：

```cpp
		V& operator[](const K& key)
		{
			pair ret = _t.Insert(pair(key, V()));
			return ret.first->second;
		}
```

find同样的类似实现，返回迭代器即可。

### 4.mapset综合简单封装代码：

```cpp
// map.h
#pragma once
#include "RBTree.h"

// 利用自己实现的红黑树去封装map
// 实验基本功能：数据不冗余，基本实现insert、[]等功能
namespace YuShen
{
	template
	class map
	{
		struct MapKeyOfT
		{
			const K& operator()(const pair& kv)  // 重载()
			{
				return kv.first;
			}
		};
	public:
		typedef typename RBTree, MapKeyOfT>::iterator iterator;

		iterator begin()
		{
			return _t.begin();
		}
		iterator end()
		{
			return _t.end();
		}
		pair insert(const pair& kv)
		{
			return _t.Insert(kv);
		}

		V& operator[](const K& key)
		{
			pair ret = _t.Insert(pair(key, V()));
			return ret.first->second;
		}

		iterator find(const K& key)
		{
			return _t.find(key);
		}
	private:
		RBTree, MapKeyOfT> _t; // 红黑树结构
	};
}
```

```cpp
// set.h
#pragma once

namespace YuShen
{
	template
	class set
	{
		struct SetKeyOfT
		{
			const K& operator()(const K& key)
			{
				return key;
			}
		};

	public:
		typedef typename RBTree::iterator iterator;

		pairinsert(const K& key)
		{
			return _t.Insert(key);
		}

		iterator begin()
		{
			return _t.begin();
		}

		iterator end()
		{
			return _t.end();
		}

		iterator find(const K& key)
		{
			return _t.find(key);
		}
	private:
		RBTree _t;
	};
}
```

另外，加上红黑树中的find实现：

```cpp
	iterator find(const K& key)
	{
		Node* tmp = _root;
		KeyOfT kot;
		while (tmp)
		{
			if (key > kot(tmp->_data)) tmp = tmp->_right;
			else if (key < kot(tmp->_data)) tmp = tmp->_left;
			else return iterator(tmp);
		}
		return iterator(nullptr);
	}
```
