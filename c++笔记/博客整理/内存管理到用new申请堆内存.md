---
link: https://blog.csdn.net/weixin_61508423/article/details/126213835
title: 【C++】内存管理到用new申请堆内存
description: 文章浏览阅读3.9k次，点赞11次，收藏25次。C/C++下的内存管理，然后由C语言用malloc在堆上申请内存过渡到C++语言用new在堆上申请内存观察这两者的区别和差异，以及探究new的底层实现原理
keywords: 【C++】内存管理到用new申请堆内存
author: 柒海啦 Csdn认证博客专家 Csdn认证企业博客 码龄3年 暂无认证
date: 2022-08-07T12:44:06.000Z
publisher: null
stats: paragraph=16 sentences=4, words=68
---
**目录**

[前言](#%e5%89%8d%e8%a8%80)

[一、C/C++中程序内存区域划分](#%e4%b8%80%e3%80%81c%2fc%2b%2b%e4%b8%ad%e7%a8%8b%e5%ba%8f%e5%86%85%e5%ad%98%e5%8c%ba%e5%9f%9f%e5%88%92%e5%88%86)

[二、C++使用new申请堆内存](#%e4%ba%8c%e3%80%81c%2b%2b%e4%bd%bf%e7%94%a8new%e7%94%b3%e8%af%b7%e5%a0%86%e5%86%85%e5%ad%98)

[1.new和delete的使用](#1new%e5%92%8cdelete%e7%9a%84%e4%bd%bf%e7%94%a8)

[2.new和delete的底层实现](#2new%e5%92%8cdelete%e7%9a%84%e5%ba%95%e5%b1%82%e5%ae%9e%e7%8e%b0)

## 前言

hello~❥(ゝω・✿ฺ) 大家好呀！欢迎能够看我的这一篇关于C++的学习笔记，让我们一起进步吧~

我们首先要了解到的是C/C++下的内存管理，然后由C语言用malloc在堆上申请内存过渡到C++语言用new在堆上申请内存观察这两者的区别和差异，以及探究new的底层实现原理！

## 一、C/C++中程序内存区域划分

我们知道，一份源文件经过 **预处理、编译、汇编、链接**步骤之后，就会变成一份二进制文件。此时其是存储在磁盘上的。当我们点击运行时，其属性加载进OS的PCB，排入cpu的运行队列成为一个进程。而此时此程序相关的二进制代码和数据才会进内存，但是此内存是虚拟内存，通常大小只有4g。

此时细化给程序分配的内存就如下图所示：

![](https://img-blog.csdnimg.cn/880f7fce79c04ebbb6a7642d312cedcc.png)

比如，区分如下变量内存空间，判断在那个区间：

```cpp
int a = 1;
static int b = 2;
int main()
{
	int c = 3;
	int* d = (int*)malloc(sizeof(int));
	if (d == NULL)
	{
		perror("malloc");
		return 0;
	}
	*d = 4;
	char e[] = "abcd";
	const char* f = "abcd";

	cout << a << b << c << *d << e << f << endl;
	free(d);
	d = nullptr;
	return 0;
}
```

a是全局变量，所以在数据段（全局数据）；b是静态全局变量，所以在数据段（静态数据）；c是局部变量，在栈帧中开辟，所以在栈上；d是地址，由栈指向堆内某块内存，所以d在栈上；*d是指针解引用，指向堆内存，所以在堆上；e同样是一个地址，所以在栈上；* **e此时指向栈内存，因为在之前编译器做的就是将"abcd"从常量区拷贝到栈上给e数组**；f是地址，所以在栈上；*f此时指向常量，常量区位于代码段（只读常量）。

了解了内存分配后，我们就可以知道平时c语言使用malloc函数的时候就是向堆内存申请空间。但是C++增加了自定义类型之后，我们想在 **堆上给自定义类型**开空间怎么办呢？同时自定义类型设计构造函数以及初始化的问题，这些该如何解决呢？C++就推出了 **new**操作符。

## 二、C++使用new申请堆内存

### 1.new和delete的使用

如图C语言中的malloc和free配对一样，C++中新增了new和delete进行配对。同样的是在堆上开辟空间，另一个就是在堆上释放空间。

> **new**:
**数据类型 变量名 = new 数据类型**;
在new数据类型后加 **(n)**表示给此堆变量赋初始值
在new数据类型后加 **[n]**表示是数组
数组类型赋给初始值：（注意C++11支持）
= new 数据类型[n]{...} ...中内置类型是0~n个数据，自定义类型必须满（一个类型需要多次赋值就在里面加{}）
**delete**:
**delete 变量名;**
如果是数组
**delete[] 变量名;**
注意：
new 对应 delete new [] 对应 delete[] 必须 **一一对应**，不对应的话会出问题。
**对于内置类型，delete可以等价于free**。

比如如下内置类型使用new申请堆空间：

```cpp
int main()
{
	int* a = new int;
	int* b = new int(1);
	int* c = new int[3];
	int* d = new int[3]{ 1, 2, 3 };
	cout << *a << endl;
	cout << *b << endl;
	for (int i = 0; i < 3; i++)
	{
		cout << c[i] << " ";
	}
	cout << endl;
	for (int i = 0; i < 3; i++)
	{
		cout << d[i] << " ";
	}
	cout << endl;
	delete a;
	delete b;
	delete[] c;
	delete[] d;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/10f80d49a2c24d9a995edb791e5eb18a.png)

> 可以通过上面的代码发现，new和malloc等的区别就是 **不用检查是否传回来是空和不用强转类型**，那么new是如何检查能否申请成功的呢？利用的是异常处理。--异常处理

比如可以向虚拟内存申请大概2g的空间，那么就会异常处理（x32位平台下）：

```cpp
int main()
{
	try
	{
		char* a = new char[1024u * 1024u * 1024u * 2 - 1];
		delete[] a;
	}
	catch (const exception& e)
	{
		cout << e.what() << endl;
	}
	return 0;
}
```

其中 **try和catch**就是接收抛出的异常，如果存在异常就进入下面的catch语句中，what函数就输出错误：

![](https://img-blog.csdnimg.cn/dfe9c9c992674bde98ec7d08426823b2.png)

-- 分配不当 --

那么初看对于内置类型，new与delete和malloc与free也没有多大的区别，最多就是malloc如果申请不到堆内存就返回NULL，而new的处理就是抛出异常。但是针对于自定义类型，这个差异就会非常大了：

```cpp
class Test
{
	int _num;
public:
	Test(int num = 0)
		:_num(num)
	{
		cout << "Test()构造" << endl;
	}
	~Test()
	{
		cout << "Test()析构" << endl;
	}
};

int main()
{
	Test* t = new Test;
	Test* t2 = new Test[3]{ 1, 2, 3 };
	delete t;
	delete[] t2;
	return 0;
}
```

![](https://img-blog.csdnimg.cn/d95ab86d2f534a639dc8cd681352fb40.png)

> 可以发现，new不仅仅在堆内申请空间，并且也 **调用了自定义类型的构造函数**。（赋的有初始值就调用对应的构造函数，否则调用默认构造函数），然后delete会在释放堆内存之前， **先调用析构函数**，然后才会释放空间。

### 2.new和delete的底层实现

new和delete终究是两个操作符，而malloc是一个函数。那么new和delete是如何完成申请空间的同时调用构造函数和析构函数呢？

首先，就不得不说两个重要的系统实现的全局函数：operator new 和 operator delete：

```cpp
/*
operator new：该函数实际通过malloc来申请空间，当malloc申请空间成功时直接返回；申请空间失败，
尝试执行空 间不足应对措施，如果改应对措施用户设置了，则继续申请，否则抛异常。
*/
void *__CRTDECL operator new(size_t size) _THROW1(_STD bad_alloc)
{
// try to allocate size bytes
void *p;
while ((p = malloc(size)) == 0)
if (_callnewh(size) == 0)
{
// report no memory
// 如果申请内存失败了，这里会抛出bad_alloc 类型异常
static const std::bad_alloc nomem;
_RAISE(nomem);
}
return (p);
}
```

```cpp
/*
operator delete: 该函数最终是通过free来释放空间的
*/
void operator delete(void *pUserData)
{
_CrtMemBlockHeader * pHead;
RTCCALLBACK(_RTC_Free_hook, (pUserData, 0));
if (pUserData == NULL)
return;
_mlock(_HEAP_LOCK); /* block other threads */
__TRY
/* get a pointer to memory block header */
pHead = pHdr(pUserData);
/* verify block type */
_ASSERTE(_BLOCK_TYPE_IS_VALID(pHead->nBlockUse));
_free_dbg( pUserData, pHead->nBlockUse );
__FINALLY
_munlock(_HEAP_LOCK); /* release other threads */
__END_TRY_FINALLY
return;
}
```

上面是两个函数的底层实现，可以大致的看出来实际上operator new用malloc申请堆内存，然后做了一些异常处理，同理，operator delete也是用free进行释放空间的。

然后我们可以查看其汇编代码：

![](https://img-blog.csdnimg.cn/32fabe7446e14503aaf561f4a31611c3.png)

就可以发现，new在调用operator new函数后即申请空间后就会去调用构造函数。同理，delete也是先调用的~析构，之后在释放空间。

综上：

> new：
调用operator new，里面malloc存在的意义：1.帮助new开空间，封装malloc，为了符合C++new的失败机制 -- 失败抛异常之后。之后调用构造函数。
delete：
调用析构函数清理，然后调用operator delete，里面封装了free，释放空间。

未完待续.......
