---
link: https://blog.csdn.net/weixin_61508423/article/details/128043581
title: 【C++】通过哈希表实现map和set
description: 文章浏览阅读1k次，点赞2次，收藏2次。关联式容器-哈希-利用哈希表实现map和set容器。_c++ set哈希表遍历
keywords: c++ set哈希表遍历
author: 柒海啦 Csdn认证博客专家 Csdn认证企业博客 码龄3年 暂无认证
date: 2024-04-04T06:35:35.000Z
publisher: null
stats: paragraph=135 sentences=65, words=1275
---
## 前言

在前面，我们通过红黑树这一底层结构实现了map和set。它们是 **关联式容器**。而现在，我们将通过哈希表这一数据结构重新实现map和set，即 **unordered**系列的关联式容器。因为它们的遍历是 **无序**的，和平衡二叉树不同，不能做到排序。

既然不能做到排序，但是原本的map和set也能兼容这个功能，那么为什么要新增 **unordered_map**和 **unordered_set**呢？新增自然有其道理，因为是 **哈希**思想，即 **映射**关系， **查找的效率**非常高，让我们一起来看看吧~

> 红黑树实现map和set博客链接：
[【C++】map&set利用红黑树进行简单封装_柒海啦的博客-CSDN博客](https://blog.csdn.net/weixin_61508423/article/details/127883440?spm=1001.2014.3001.5501 "【C++】map&set利用红黑树进行简单封装_柒海啦的博客-CSDN博客")

![](https://img-blog.csdnimg.cn/503043d1a16f4acf807f18c0b2b81afa.png)

**目录**

[一、哈希概念](#%e4%b8%80%e3%80%81%e5%93%88%e5%b8%8c%e6%a6%82%e5%bf%b5)

[1.哈希函数](#1%e5%93%88%e5%b8%8c%e5%87%bd%e6%95%b0)

[2.哈希冲突](#2%e5%93%88%e5%b8%8c%e5%86%b2%e7%aa%81)

[1.闭散列解决哈希冲突](#1%e9%97%ad%e6%95%a3%e5%88%97%e8%a7%a3%e5%86%b3%e5%93%88%e5%b8%8c%e5%86%b2%e7%aa%81)

[1.1 线性探测](#11%20%e7%ba%bf%e6%80%a7%e6%8e%a2%e6%b5%8b)

[1.2 二次探测](#12%20%e4%ba%8c%e6%ac%a1%e6%8e%a2%e6%b5%8b)

[1.3 扩容问题](#13%20%e6%89%a9%e5%ae%b9%e9%97%ae%e9%a2%98)

[2.开散列解决哈希冲突](#2%e5%bc%80%e6%95%a3%e5%88%97%e8%a7%a3%e5%86%b3%e5%93%88%e5%b8%8c%e5%86%b2%e7%aa%81)

[二、闭散列实现哈希表](#%e4%ba%8c%e3%80%81%e9%97%ad%e6%95%a3%e5%88%97%e5%ae%9e%e7%8e%b0%e5%93%88%e5%b8%8c%e8%a1%a8)

[三、开散列实现哈希表](#%e4%b8%89%e3%80%81%e5%bc%80%e6%95%a3%e5%88%97%e5%ae%9e%e7%8e%b0%e5%93%88%e5%b8%8c%e8%a1%a8)

[四、利用开散列哈希表模拟实现map和set](#%e5%9b%9b%e3%80%81%e5%88%a9%e7%94%a8%e5%bc%80%e6%95%a3%e5%88%97%e5%93%88%e5%b8%8c%e8%a1%a8%e6%a8%a1%e6%8b%9f%e5%ae%9e%e7%8e%b0map%e5%92%8cset)

## 一、哈希概念

理解一个哈希概念是重要前提。

我们知道，在之前的顺序存储和红黑树中，我们插入和取出元素，都是通过 **比较**去找的。即通过相同元素之间比较关系建立起的数据结构。顺序结构的查找时间复杂度为O(N)，平衡搜索树结构的查找时间为树的高度O(log_2N)。

但是查找效率还是不高。前面说了，哈希的核心就是映射关系，即哈希的插入和取出元素不再通过比较去实现，而是通过 **元素本身去映射一个位置**。下次取的时候直接从这个映射位置找到即可。

> Hash，一般翻译做散列、杂凑，或音译为哈希，是把任意长度的[输入](https://baike.baidu.com/item/%E8%BE%93%E5%85%A5/5481954?fromModule=lemma_inlink "输入")（又叫做预映射pre-image）通过散列算法变换成固定长度的[输出](https://baike.baidu.com/item/%E8%BE%93%E5%87%BA/11056752?fromModule=lemma_inlink "输出")，该输出就是散列值。这种转换是一种[压缩映射](https://baike.baidu.com/item/%E5%8E%8B%E7%BC%A9%E6%98%A0%E5%B0%84/5114126?fromModule=lemma_inlink "压缩映射")，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。简单的说就是一种将任意长度的消息压缩到某一固定长度的[消息摘要](https://baike.baidu.com/item/%E6%B6%88%E6%81%AF%E6%91%98%E8%A6%81/4547744?fromModule=lemma_inlink "消息摘要")的函数。
Hash算法是一个广义的算法，也可以认为是一种思想，使用Hash算法可以提高存储空间的利用率，可以提高数据的查询效率，也可以做[数字签名](https://baike.baidu.com/item/%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D/212550?fromModule=lemma_inlink "数字签名")来保障数据传递的安全性。所以Hash算法被广泛地应用在互联网应用中。（百度百科）

那么重点就是如何去映射一个位置，已经关于此位置衍生出来的一系列问题。

### 1.哈希函数

通过上面我们了解到，想要元素映射到一个位置，就需要哈希函数进行映射。

下面我们通过一个例子引入哈希函数以及相关映射概念。

> ![](https://img-blog.csdnimg.cn/e297c570bd2f452f9fa045fb498635b2.png)
题目链接：
[387. 字符串中的第一个唯一字符 - 力扣（Leetcode）](https://leetcode.cn/problems/first-unique-character-in-a-string/ "387. 字符串中的第一个唯一字符 - 力扣（Leetcode）")

看到这一题，你的第一想法是什么？总不会是遍历每一个字符，然后每一个字符和字符串中的其他字符进行比较吧，这样的话效率太低了，时间复杂度为O(N^2)。

这个时候就可以借助哈希的思想，也是哈希函数- **直接寻址法**：

既然是不重复的字符，那么就和此字符的出现次数有关，我们可以将此字符映射到一个数组位置，此位置就代表此字符，然后当前位置就存储此字符的出现次数即可。题目要求我们找到第一个不重复的字符，只需要按顺序遍历每个字符，取出它映射下标的次数，为1就是第一个不重复的，直接返回对应字符即可。

那么为什么是直接寻址法呢？因为是字符串，所以字母只有26个字母，我们的数组也只需要26个空间即可，每次就直接对应一个地址即可：

```cpp
class Solution {
public:
    int firstUniqChar(string s) {
        int arr[26] = { 0 };
        for (auto e : s) arr[e - 'a']++;  //直接定址法
        for (int i = 0; i < s.size(); ++i)
        {
            if (arr[s[i] - 'a'] == 1) return i;
        }
        return -1;
    }
};
```

这样实现的时间复杂度就是O(N)，效率快了太多了。

如上也就可以看到e - 'a'就是一个直接定址法，此法比较常用也是最基本的hash函数。

> 常见的哈希函数：
1. 直接定制法
取关键字的某个线性函数为散列地址：Hash（Key）= A*Key + B 优点：简单、均匀 缺点：需要事先知道关键字的分布情况 使用场景：适合查找比较小且连续的情况。
2. 除留余数法
设散列表中允许的地址数为m，取一个不大于m，但最接近或者等于m的质数p作为除数，按照哈希函数：Hash(key) = key% p(p

可以看到，上面抛出了一个问题，那就是哈希冲突。什么是哈希冲突呢？

### 2.哈希冲突

先给定义：

> 对于 **两个数据元素**的关键字和 (i != j)，有 != ，但有：Hash( ) == Hash( )，即：不同关键字通过相同哈希哈数计算出 **相同**的哈希地址，该种现象称为哈希冲突或哈希碰撞。把具有不同关键码而具有相同哈希地址的数据元素称为"同义词"。

比如上述的哈希函数中的除留余数法，假设除数为10，那么0和10留下的余数就是一致的，此时两者就 **映射**到 **同一位置**上了。

但是直接定址法就不会出现问题，因为是一个元素映射的一个唯一的位置。但是，如果我们给的不是一串字母，而是1 20 890呢？这样的话，如果元素数值差异较大，但是数量较少的话会严重浪费储存空间，这不就和我们的预想反着来了嘛。所以才会有除留余数法。这样的话就不受元素差异影响了，只需考虑开多少空间的问题。

回到哈希冲突，如何解决这个问题呢？下面提供了两种解决方法：

#### 1.闭散列解决哈希冲突

我们使用数组对元素进行存储。

从问题的根源出发，原因就是哈希函数产生的哈希地址冲突了。那么 **我们不妨每个哈希地址设置一种状态**，没有插入就为空，第一次插入就设置为存在，第二次插入此位置，因为此位置的状态为存在，所以就往后找空位置。

如何向后找空位置呢？下面提供了两种方法：

#### 1.1 线性探测

**线性探测说直白点就是每次遇到冲突了就地址+1**，移动到下一个位置，如果还是冲突的话就继续+1遇到空位置为止。

那么删除呢？我们是直接把元素对应的哈希地址置为空吗？显然不是。因为通过上面的哈希冲突我们知道，那就是在 **初次映射得到的哈希地址一致**，那么重复的元素都是从当前位置开始往后进行线性探测，如果其中一个元素（后面还跟着元素）被置为空的话，那么就找不到后面的重复映射的元素了。所以我们还需要加入一种状态，那就是 **删除状态**。所以删除的时候就置为删除状态就可以了。

插入也改一下，在哈希冲突时，线性探测往后找，遇到空或者删除状态就插入。查找也非常简单，每次映射到对应的哈希地址，去找对应位置存储的元素，一致就返回，不一致往后一个一个去找，直到遇到空即可。

#### 1.2 二次探测

首先可以看一下线性探测的缺点：由于每次发生哈希冲突后都是连续的往后找，那么数据就很容易的堆积到一起，此时的 **冲突概率**就会越来越大，冲突越来越大的话就可以发现 **效率**会越来越慢。所以为了缓解这个问题，提出了二次探测。

实际上也就是在原来冲突的哈希地址的基础上，每次找下一个位置为(H0 + i ^ 2) % m的地址。其中H0位当前哈希地址，i为1234....，m为当前哈希表的总长。

插入删除查找类似，只不过不是一个一个往后找吗，而是每次以上面的公式去找下一个为位置。

#### 1.3 扩容问题

首先先谈谈容量问题。

你认为哈希表的扩容能像以前那样 **满了**直接扩吗？显然不能，因为存在 **冲突概率**。哈希表自然是为了提高效率的，如果一个哈希表中存储的全是冲突元素，那哈希的作用不就微乎其微了吗，那还不如直接遍历一遍数组省去哈希函数的麻烦。

所以我们需要降低冲突概率，冲突概率用负载因子进行计算，下面给出负载因子的计算公式：

> 负载因子=插入元素个数 / 散列表的长度

在大量实验证明下，一般闭散列（开放定址法）我们保持住0.7即可保证较高效率。即当哈希表中大于负载因子的时候就需要重新扩容。

扩容也不是简简单单直接将值拷贝过去即可，而是需要重新进行映射。 **复用插入代码**即可。

#### 2.开散列解决哈希冲突

闭散列我们可以明显的发现问题：一旦冲突的话就是一大片冲突。虽然二次探测稍微缓和了一点但是没有从 **本质上**解决问题。

那么我们可以这样想，如果发生了哈希冲突不是往后面找位置进行占用，而是 **挂在当前位置**下呢？所以，这里我们就可以借助单链表，数组也就变成了存放指针的数组。

开散列也称哈希桶，每个数组元素就相当于一个桶，桶里存放的就是存在哈希冲突的元素。插入的时候，就直接头插即可（尾插的话要往下也可以，但是效率不好）。寻找的话也是根据哈希地址找到对应的桶，然后遍历桶里的元素去寻找即可，删除就是找到对应元素的结点，链表的删除即可。

因为是开散列，所以只要指针数组满了就扩容即可。那么每次扩多大呢？经过实验发现，每次扩成一个素数，似乎效率更高，所以我们不妨每次以素数的大小去扩。另外，需要注意此时的扩容就不能像上面直接复用了，因为是指针，所以浅拷贝不说，另外桶在当前容量下是冲突的，但是一旦变了容量的话那就不一定冲突了。所以需要老老实实的每个结点再次插入。

## 二、闭散列实现哈希表

![](https://img-blog.csdnimg.cn/6ab570fe23264a889bb9db06ca25a79b.png)

代码如下：（下面只以线性探测为例，二次探测类似）

```cpp
// 闭散列
namespace CloseHash
{
	enum class State  // 设置状态，防止删除&插入&查找存在冲突
	{
		EMPTY,  // 空
		DELETE,  // 删除
		EXIST  // 存在
	};

	template
	struct HashElem  // 元素类型
	{
		pair _kv;
		State _state = State::EMPTY;  // 默认给空
	};

	template
	ostream& operator<& he)
	{
		out << "[" << he._kv.first << ":" << he._kv.second << "]";
		return out;
	}

	template>
	class HashTable
	{
	public:
		// 构造函数 - 可以一上来开空间
		HashTable(size_t capacity = 3)
			:_data(capacity), _size(0)
		{}

		// 插入
		bool Insert(const pair& kv)
		{
			// 防止数据冗余，首先对进入的kv的frist进行一个查找
			if (Find(kv.first)) return false;
			// 首先考虑扩不扩容
			CheckCapacity();
			size_t hashi = Hash()(kv.first) % _data.size();  // 除留余数法 使用了匿名对象
			while (_data[hashi]._state == State::EXIST)  // 遇到标记删除和空均可插入
			{
				hashi++;  // 线性探测
				// 注意是一个循环进行的过程，超过边界了没有遇到空或者删除就到第一个去，循环数组
				hashi %= _data.size();
			}
			_data[hashi]._kv = kv;
			_data[hashi]._state = State::EXIST;
			++_size;
			return true;
		}

		// 删除
		bool Erase(const K& key)
		{
			HashElem* tmp = Find(key);
			if (tmp == nullptr) return false;  // 没有找到
			tmp->_state = State::DELETE;
			--_size;
			return true;
		}

		HashElem* Find(const K& key)
		{
			if (_size == 0) return nullptr;
			int hashi = Hash()(key) % _data.size();
			while (_data[hashi]._state != State::EMPTY)
			{
				if (_data[hashi]._state == State::EXIST && _data[hashi]._kv.first == key)
				{
					return &_data[hashi];
				}
				hashi++;
				hashi %= _data.size();  // 循环查找
			}
			return nullptr;  // 没有找到
		}

		void CheckCapacity()  // 检查是否扩容 - 根据负载因子：负载因子 = 当前数据个数 / 当前存储空间长度 - 闭散列负载因子建议控制在0.7左右
		{
			// 此时扩容就不再是以前单纯的扩展空间和迁移数据了，对于哈希表来说，由于是根据除留余数法进行的，所以一旦存储长度发生变化，就需要重新进行映射
			if (_data.size() == 0 || _size * 1.0 / _data.size() >= 0.7)
			{
				size_t newcapacity = _size == 0 ? 10 : _data.size() * 2;
				HashTable newHash(newcapacity);  // 重新建立一个哈希表
				// 开完空间首先重新映射
				for (auto& e : _data)
				{
					if (e._state == State::EXIST) newHash.Insert(e._kv);
				}
				_data.swap(newHash._data);  // vector自带的swap交换函数
			}
		}

		void Print()
		{
			for (auto& e : _data)
			{
				if (e._state == State::EXIST) cout << "[" << e._kv.first << ":" << e._kv.second << "]" << " ";
			}
			cout << endl;
		}

		size_t Capacity()
		{
			return _data.size();
		}

		size_t Size()
		{
			return _size;
		}
	private:
		vector> _data;
		size_t _size;  // 实际有效数据个数
	};
}
```

哈希仿函数：

```cpp
template
struct HashFunc
{
	size_t operator()(const K& k)
	{
		return (size_t)k;  // 针对于数而言
	}
};
```

## 三、开散列实现哈希表

![](https://img-blog.csdnimg.cn/fc4f726b2d3644d39aafd2c9744a507f.png)

代码如下：

```cpp
namespace OpenHash  // 哈希桶 - 开散列 - 存在哈希冲突，就挂结点在下面
{
	template
	struct HashBucketNode
	{
		pair _kv;
		HashBucketNode* _next;

		HashBucketNode(const pair& kv)
			:_kv(kv), _next(nullptr)
		{}
	};

	template>
	class HashBucket
	{
		typedef HashBucketNode Node;
	public:
		HashBucket(size_t capacity = 3)
			:_size(0)
		{
			_data.resize(capacity, nullptr);  // 全部初始化为空
		}

		~HashBucket()
		{
			for (size_t i = 0; i < _data.size(); ++i)
			{
				Node* cur = _data[i];
				while (cur)
				{
					Node* next = cur->_next;
					delete cur;
					cur = next;
				}
				_data[i] = nullptr;
			}
		}

		bool Insert(const pair& kv)
		{
			if (Find(kv.first)) return false;  // 防止数据冗余
			CheckCapacity(); // 扩容

			size_t hashi = Hash()(kv.first) % _data.size();
			Node* cur = new Node(kv);
			cur->_next = _data[hashi];
			_data[hashi] = cur;
			++_size;
			return true;
		}

		bool Find(const K& key)
		{
			size_t hashi = Hash()(key) % _data.size();
			Node* cur = _data[hashi];
			while (cur)
			{
				if (cur->_kv.first == key) return true;
				cur = cur->_next;
			}
			return false;
		}

		bool Erase(const K& key)
		{
			if (_size == 0) return false;
			size_t hashi = Hash()(key) % _data.size();

			Node* pre = nullptr;
			Node* cur = _data[hashi];
			while (cur)
			{
				if (cur->_kv.first == key)
				{
					// 此时是头删
					if (pre == nullptr)
					{
						pre = cur->_next;
						_data[hashi] = pre;
					}
					else  // 中间删
					{
						pre->_next = cur->_next;
					}
					delete cur;
					--_size;

					return true;
				}
				pre = cur;
				cur = cur->_next;
			}
			// 没有找到
			return false;
		}

		size_t BucketsCount()  // 返回当前桶的个数 - 也就是哈希表的表长
		{
			return _data.size();
		}

		size_t BucketsSize()  // 当前数据个数
		{
			return _size;
		}

		void Print()
		{// 测试遍历用
			for (size_t i = 0; i < _data.size(); ++i)
			{
				Node* cur = _data[i];
				while (cur)
				{
					cout << "[" << cur->_kv.first << ":" << cur->_kv.second << "]";
					cur = cur->_next;
				}
			}
			cout << endl;
		}
	private:
		vector _data;
		size_t _size;  // 存储的有效数据个数

		// 每次返回一个素数 - 这样每次摸上一个素数 - 扩容就按这个标准来
		inline size_t __stl_next_prime(size_t n)
		{
			static const size_t __stl_num_primes = 28;
			static const size_t __stl_prime_list[__stl_num_primes] =
			{
				53, 97, 193, 389, 769,
				1543, 3079, 6151, 12289, 24593,
				49157, 98317, 196613, 393241, 786433,
				1572869, 3145739, 6291469, 12582917, 25165843,
				50331653, 100663319, 201326611, 402653189, 805306457,
				1610612741, 3221225473, 4294967291
			};

			for (size_t i = 0; i < __stl_num_primes; ++i)
			{
				if (__stl_prime_list[i] > n)
				{
					return __stl_prime_list[i];
				}
			}

			return -1;
		}

		void CheckCapacity()
		{
			// 扩展空间
			if (_data.size() == 0 || _size == _data.size())  // 哈希桶  控制负载因子不超过1 实际数据个数 / 哈希表长
			{
				vector newdata;
				newdata.resize(__stl_next_prime(_size), nullptr);  // 首先全部都是空的
				// 此时不可以像闭散列那样进行复用映射了，因为复用映射此时就是浅拷贝
				for (size_t i = 0; i < _data.size(); i++)
				{
					Node* cur = _data[i];
					while (cur)  // 依次重新进行映射，然后转移结点数据
					{
						Node* next = cur->_next;

						size_t hashi = Hash()(cur->_kv.first) % newdata.size();
						cur->_next = newdata[hashi];
						newdata[hashi] = cur;

						cur = next;
					}
					_data[i] = nullptr;
				}
				_data.swap(newdata);
			}

		}

	};
}
```

## 四、利用开散列哈希表模拟实现map和set

和当初利用红黑树结构封装类似，同样的我们需要哈希表实现一个迭代器。由于其是无序的，所以只需要设计一个向前遍历的 **迭代器**即可，其余操作类似，这里直接上代码了：

哈希表实现：

```cpp
namespace OpenHash  // 哈希桶 - 开散列 - 存在哈希冲突，就挂结点在下面
{
	template
	struct HashBucketNode
	{
		T _data;
		HashBucketNode* _next;

		HashBucketNode(const T& data)
			:_data(data), _next(nullptr)
		{}
	};
	// 模板类声明
	template
	class HashBucket;

	template
	struct __HashBucketIterator
	{
		typedef HashBucketNode Node;
		typedef HashBucket Hb;
		typedef __HashBucketIterator Self;

		Node* node;
		Hb* hb;

		__HashBucketIterator(Node* Node, Hb* Hb)
			:node(Node), hb(Hb)
		{}

		T& operator*()
		{
			return node->_data;
		}

		T* operator->()
		{
			return &node->_data;
		}

		Self& operator++()  // 前置++
		{
			if (node->_next)  // 存在就不用通过哈希表去找了
			{
				node = node->_next;
			}
			else
			{
				KeyOfT kot;
				size_t hashi = Hash()(kot(node->_data)) % hb->_data.size();
				Node* cur = hb->_data[++hashi];
				while (cur == nullptr)
				{
					hashi++;
					if (hashi >= hb->_data.size()) break;
					cur = hb->_data[hashi];
				}
				node = cur;
			}
			return *this;
		}

		bool operator==(const Self& s) const
		{
			return s.node == node;
		}

		bool operator!=(const Self& s) const
		{
			return !(s == *this);
		}
	};
	template
	class HashBucket
	{
		typedef HashBucketNode Node;

		template
		friend struct __HashBucketIterator;  // 友元类  注意模板类的声明

	public:
		typedef __HashBucketIterator iterator;

		iterator begin()
		{
			Node* cur = nullptr;
			for (size_t i = 0; i < _data.size(); ++i)
			{
				cur = _data[i];
				if (cur) break;
			}
			return iterator(cur, this);
		}

		iterator end()
		{
			return iterator(nullptr, this);
		}

		HashBucket(size_t capacity = 3)
			:_size(0)
		{
			_data.resize(capacity, nullptr);  // 全部初始化为空
		}

		~HashBucket()
		{
			for (size_t i = 0; i < _data.size(); ++i)
			{
				Node* cur = _data[i];
				while (cur)
				{
					Node* next = cur->_next;
					delete cur;
					cur = next;
				}
				_data[i] = nullptr;
			}
		}

		pair Insert(const T& data)
		{
			KeyOfT kot;
			iterator ret = Find(kot(data));
			if (ret != end()) return pair(ret, false);
			CheckCapacity(); // 扩容

			size_t hashi = Hash()(kot(data)) % _data.size();
			Node* cur = new Node(data);
			cur->_next = _data[hashi];
			_data[hashi] = cur;
			++_size;
			return make_pair(iterator(cur, this), true);
		}

		iterator Find(const K& key)
		{
			KeyOfT kot;
			size_t hashi = Hash()(key) % _data.size();
			Node* cur = _data[hashi];
			while (cur)
			{
				if (kot(cur->_data) == key) return iterator(cur, this);
				cur = cur->_next;
			}
			return iterator(nullptr, this);
		}

		bool Erase(const K& key)
		{
			if (_size == 0) return false;
			KeyOfT kot;
			size_t hashi = Hash()(key) % _data.size();

			Node* pre = nullptr;
			Node* cur = _data[hashi];
			while (cur)
			{
				if (kot(cur->_data) == key)
				{
					// 此时是头删
					if (pre == nullptr)
					{
						pre = cur->_next;
						_data[hashi] = pre;
					}
					else  // 中间删
					{
						pre->_next = cur->_next;
					}
					delete cur;
					--_size;

					return true;
				}
				pre = cur;
				cur = cur->_next;
			}
			// 没有找到
			return false;
		}

		size_t BucketsCount()  // 返回当前哈希表的表长
		{
			return _data.size();
		}

		size_t BucketsSize()  // 当前数据个数
		{
			return _size;
		}

	private:
		vector _data;
		size_t _size;  // 存储的有效数据个数

		// 每次返回一个素数 - 这样每次摸上一个素数 - 扩容就按这个标准来
		inline size_t __stl_next_prime(size_t n)
		{
			static const size_t __stl_num_primes = 28;
			static const size_t __stl_prime_list[__stl_num_primes] =
			{
				53, 97, 193, 389, 769,
				1543, 3079, 6151, 12289, 24593,
				49157, 98317, 196613, 393241, 786433,
				1572869, 3145739, 6291469, 12582917, 25165843,
				50331653, 100663319, 201326611, 402653189, 805306457,
				1610612741, 3221225473, 4294967291
			};

			for (size_t i = 0; i < __stl_num_primes; ++i)
			{
				if (__stl_prime_list[i] > n)
				{
					return __stl_prime_list[i];
				}
			}

			return -1;
		}

		void CheckCapacity()
		{
			// 扩展空间
			if (_data.size() == 0 || _size == _data.size())  // 哈希桶  控制负载因子不超过1 实际数据个数 / 哈希表长
			{
				KeyOfT kot;
				vector newdata;
				newdata.resize(__stl_next_prime(_size), nullptr);  // 首先全部都是空的
				// 此时不可以像闭散列那样进行复用映射了，因为复用映射此时就是浅拷贝
				for (size_t i = 0; i < _data.size(); i++)
				{
					Node* cur = _data[i];
					while (cur)  // 依次重新进行映射，然后转移结点数据
					{
						Node* next = cur->_next;

						size_t hashi = Hash()(kot(cur->_data)) % newdata.size();
						cur->_next = newdata[hashi];
						newdata[hashi] = cur;

						cur = next;
					}
					_data[i] = nullptr;
				}
				_data.swap(newdata);
			}

		}

	};
}
```

UnorderedMap：

```cpp
#pragma once
#include "HashTable.h"

// 封装哈希表实现的map结构
namespace OpenHash
{
	template>
	class UnorderedMap
	{
		struct MapKeyOfT
		{
			const K& operator()(const pair& kv)
			{
				return kv.first;
			}
		};
	public:
		typedef typename HashBucket, Hash, MapKeyOfT>::iterator iterator;

		UnorderedMap(size_t capacity = 3)
			:_hb(capacity)
		{}

		iterator begin()
		{
			return _hb.begin();
		}

		iterator end()
		{
			return _hb.end();
		}

		pair insert(const pair& kv)
		{
			return _hb.Insert(kv);
		}

		V& operator[](const K& key)
		{
			pair ret = _hb.Insert(make_pair(key, V()));
			return (ret.first)->second;
		}

		iterator find(const K& key)
		{
			return _hb.Find(key);
		}

		bool erase(const K& key)
		{
			return _hb.Erase(key);
		}

	private:
		HashBucket, Hash, MapKeyOfT> _hb;  // 底层封装哈希表
	};
}
```

UnorderedSet：

```cpp
#pragma once
#include "HashTable.h"

// 封装哈希实现的Set
namespace OpenHash
{
	template>
	class UnorderedSet
	{
		struct SetKeyOfT
		{
			const K& operator()(const K& key)
			{
				return key;
			}
		};
	public:
		typedef typename HashBucket::iterator iterator;

		iterator begin()
		{
			return _hb.begin();
		}

		iterator end()
		{
			return _hb.end();
		}

		pair insert(const K& key)
		{
			return _hb.Insert(key);
		}

		bool erase(const K& key)
		{
			return _hb.Erase(key);
		}

		iterator find(const K& key)
		{
			return _hb.Find(key);
		}

	private:
		HashBucket _hb;
	};
}
```
