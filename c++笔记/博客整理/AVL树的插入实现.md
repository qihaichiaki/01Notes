---
link: https://blog.csdn.net/weixin_61508423/article/details/127761743
title: 【C++】AVL树的插入实现
description: 文章浏览阅读738次，点赞4次，收藏2次。介绍提高查找二叉树的效率的方法-平衡搜索二叉树中的实现方法之一AVL树_avl树插入陈越
keywords: avl树插入陈越
author: 柒海啦 Csdn认证博客专家 Csdn认证企业博客 码龄3年 暂无认证
date: 2023-01-29T07:53:02.000Z
publisher: null
stats: paragraph=144 sentences=79, words=1785
---
## 前言

针对于前面所实现的搜索二叉树，其实我们会发现一些问题，就是可能会演化为单支树或者随便两支子树高度相差大。如果二叉树的结构是这样的话就不能保证查找效率高，因为只能将树保持为左右子树差不多的高度才可以保证查找效率。

这样的搜索二叉树我们可以称之为平衡搜索二叉树。现在我们就开启其中一个实现树- **AVL树**，了解其 **插入方法**即其 **平衡树调整的基本结构**。

> 上一篇C++博客~
[【C++】二叉搜索树_柒海啦的博客-CSDN博客](https://blog.csdn.net/weixin_61508423/article/details/127699544?spm=1001.2014.3001.5501 "【C++】二叉搜索树_柒海啦的博客-CSDN博客")

![](https://img-blog.csdnimg.cn/0506512a085a46ceb49afcdae76aa71c.png)

**目录**​​​​​​​

[一、AVL树结构](#%e4%b8%80%e3%80%81avl%e6%a0%91%e7%bb%93%e6%9e%84)

[1.pair结构](#1pair%e7%bb%93%e6%9e%84)

[2.AVL结点结构](#2avl%e7%bb%93%e7%82%b9%e7%bb%93%e6%9e%84)

[AVL树结点结构：](#avl%e6%a0%91%e7%bb%93%e7%82%b9%e7%bb%93%e6%9e%84%ef%bc%9a)

[二、AVL树调节平衡](#%e4%ba%8c%e3%80%81avl%e6%a0%91%e8%b0%83%e8%8a%82%e5%b9%b3%e8%a1%a1)

[1.控制平衡因子](#1%e6%8e%a7%e5%88%b6%e5%b9%b3%e8%a1%a1%e5%9b%a0%e5%ad%90)

[2.旋转调节](#2%e6%97%8b%e8%bd%ac%e8%b0%83%e8%8a%82)

[左单旋：](#%e5%b7%a6%e5%8d%95%e6%97%8b%ef%bc%9a)

[右单旋：](#%e5%8f%b3%e5%8d%95%e6%97%8b%ef%bc%9a)

[左右双旋：](#%e5%b7%a6%e5%8f%b3%e5%8f%8c%e6%97%8b%ef%bc%9a)

[右左双旋：](#%e5%8f%b3%e5%b7%a6%e5%8f%8c%e6%97%8b%ef%bc%9a)

[三、AVL树代码实现](#%e4%b8%89%e3%80%81avl%e6%a0%91%e4%bb%a3%e7%a0%81%e5%ae%9e%e7%8e%b0)

[1.AVL树结点](#1avl%e6%a0%91%e7%bb%93%e7%82%b9)

[2.AVL树插入](#2avl%e6%a0%91%e6%8f%92%e5%85%a5)

[左单旋](#%e5%b7%a6%e5%8d%95%e6%97%8b)

[右单旋](#%e5%8f%b3%e5%8d%95%e6%97%8b)

[右左双旋](#%e5%8f%b3%e5%b7%a6%e5%8f%8c%e6%97%8b)

[左右双旋](#%e5%b7%a6%e5%8f%b3%e5%8f%8c%e6%97%8b)

[四、AVL树的验证](#%e5%9b%9b%e3%80%81avl%e6%a0%91%e7%9a%84%e9%aa%8c%e8%af%81)

[五、综合代码](#%e4%ba%94%e3%80%81%e7%bb%bc%e5%90%88%e4%bb%a3%e7%a0%81)

## 一、AVL树结构

在一开始的二叉树搜索树中，我们会发现有两个模型：key和key-value模型。对于key-value模型，按照我们上次实现搜索二叉树的思路，就会直接在结点结构里面加上value变量存key所对应的value值。

但是，这里我们使用一种 **键值对的结构**来代替上述key-value。

### 1.pair结构

```cpp
	template  // 简单的一个模板类
	struct pair
	{
		K frist;
		V second;

		pair()
			:frist(K())
			,second(V())
		{}
		pair(const K& k, const V& v)
			:frist(k)
			,second(v)
		{}
	};
```

上面是我模仿pair的基本结构自己写的简单代码。可以发现，这是一个模板类，需要给定类型，其中成员 **frist存储k，second成员存v**。库中同样存在实现，放在std命名空间下的：

![](https://img-blog.csdnimg.cn/a6ee6048349d402e9ab01291493a6f05.png) ![](https://img-blog.csdnimg.cn/992016add698443d8e91dca77186e436.png)

可以看到，上面每次构造一个pair对象都要传入类型，很烦，以后利用此结构进行多次插入的时候就很麻烦。那么能不能让编译器帮我们解决呢？也就是自动识别类型：

![](https://img-blog.csdnimg.cn/08367dc3ff224afd9c463ce277dc6914.png)![](https://img-blog.csdnimg.cn/229c3a93100c416a88bb0d07480dcd1e.png)

我们可以发现，make_pair本身是一个函数，根据我们传入的参数自动的实例化了一个pair对象然后传出来。这样就会方便很多。

在之后的AVL树结构中（在先前的二叉树搜索树中实现key和key-value模型均利用key进行比较）， **我们就可以利用pair的成员first即存储的key值进行比较**。

现在回到正题。既然数据是由pair结构来进行存储键值对，那么一个AVL树的结点内的结构该如何定义呢？

### 2.AVL结点结构

首先搜索二叉树的结构不能忘掉：Node* left，right分别指向左右子树。data使用pair。就用此结构能完成对平衡的调节吗？

我们其实从这里就可以探讨一下所谓平衡：基础的二叉搜索树之所以会出现那样的问题，根本在于在插入的时候并没有关注 **左右子树的高度差，其次当出现一边树比另一边树高的时候，应该将低的那一边树的结点给旋转下去**。

![](https://img-blog.csdnimg.cn/d9ed644413a9490b8e1739777fa91604.png)

可以看到上面所给的二叉树例子。当一个二叉树子树过长的时候，我们进行适当的旋转就可以达到降低高度的目的。所以针对高度差和旋转问题，我实现AVL树的时候引入的结构是比较好实现的 **平衡因子和三叉链（Node* parent）**。

#### **AVL树结点结构** ：

> ![](https://img-blog.csdnimg.cn/1e6baddfb87b46a58ac0cd7faa28629e.png)
**pair**：键值对容器。
**结点指针**：三叉链-分别指向左子树结点，右子树结点，父结点。
**平衡因子**：控制左右子树高度差。（利用规则， **每次新增在一个结点右++，左就--。初始化为0**）

那么，现在的关键在于，我们要如何控制变化平衡因子的规则，在何时进行旋转，并且对于的平衡因子也要进行变化。

## 二、AVL树调节平衡

实际上，调节平衡自然是要在插入新的元素中要做的事情（删除也需要，这里只讲AVL树基本构造，所以省去了删除操作）。

现在我们在原本二叉树结点的结构上定义了AVL树结点的结构：三叉链方便控制旋转，平衡因子检查左右子树高度。

那么现在我们为了控制整个树的平衡，AVL树的规则为 **任意的左右子树高度差不超过1**。这样的话，现在平衡因子的控制思路为： **左子树插入为--，右子树插入为++**。 **如果两子树高度差超过1，就需要旋转调节平衡**。

我引入例子来帮助理解我们理解AVL树是如何调节平衡的：

> 比如我们去插入如下数：4, 2, 6, 1, 3, 5, 15, 7, 16, 14（此时均代表key值）
在一开始从4开始到16插入按照搜索二叉树来走都没有违反AVL树的规则。
![](https://img-blog.csdnimg.cn/832c7a63f61143fe8cb3a86a6e5af188.png)
一旦插入14，我们发现此时的结构发生了变化
![](https://img-blog.csdnimg.cn/28249e2544f34855bf1e592b9fa65e20.png)
可以看到由于新插入的结点影响，导致右子树比左子树高。而此时根据平衡因子规则6结点的平衡因子就为2,15结点为-1。此时就该进行旋转：
![](https://img-blog.csdnimg.cn/60e583946afa4c5d8d63c911064bc66a.gif)

### 1.控制平衡因子

上面只是对一个实例进行初步的认知，总而言之，我们控制平衡因子的大致方法：

> 1.新结点在当前结点左子树插入就平衡因子-1，在右子树插入平衡因子就+1。
2.当前结点新增后平衡因子变为1或者-1，就要向上调整父结点的平衡因子。
3.如果当前结点变为2或者-2，说明右子树或者左子树高度差破坏了不超过1的AVL数规则，此时就要进行旋转调节平衡因子达到平衡的效果。

问题的关键在于第3点，当平衡因子的绝对值超过1的时候，我们就需要对 **多种抽象情况**进行旋转调节。

### 2.旋转调节

根据多种二叉树进行分析，我具象成了如下几种的插入情况：

#### 左单旋：

![](https://img-blog.csdnimg.cn/808f4a85dab8471dbe5dde0110552d19.png)![](https://img-blog.csdnimg.cn/201bff499eb64725a933180fe2e1f49a.png)

> 分析：
**旋转原因**：
观察图1，原本结构p结点平衡因子为1，R结点平衡因子为0。此时在R结点的右子树插入新结点，导致R结点平衡因子变为1，继续向上导致P结点平衡因子变为 **2**，引发旋转。
**旋转过程：**
由于此时右边层数高于左边层数，那么此时只需要将p和其 **左子树压下去**即可。根据 **二叉搜索树规则**，因为其均小于右子树（左小，右大），因为此时R的右子树（c）增加了一个长度，那么可以将p也视为在b（R的左子树）增加一个长度，具体操作为p作为R的新左子树头结点，将b接在p的右子树。此时就左旋转形成图2。
**旋转结果**：
R的平衡因子原本插入后变为1，p变为2。 **旋转后R和p的平衡因子均变为0。此时无需向上调整**。（子树只要不变为1或者-1也就是没有增加高度就不会向上调整）但是注意p为根结点的话就要特别处理。
**注**：
h表示子树长度，并且h>=0。R和P均为具体的结点，abc只是表示这一部分的子树。

#### 右单旋：

![](https://img-blog.csdnimg.cn/e40b50e32feb484a91b8ec765084b5e7.png)![](https://img-blog.csdnimg.cn/6becdc144b5a4193802bb59b9d7dd654.png)

> 分析：
**旋转原因**：
观察图1，原本结构p结点平衡因子为-1，L结点平衡因子为0。此时在L结点的左子树插入新结点，导致L结点平衡因子变为-1，继续向上导致P结点平衡因子变为 **-2**，引发旋转。
**旋转过程：**
由于此时左边层数高于右边层数，那么此时只需要将p和其 **右子树压下去**即可。根据 **二叉搜索树规则**，因为其均大于左子树（左小，右大），因为此时L的左子树（a）增加了一个长度，那么可以将p也视为在b（L的右子树）增加一个长度，具体操作为p作为L的新右子树头结点，将b接在p的左子树。此时就右旋转形成图2。
**旋转结果**：
L的平衡因子原本插入后变为-1，p变为-2。 **旋转后L和p的平衡因子均变为0。此时无需向上调整**。（子树只要不变为1或者-1也就是没有增加高度就不会向上调整）但是注意p为根结点的话就要特别处理。
**注**：
h表示子树长度，并且h>=0。L和P均为具体的结点，abc只是表示这一部分的子树。

#### 左右双旋：

![](https://img-blog.csdnimg.cn/3eb2a0b0704242b4bfa32737ec5b7154.png)![](https://img-blog.csdnimg.cn/52ba969373cb49689b2bcb8ceda41c01.png)![](https://img-blog.csdnimg.cn/d89c57e84ab14b9c8bab70a8a488a403.png)​​​​​​​

> 分析：
**旋转原因**：
实际上是在右单旋的场景下，对L结点的右子树b进行插入。由于此时插入的是中间位置（无论是图1中的c还是d子树，上述以插入c举例--注意插入c和d虽然和旋转无关但是和 **平衡因子有关**）所以单纯的右旋转只会转化为对称的一个高度差超出1的二叉搜索树而已。此时将LR变化为-1（从d新增变为1），L变成1，p结点平衡因子变为-2，从而引发左右双旋。
**旋转过程：**
既然右旋转不行，我们先单独看图1中根结点的左子树。由于此时是LR结点的子树高，我们此时想要降低高度目的也就是想将一个结点旋转下去，可以针对于L结点，将其进行左单旋变成图2.此时图中的平衡因子值为之前插入时变化的，并没有修改。
此时就可以演变为LR结点的左子树新增结点，此时再将P结点进行右单旋即可。如图三。
**旋转结果**：
旋转完后此时就发现是一个平衡树了。对应的将L的平衡因子修改为0，p的平衡因子修改为1（ **注意这两个平衡因子和一开始LR对哪个子树插入有关**）LR平衡因子肯定为0。同样的不需要向上调整了，这里也需要注意 **h为0**的情况，这种情况类似的进行分析。（如果h为0，此时的 **LR就是新增结点**）
**注**：
h表示子树长度，并且h>=0。L、LR和P均为具体的结点，abcd只是表示这一部分的子树。

#### 右左双旋：

![](https://img-blog.csdnimg.cn/647a82ad8db344768cfa974fe4dfab59.png)![](https://img-blog.csdnimg.cn/22a68139b8344fe5ab09f87ae515823d.png)![](https://img-blog.csdnimg.cn/a987bfd39f3b42d5b84d66229ae645d5.png)

> 分析：
**旋转原因**：
实际上是在左单旋的场景下，对R结点的左子树b进行插入。由于此时插入的是中间位置（无论是图1中的c还是d子树，上述以插入d举例--注意插入c和d虽然和旋转无关但是和 **平衡因子有关**）所以单纯的左旋转只会转化为对称的一个高度差超出1的二叉搜索树而已。此时将RL变化为1（从c新增变为-1），R变成-1，p结点平衡因子变为2，从而引发右左双旋。
**旋转过程：**
既然左旋转不行，我们先单独看图1中根结点的右子树。由于此时是RL结点的子树高，我们此时想要降低高度目的也就是想将一个结点旋转下去，可以针对于R结点，将其进行右单旋变成图2.此时图中的平衡因子值为之前插入时变化的，并没有修改。
此时就可以演变为RL结点的右子树新增结点，此时再将P结点进行左单旋即可。如图三。
**旋转结果**：
旋转完后此时就发现是一个平衡树了。对应的将R的平衡因子修改为0，p的平衡因子修改为1（ **注意这两个平衡因子和一开始RL对哪个子树插入有关**）RL平衡因子肯定为0。同样的不需要向上调整了，这里也需要注意 **h为0**的情况，这种情况类似的进行分析。（如果h为0，此时的 **RL就是新增结点**）
**注**：
h表示子树长度，并且h>=0。R、RL和P均为具体的结点，abcd只是表示这一部分的子树。

好了，了解了插入进行旋转平衡搜索二叉树后，我们直接用代码进行实操吧~

## 三、AVL树代码实现

### 1.AVL树结点

三叉链、pair、平衡因子。存在pair就需要两个模板参数：（注意给一个pair参数的构造函数）

```cpp
template  // 用来实例化pair的参数
struct AVLTreeNode  // AVL因为需要旋转，所以需要靠三叉链进行完成 并且加上平衡因子
{
	AVLTreeNode* left;
	AVLTreeNode* right;
	AVLTreeNode* parent;

	pair kv;
	int bf;  // 重要成员-平衡因子

	AVLTreeNode(const pair& data)
		:left(nullptr)
		,right(nullptr)
		,parent(nullptr)
		,kv(data)
		,bf(0)
	{}
};
```

### 2.AVL树插入

首先AVL树类的基本结构搭建起来：

```cpp
template
class AVLTree
{
public:
	typedef AVLTreeNode Node;

// ......

private:
	Node* _root = nullptr;  // 存一个根结点即可
};
```

首先按照搜索二叉树那样去新增结点：

```cpp
// 插入
	bool insert(const pair& data)
	{
		// 首先按照搜索二叉树走，右边插入此结点平衡因子++，左边插入--。此时检查平衡因子
		if (_root == nullptr)  // 如果第一次插入
		{
			_root = new Node(data);
			return true;
		}

		Node* p = nullptr;
		Node* c = _root;
		while (c)
		{
			if (c->kv.first > data.first)
			{
				p = c;
				c = c->left;
			}
			else if (c->kv.first < data.first)
			{
				p = c;
				c = c->right;
			}
			else  // 不允许数据冗余，返回false
			{
				return false;
			}
		}

		c = new Node(data);
		if (p->kv.first > c->kv.first)
		{
			p->left = c;
		}
		else
		{
			p->right = c;
		}
		c->parent = p;

// 调节平衡因子
// ......

    }
```

此时c为新增结点，p就要去修改平衡因子。和之前的说的那样，如果是左就--，右就++，如果此时p的平衡因子为1或者-1就要继续，为0结束，为2或者-2就要进行旋转。那么以p结点为循环进行：

```cpp
		while (p)
		{
			if (p->left == c)
			{
				p->bf--;
			}
			else
			{
				p->bf++;
			}

			if (p->bf == 0)
			{
				return true;  // 此时子树没有增长，不用往上调整了
			}
			else if (abs(p->bf) == 1)  // 子树增长，上面的平衡因子需要变化
			{
				c = p;
				p = p->parent;
			}
			else if (abs(p->bf) == 2)  // 不符合AVL树的性质，需要进行旋转
			{
            // ......

            }
			else  // 不可能出现，出现了说明之前的结构就不对，检查上述代码
			{
				assert(false);
			}
		}

		return true;
```

结合上面我们所讨论的旋转的四种情况，根据此时p结点和c结点的平衡因子进行判断分为如下几种情况：

```cpp
				//左单旋 - 最右增加
				if (p->bf == 2 && c->bf == 1)
				{
					RotateL(p);
				}
				else if (p->bf == -2 && c->bf == -1)  // 右单旋 - 最左增加
				{
					RotateR(p);
				}
				else if (p->bf == 2 && c->bf == -1)  //右左双旋
				{
					RotateRL(p);
				}
				else  // 左右双旋
				{
					RotateLR(p);
				}
				break;
```

针对每个旋转实现也如下：（将其定义为私有方法）

#### 左单旋

```cpp
void RotateL(Node* p)  // 左单旋
	{
		Node* subR = p->right;
		Node* subRL = subR->left;  // 小心此结点可能为空

		if (subRL) subRL->parent = p;
		p->right = subRL;
		Node* pp = p->parent;  // 记录一下上一层结点
		p->parent = subR;
		subR->left = p;
		if (pp == nullptr)  // 如果p为根结点
		{
			subR->parent = nullptr;
			_root = subR;  // 别忘了修改属性
		}
		else
		{
			if (pp->left == p) pp->left = subR;
			else pp->right = subR;
			subR->parent = pp;
		}
		// 修改因子
		p->bf = subR->bf = 0;
	}
```

#### 右单旋

```cpp
	void RotateR(Node* p)  // 右单旋
	{
		Node* subL = p->left;
		Node* subLR = subL->right;  // 小心此结点可能为空

		if (subLR) subLR->parent = p;
		p->left = subLR;
		Node* pp = p->parent;  // 记录一下上一层结点
		p->parent = subL;
		subL->right = p;
		if (pp == nullptr)  // 如果p为根结点
		{
			subL->parent = nullptr;
			_root = subL;  // 别忘了修改属性
		}
		else
		{
			if (pp->left == p) pp->left = subL;
			else pp->right = subL;
			subL->parent = pp;
		}
		// 修改因子
		p->bf = subL->bf = 0;
	}
```

#### 右左双旋

```cpp
	void RotateRL(Node* p)  // 右左双旋
	{
		Node* subR = p->right;
		Node* subRL = subR->left; // 双旋变化的三个结点
		int bf = subRL->bf;

		RotateR(p->right);
		RotateL(p);
		// 旋好后注意平衡因子

		subRL->bf = 0;  // 双旋后的所在子树头结点，一定为0
		if (bf == 0)
		{
			subR->bf = 0;
			p->bf = 0;
		}
		else if (bf == 1)
		{
			//一开始subRL的右子树新增 ，此时到了subR的左子树
			subR->bf = 0;
			p->bf = -1;
		}
		else if (bf == -1)
		{
			p->bf = 0;
			subR->bf = 1;
		}
		else
		{// 理论上不存在此种情况，出现报错检查前面的问题
			assert(false);
		}
	}
```

#### 左右双旋

```cpp
	void RotateLR(Node* p)  // 左右双旋
	{
		Node* subL = p->left;
		Node* subLR = subL->right; // 双旋变化的三个结点
		int bf = subLR->bf;

		RotateL(p->left);
		RotateR(p);
		// 旋好后注意平衡因子

		subLR->bf = 0;  // 双旋后的所在子树头结点，一定为0
		if (bf == 0)  // 自己为新增结点 h为0
		{
			p->bf = 0;
			subL->bf = 0;
		}
		else if (bf == 1)
		{
			p->bf = 0;
			subL->bf = -1;
		}
		else if (bf == -1)
		{
			subL->bf = 0;
			p->bf = 1;
		}
		else
		{// 理论上不存在此种情况，出现报错检查前面的问题
			assert(false);
		}
	}
```

## 四、AVL树的验证

上述插入AVL树的大致结构完成了。那么我们真正的去构造AVL树的时候如何去判断其是不是呢？

首先判断其中序是否满足有序（二叉搜索树）

其次判断每个结点对应的左右子树高度差是否和对应的结点平衡因子相等或者绝对值超过1了，代码简单实现如下（递归实现）：

```cpp
	bool IsBalanceTree()
	{  // 检查是否是平衡二叉树
		return _isBalanceTree(_root);
	}
private:
	bool _isBalanceTree(Node* root)
	{
		if (root == nullptr) return true;
		int leftheight = isheight(root->left);
		int rightheight = isheight(root->right);
		int n = rightheight - leftheight;
		if (n != root->bf || abs(n) > 1) return false;
		return _isBalanceTree(root->left) && _isBalanceTree(root->right);
	}

	int isheight(Node* root)
	{
		if (root == nullptr) return 0;
		int left = isheight(root->left);
		int right = isheight(root->right);
		return left > right ? left + 1 : right + 1;
	}
```

## 五、综合代码

综上，简单实现AVL树结构的代码如下：（测试代码自己写哦~）

```cpp
// AVL树 - 二叉搜索树优化，控制树的左右高度差不超过1 - 本版本利用平衡因子进行实现
template  // 用来实例化pair的参数
struct AVLTreeNode  // AVL因为需要旋转，所以需要靠三叉链进行完成 并且加上平衡因子
{
	AVLTreeNode* left;
	AVLTreeNode* right;
	AVLTreeNode* parent;

	pair kv;
	int bf;  // 重要成员-平衡因子

	AVLTreeNode(const pair& data)
		:left(nullptr)
		,right(nullptr)
		,parent(nullptr)
		,kv(data)
		,bf(0)
	{}
};

template
class AVLTree
{
public:
	typedef AVLTreeNode Node;
	AVLTree() = default;  // 编译器必须生成默认构造

	// 查找
	bool find(const K& k)
	{
		Node* c = _root;
		while (c)
		{
			if (c->kv.first > k)
			{
				c = c->left;
			}
			else if (c->kv.first < k)
			{
				c = c->right;
			}
			else
			{
				return true;
			}
		}
		return false;
	}

	// 插入
	bool insert(const pair& data)
	{
		// 首先按照搜索二叉树走，右边插入此结点平衡因子++，左边插入--。此时检查平衡因子
		if (_root == nullptr)  // 如果第一次插入
		{
			_root = new Node(data);
			return true;
		}

		Node* p = nullptr;
		Node* c = _root;
		while (c)
		{
			if (c->kv.first > data.first)
			{
				p = c;
				c = c->left;
			}
			else if (c->kv.first < data.first)
			{
				p = c;
				c = c->right;
			}
			else  // 不允许数据冗余，返回false
			{
				return false;
			}
		}

		c = new Node(data);
		if (p->kv.first > c->kv.first)
		{
			p->left = c;
		}
		else
		{
			p->right = c;
		}
		c->parent = p;

		// 调节平衡因子
		while (p)
		{
			if (p->left == c)
			{
				p->bf--;
			}
			else
			{
				p->bf++;
			}

			if (p->bf == 0)
			{
				return true;  // 此时子树没有增长，不用往上调整了
			}
			else if (abs(p->bf) == 1)  // 子树增长，上面的平衡因子需要变化
			{
				c = p;
				p = p->parent;
			}
			else if (abs(p->bf) == 2)  // 不符合AVL树的性质，需要进行旋转
			{
				// 大致分3种情况，右单旋，左单旋，双旋。双旋分为先左单旋、再右单旋；先右单旋，再左单旋。根据抽象出来的bf进行决定
				//左单旋 - 最右增加
				if (p->bf == 2 && c->bf == 1)
				{
					RotateL(p);
				}
				else if (p->bf == -2 && c->bf == -1)  // 右单旋 - 最左增加
				{
					RotateR(p);
				}
				else if (p->bf == 2 && c->bf == -1)  //右左双旋
				{
					RotateRL(p);
				}
				else  // 左右双旋
				{
					RotateLR(p);
				}
				break;
			}
			else  // 不可能出现，出现了说明之前的结构就不对，检查上述代码
			{
				assert(false);
			}
		}

		return true;
	}

	void order()
	{
		// 非递归中序遍历
		std::stack s;
		Node* c = _root;

		while (c || !s.empty())
		{
			if (c)
			{
				s.push(c);
				c = c->left;
			}
			else
			{
				c = s.top();
				cout << c->kv.first << ":" << c->kv.second << " ";
				c = c->right;
				s.pop();
			}
		}
		std::cout << endl;
	}

	bool IsBalanceTree()
	{  // 检查是否是平衡二叉树
		return _isBalanceTree(_root);
	}
private:
	bool _isBalanceTree(Node* root)
	{
		if (root == nullptr) return true;
		int leftheight = isheight(root->left);
		int rightheight = isheight(root->right);
		int n = rightheight - leftheight;
		if (n != root->bf || abs(n) > 1) return false;
		return _isBalanceTree(root->left) && _isBalanceTree(root->right);
	}

	int isheight(Node* root)
	{
		if (root == nullptr) return 0;
		int left = isheight(root->left);
		int right = isheight(root->right);
		return left > right ? left + 1 : right + 1;
	}

	void RotateL(Node* p)  // 左单旋
	{
		Node* subR = p->right;
		Node* subRL = subR->left;  // 小心此结点可能为空

		if (subRL) subRL->parent = p;
		p->right = subRL;
		Node* pp = p->parent;  // 记录一下上一层结点
		p->parent = subR;
		subR->left = p;
		if (pp == nullptr)  // 如果p为根结点
		{
			subR->parent = nullptr;
			_root = subR;  // 别忘了修改属性
		}
		else
		{
			if (pp->left == p) pp->left = subR;
			else pp->right = subR;
			subR->parent = pp;
		}
		// 修改因子
		p->bf = subR->bf = 0;
	}

	void RotateR(Node* p)  // 右单旋
	{
		Node* subL = p->left;
		Node* subLR = subL->right;  // 小心此结点可能为空

		if (subLR) subLR->parent = p;
		p->left = subLR;
		Node* pp = p->parent;  // 记录一下上一层结点
		p->parent = subL;
		subL->right = p;
		if (pp == nullptr)  // 如果p为根结点
		{
			subL->parent = nullptr;
			_root = subL;  // 别忘了修改属性
		}
		else
		{
			if (pp->left == p) pp->left = subL;
			else pp->right = subL;
			subL->parent = pp;
		}
		// 修改因子
		p->bf = subL->bf = 0;
	}

	void RotateRL(Node* p)  // 右左双旋
	{
		Node* subR = p->right;
		Node* subRL = subR->left; // 双旋变化的三个结点
		int bf = subRL->bf;

		RotateR(p->right);
		RotateL(p);
		// 旋好后注意平衡因子

		subRL->bf = 0;  // 双旋后的所在子树头结点，一定为0
		if (bf == 0)
		{
			subR->bf = 0;
			p->bf = 0;
		}
		else if (bf == 1)
		{
			//一开始subRL的右子树新增 ，此时到了subR的左子树
			subR->bf = 0;
			p->bf = -1;
		}
		else if (bf == -1)
		{
			p->bf = 0;
			subR->bf = 1;
		}
		else
		{// 理论上不存在此种情况，出现报错检查前面的问题
			assert(false);
		}
	}

	void RotateLR(Node* p)  // 左右双旋
	{
		Node* subL = p->left;
		Node* subLR = subL->right; // 双旋变化的三个结点
		int bf = subLR->bf;

		RotateL(p->left);
		RotateR(p);
		// 旋好后注意平衡因子

		subLR->bf = 0;  // 双旋后的所在子树头结点，一定为0
		if (bf == 0)  // 自己为新增结点 h为0
		{
			p->bf = 0;
			subL->bf = 0;
		}
		else if (bf == 1)
		{
			p->bf = 0;
			subL->bf = -1;
		}
		else if (bf == -1)
		{
			subL->bf = 0;
			p->bf = 1;
		}
		else
		{// 理论上不存在此种情况，出现报错检查前面的问题
			assert(false);
		}
	}
private:
	Node* _root = nullptr;  // 存一个根结点即可
};
```

欢迎大佬补充~未完待续~~~
