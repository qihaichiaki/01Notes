## C++是如何工作的  
* C++程序的流程：C++源文件-编译器->二进制文件(库/可执行程序)  
	1. 预处理-实际编译发生前被处理了，``#include <iostream>``  
	2. 编译-C++代码转换为机器代码  
	3. 链接-将所有生成的obj（cpp->编译后的文件，每一个cpp源文件）链接为一个文件  

* C++中同一个项目多个文件之间存在相互调用的情况（以windows下的vs为例）：  
	- 编译阶段： C++代码操作中，涉及到了多个文件互相调用的情况，需要做提前声明，告知**编译器**在此文件中的某个函数或者变量是被**定义**了的。但是实际上，编译器并不知道是否真正的被定义了。但是选择相信声明的那个  **声明-过编译**  
	- 链接阶段： 能够过编译阶段，但是实际上最终使用的时候，我们需要对应变量或者函数的实际**定义**，因为这样才能运行程序。此时你声明的事物应该在其他文件中被**定义**了的，经过链接操作，链接器会将之前编译阶段生成的obj文件链接混合为一个exe文件。如果正确被定义，则成功，否则失败。**定义-过链接**  
		* error-找不到外部链接 - 链接失败  


## C++编译器是如何工作的  
* 源文件-编译->obj中间文件  、
* 每一个Cpp文件就是一个翻译单元，目的是告诉编译器我们需要将此文件翻译为obj文件  
	- 实际上一个cpp文件也可以包含其他cpp文件  
1. 预处理：对于预处理语句进行处理  
	- 预处理语句：``#include、#if、#pragma、#define``  
	- ``#include`` -很简单，复制粘贴目标文件到当前文件中  
	- 预处理后结果可以生成.i文件（可以看到对源文件预处理后的实际结果）  
	- 预处理语法  
	> #define 目标字符 属性 ->可以自定义字符替代后面的属性、或者是函数  
	> #if #endif #elif  在if和endif包围的区域根据if后面的值判断是否裁剪或者保留  

2. 编译：.i文件处理为汇编语言.a（已经是机器要做的指令了，只不过没有翻译为2进制，人类是可读的）  
	- 在汇编过程中，调用函数的地方-会根据**函数签名**进行调用。实际上连接器就是根据此进行链接的。此函数签名在全项目中唯一出现  
	- 如果启用了优化选项。在程序编写代码的基础上，对**重复的，多余的操作**优化，对于一些经常需要用到的地方进行优化（比如将变量优化到寄存器上，增加读取的速度...）  

3. 汇编：.a文件翻译为机器语言-二进制.obj  

## C++链接器是如何工作的  
* 目的：找到每个符号和函数在哪里，把它们链接起来  
* main函数同样需要进行链接。因为这是一个应用程序的起点  
### static静态函数  
* 当然，如果我们想让一个函数，只在当前文件可见，意思是生成obj文件，不会参与到链接的过程中去，可以在函数定义的开头增加**static**关键字，告诉链接器此不用链接到外部文件，外部文件也不会链接到此函数  

* 常见的链接错误：  
	1. 找不到确切的函数定义（返回值、参数、函数名），返回一个链接错误  
	2. 重复的链接符号（函数一致...），返回一个链接错误  
		- 通常犯错在**include**文件上，因为可能多个文件include一个头文件，此头文件中只存在一个函数定义，但是预处理后多份cpp文件就会生成多份定义。此时将编译器生成的obj链接在一起的时候就会出现重复的链接情况！**解决方案**：1.将头文件中定义的函数设置为static，这样在每个obj中对应的函数是局部本文件的，不会产生链接冲突、2.将函数设置为展开函数-inline，即将函数调用直接替换为函数体（注意针对函数代码量少的情况）3.include的文件中应该只存在函数声明，定义放在一个文件中（通常是对应文件名的.cpp文件-存放定义），此时定义只在一个函数中定义，不会出现链接重复问题  

## C++基本数据类型  
* 基本数据类型构成了我们程序中存储的任何数据  
	- 唯一区别：内存所占大小  
	- 语义区分可以由我们程序员定义-C++是强大的！，但是通常存在一些约定  
* int - 传统上为4byte  
	- 表示范围：21亿到-21亿左右，是有符号整数  
	- 分析数据：4byte = ``4*8`` = 32bit 31位的二进制数表示的范围（注意其中1位表示符号）  
	- 指定unsigned 表示无符号整数，此时是32位二进制数表示范围-只能是正数  42亿左右  
* char - 1byte  
	- 内部存储的数字如果被命令行打印出来会根据ascii码表表示出来字符（实际上是和当前环境下的字符编码相关）  
	- 
* short 2byte  
* long 4byte  
* long long 8byte  
* float 4byte  
	- 注意数值的小数默认位double类型，需要在后面加上f/F来表示其是float类型  
* double 8byte  
* bool 1byte  
	- 因为在寻址变量的时候是按byte去寻找的，无法按照1bit去寻找，所以只能设置1byte单位的内存  
	- 动一点小脑筋：1个bool类型可以存储8个0or1的值哦  


* 注意：实际变量类型对应的内存大小和编译器相关，上述是在vs的参考下  

* C++中调用sizeof() 查看变量所占用的字节大小  


## C++函数  
* 目的：为了解决重复代码的问题  
* 返回类型 函数名 参数列表 函数体  
* 返回使用关键字return，函数体使用{}包裹  
* 如果不返回变量，使用void作为返回类型  

* 函数会在栈结构上进行压栈操作-不同函数调用会来回反复切换。如果将无意义或者不需要写成函数的代码包装成函数会大大降低性能  
* 注意要点：函数的目的->**防止代码重复**  

## C++头文件  
* 语言中类似C++存在翻译单元-编译的语言会存在C++头文件这一类型的说法  
* 头文件一般用于声明一些变量、函数用于其他未定义想使用的源文件中通过编译  
	- 为了解决多个文件中均需要声明未定义的函数/变量，我们可以创建头文件，去减少这些复制粘贴的行为（实际上编译还是复制粘贴，只不过让我们的编译器在预处理阶段帮助我们做了）  

* 但是这样会带来一个隐患，设想一下：假设存在头文件A，B包含了头文件A，C包含了头文件B，但是同时，他也包含了头文件A。此时会发生什么变化？头文件A被重复包含替换了两次在C翻译单元中，重复包含了显然会增加文件大小影响使用（如果头文件中存在定义则会变成多次定义的结果）  
	- 所以存在一种预处理模式，vs中使用#pragma once，或者更通用的宏定义``#ifndef __MY_HEAD__ #endif``，来防止当前头文件在**同一个翻译单元(源文件)**被复制粘贴两次。
	- 原理就是（#ifndef）：第一次包含的时候监测当前文件是否定义此宏，没有定义就会定义（配合#define定义符号）并且不会裁剪内部的头文件内容。如果定义了则表示这是第二次（或者多次）包含了此头文件，预处理阶段会将其裁剪掉，只保留一次结果  

* 现在通常喜欢使用简洁一点的#pragma once，大部分编译器均支持这样的语法  
* 对于#include，需要知道的是：  
	1. 使用尖括号``<>``，通常指定文件在编译器设置的一个目录里面  
	2. 使用引号``""``，通常是指定相对源文件的文件位置，但是也可以指定在系统设置的目录里（相对位置没有搜索到的话）  

* 在C++中，通常和C库作为区分，没有后缀名，而C内部包含库一般存在后缀名.h  

## Visual Studio-debug  
* 断点和读取内存 -   
* 记住一个原则：计算机是对的。  
* 前提：debug模式  
* f9设置断点  f11进入运行的函数内部  f10跳转到下一行  shift+f11回到调用函数的位置  f5继续执行，会跳转到下一个断点  

* 在编译器的处理过程中，会准备变量，其内存中会全部设置cc，提示程序员当前此处的内存-变量并没有初始化  
* 因为是debug模式，程序会做出一些额外的事情，在release模式下就会优化掉  

## C++的条件语句  
* if、else if、else...  
* 实际上，这些分支和跳转都会极大的消耗性能开支-因为不断指定机器指令跳转到内存的不同地方，所以一些优化的代码会**特别避免分支**  
* C++中的else if实际上不是关键字，而是else{if}  ->只是将它们放到了一行而已（编译器没有做任何的处理，类似于 else log("test"); => else if(true) ...）  
* 编程可以分为数学编程和逻辑编程。数学编程就是提高代码效率的关键  

## Visual Studio最佳设置  
* 有时候自己的文件需要和vs下的文件做好区分，需要自己去设置：点击显示所有文件，创建src文件，拖入即可。一般此时不展示过滤器结构即可（虚拟的）  
* 输出目录：``$(SolutionDir)bin\$(Platform)\$(Configuration)\`` 当前项目目录下->bin目录下->x32、x64->debug模式还是release模式  
* 中间目录：``$(SolutionDir)bin\intermediates\$(Platform)\$(Configuration)\`` 将一些中间文件放在intermediates目录下，平台和模式也要标注好  

* 注意需要vs保存到模板，这些设置才会生效，<a href="https://www.cnblogs.com/metahuber/p/vs-project-template.html">参考文章</a>  

## C++循环  
* for循环，第一段开始时运行一次 每次判断是否进入循环 当前循环结束执行语句  
* 另外：for(int i;;)的i会在循环结束后自动释放（因为i生命周期为for循环，出了自然自动进行回收），但while无法做到这一点，不可能在循环内部定义，也不可能在外面定义之后自动释放  

## C++控制流语句  
* return、break、continue  
* 控制程序的流程  

## C++指针  
* 一个程序中最重要的就是内存  
* 指针：是一种存储内存地址的数字  
* 0意味着没有，无效的内存地址->设定上的空指针  
* 获得变量的地址可以使用``&``  
* 逆向引用-解引用可以读写指针所保存的那个地址中存储的值 ``*指针变量``  

## C++引用  
* 实际上引用就是指针的一个扩展  
* 引用必须"引用"，类似于变量的别名  
* ``类型&``  
* 相对于指针，使用简单简洁  

## C++类  
* 类是：抽象的数据类型组合体：函数和属性  
* 存在可见性-public、private...  
* 类中的函数被称作方法  
* 让代码更加简洁  

## C++中类与结构体  
* 技术层面上：struct默认public可见度，class默认private可见度  
* 语义层次上：  
	- 只表示变量的结构 - POD 适合struct  
		* 注意，这并不代表不可以添加方法，只要是围绕这这几个简单的变量处理的都可以  
	- 继承中使用class，这是一种复杂的功能过程  
	- 简而言之：简单的结构使用struct，复杂使用class  

## 设计C++类  
* 设计一个类或者API的时候，需要反向思考，从如何使用它进行下手  
* 一般设计类中的成员变量，可以在前面增加``_``,保持代码干净，并且很有帮助  
	- 编码习惯：``m_``表示后面是一个成员变量（Member），后面接大驼峰  
* 类中书写的时候，可以将方法和变量书写在不同的部分，即使它们都是属于公开或者私有的  


## C++中的static  
* 使用存在区别，区别于所在的上下文：是否在类和结构体中，以及局部变量  
1. 类和结构体外部的static：声明为static的符号，链接将只在内部-只能对你定义它的翻译单元可见  
	- 此时可以类比为：将翻译单元类比为class，而此时设置了static的变量或者方法相当于在private可见域中，其他翻译单元对其是**不可见**的  
	- 编码习惯：``s_`` 表示这个变量是一个静态（static）的。后面是大驼峰  
2. 类和结构体内部的静态变量，函数，将与所有实例共享内存  
	- 不通过类的实例去调用这些变量或者函数  
	- 注意在类中的静态变量只是声明，需要在外部定义才能使用（或者加上const，在类内部进行初始化）  
	- 当你想要在对应类共享此变量，或者都与此类型相关的时候，就可以定义类的静态成员，因为这样只会被实例化一次，和类同生命周期。  
	- 静态方法不能访问**非静态变量**，因为此时函数并没有传递属于当前类实例的``*const this``指针  
3. 局部静态变量（local static）  
	* 相当于将局部变量的生命周期提升到了整个程序的生存期。注意作用域还是当前局部的。但是可以将其作为返回值传递出去（利用引用或者指针）  
	* 可以利用这一点，在类的单例模式中，可以使用静态局部变量代替类的静态成员属性。也可以做到同样的效果。（C++11中，静态局部变量是线程安全的）  
		- 问题：如何析构呢？难道单例类创造出来一定伴随着程序的生命周期吗？（感觉似乎是的）  
		- 用途是简化代码的书写  

## C++中的extern  
* 声明一个全局变量在当前翻译单元中去**外部翻译单元**寻找对应符号的全局变量  


## C++的枚举  
* enum 枚举集合名字  
* 本质：数值集合，作为类型向外提供  
* 里面的数值只能是**整数**，并且可以定义变量名-实际上是整数，但是可以在代码中使用变量名用于代替此整数。中间由``,``隔开，依次递增，并且可以赋值  
* 可以指定给枚举赋值的整数类型：默认为int，使用方法类似于继承的写法，在enum名字后面增加``: unsigned char``  
* 给特定的整数或者整数组命名的一种方式  
* 并且以此为类型即可限制在枚举的这些数值里面  -**限制条件**  
* 注意此时的类型在作用域上没有什么作用，不能将其当作命名空间  
* 注意区分：枚举类-``enum class``,此时可以将其当作一种命名空间  

## C++构造函数  
* 是对应类型初始化的地方  
* 每次构造一个对象实例都会调用的方法  
* 默认的构造函数对基本数据类型不做处理  

## C++析构函数  
* 对象生命周期到期（被摧毁时），被调用(注意调用析构函数并不会**摧毁对象**)。  
* 避免内存泄漏。  
* 或者利用这样的特点，制作一个智能指针或者锁守卫.....  

## C++继承  
* 避免代码重复，提供向后扩展的功能  
* 扩展现有类并为基类提供新功能的一种方式  

## C++虚函数  
* 能够允许基类中的方法在子类中构成重写  
* virtual-创建虚函数表  
* 实现动态编译，达到同一种类型表现出多种不同类型的效果  
* C++11引入了override，在派生类重写函数的参数括号后面-更具备可读性，还可以防止我们将函数名写错  
* 运行时成本：  
	1. 额外的内存存储虚函数表  
	2. 运行的时候会遍历虚函数表找到映射的目标函数-额外的性能损失  
* 对于运行效率，可能在嵌入式平台上存在一定的影响，其余影响不是很大，可以放心使用  

## C++纯虚函数  
* 基类中定义一个没有实现的函数，强制子类去实现该函数，否则还是抽象类。抽象类是无法实例化出对象的  

* 可以利用此特性设置一个接口类，保证类有一个特定的方法，将其（抽象基类）作为参数放入一个通用的函数中，调用此方法-达到不同类型的方法一致  

## C++的可见性  
* 访问修饰符：private(私有，友元和本身可见)、protected（本身和继承可见）、public（公开的）  
* 可见性是让代码更加容易维护，容易理解，不管是阅读代码还是扩展代码  

## C++的数组  
```c++
	int example[5];
	int* ptr = example;
```

* 同一类型的元素集合  
* 注意数组的边界检查  
* 数组实际上就是一个指针，使用指针偏移也可以达到写入、读取数组内存的操作  
* 本质：只是一个连续的数据块，可以使用索引一样索引它们，写到特定的页面内去  
* 存在生存期和内存间接寻址（通过地址找到堆内存上的数组），可以使用new关键字创建堆上的数组。  
* sizeof对直接的数组指针（栈上创建的），显示的是整个数组占用的实际内存大小  
* 对于在栈上创建的数组，给的数组元素数量必须是编译时就知道的常量大小，不能是变量  
* C++11中的数组：  
	- std::array  
	- 优势：存在边界检查、数组大小  

## C++字符串  
* 字符串实际上就是字符数组  
* 每个字符串实际上存在一个`\0(编码为00)`为结束符号表示该字符串结束，C中字符串会默认在后面添加（字符数组的话注意需要自己去添加）  
* C++中的字符串：``std::string => std::baseString<char> ``  
* 字符串的拷贝通常效率低下，传递参数的时候，利用引用减少拷贝，效率上可以进行优化  

## C++字符串字面量  
* 字符串字面量是存储在内容的只读部分的  
* 一般修改字符数组的时候，通过正常从常量区拷贝过来到栈、或者堆上内存的时候，就可以进行修改对应的字符，但是``char*``这种指针是不行的  

* 宽字符类型``wchar_t`` 一个字符占用多个字节（不一定是2byte?win - 2、linux - 4），其中的字符串字面量应该在前面加上``L"hello"``  
* ``char16_t char32_t``也是常用的类型，对应字符串字面量前面加上``u"hello"、U"hello"``  
* 默认字符串字面量前面加的就是``u8"hello"``  
* C++14中：``using namespace std::string_literals;``提供了一些方便的字符串函数  
	- 初始化string时，解决两个字面量相加的问题，利用运算符重载函数s，可以完成字面量相加然后赋值构造string类型：``std::string str = "hello"s + "word"``  
	- R在字符串字面值前面加上，可以忽略转移字符（实际的换行还是起效果的）  

## C++中的const | mutable  
* 一个机制，让我们的代码更加干净，写代码存在特定的限制  
* 语法上指定对应变量为一个常变量  
* 对于指针，在``*``左边可以定值，右边可以定向  
* 在类中，成员方法后面添加this，表示其中的隐藏this指针``*``前面增加了const，这样此类的const实例可以使用这个成员方法（因为``const *const  this``，不会发送越级使用）  
	- 在对于的成员方法里面也就不能对类中的属性进行修改了  
	- 如果想在const方法下修改类中的某些属性，可以在某些属性前面增加修饰符``mutable``,这样在const方法中也可以修改对应的属性  
* 代码总结：  
```c++
	class Demo
	{
		int* _x;
    public:
    	const int* const GetX() const
    	{
            return _x;
    	}
	}
```

## C++中的mutable  
* mutable的作用：  
	1. 如果想在const方法下修改类中的某些属性，可以在某些属性前面增加修饰符``mutable``,这样在const方法中也可以修改对应的属性  
	2. lambda中想要通过值传入，并且想对其进行修改的时候，需要在``{}``前面加上mutable方可生效  
* 在const下的应用场景就是仍然想在const对象下能够调用，但是在const方法中想要修改一些和对象无关紧要的变量，比如统计这个方法调用了多少次。此时mutable就很有用了  

## C++中的构造函数初始化列表  
* 构造函数中初始化类成员  
* 在构造函数和参数后面，增加``:``和``,``间隔，是c++程序真正初始化类成员的地方  
* 需要注意，类初始化成员不会按照初始化列表的顺序，而是按照类中定义类成员的顺序进行初始化  
* 在一些自定义类型初始化时，初始化列表能够提供直接一次的初始化，而不是构造两次消耗性能  
```c++
class Test
{
	string _name;
	int _score;
public:
  	Test()
  		: _name("qihai"), _score(666)
	{}  
};
```

## C++中的三元操作符  
* ``?:`` if语句的语法糖  
* 在一些复杂数据结构面前，能够提升性能（减少中间临时对象的产生）  

## C++创建对象  
* 栈上对象、堆上对象  
* 区分：栈是自动回收（根据当前作用域的生命周期）（栈通常很小），堆需要我们手动回收（堆通常很大）  
	- 堆上分配内存性能比栈消耗大  

## C++ new关键字  
* new关键字不仅分配内存，还会调用构造函数  
* ``new[] - delete[] new - delete malloc - free``  
* placement new - 指定内存地址初始化你想new的对象：  
```c++
	int* b = new int[50];
	Entity* e = new(b) Entity;
```

## C++隐式类型转换和explicit关键字  
* C++允许编译器对代码执行一次隐式转换  
* 一般隐式类型转换是构造函数中存在对应类型的唯一参数，此时就可以发生隐式类型转换  
* explicit在构造函数前使用，作用就是防止出现隐式类型转换  

## C++运算符及其重载  
* 基于运算符新的含义，添加、创建参数  
* ``operator运算符`` 从左到右参数决定在运算符中从左到右的位置。注意类中存在隐藏参数this  

## C++中的this关键字  
* this指向当前对象实例的指针，该方法属于这个对象实例  

## C++的对象生存期  
* 作用域：函数作用域、if、循环作用域、类作用域（基本就是{}所包裹的区域）  
* 基于栈的变量，一旦出作用域，就会被释放或者摧毁；  
* 基于作用域做的很多自动化代码：智能指针、作用域锁等  

## C++的智能指针  
* 智能指针的出现，就是为了解决new出来的堆内存地址能够在对象不需要使用的时候自动回收的一种方式（使用栈内存的生命周期来控制管理new和delete的时机，从而自动化管理）  
* 智能指针：（memory头文件）  
	1. unique_ptr: 唯一的智能指针，不可copy  
		* 注意使用其最好的方式是：``std::unique_ptr<Test> test = std::make_unique<Test>();（C++14）`` 这样可以有效避免构造函数抛出异常（导致申请的内存得不到回收-造成内存泄漏），这样安全一些  
		* 低开销，最简单的智能指针对象  
	2. shared_ptr：共享的智能指针，可以copy，采用了引用计数的方式回收内存  
		* 使用最好的方式：``std::shared_ptr<Test> test = std::make_shared<Test>();（C++14）``,这个除了避免异常带来的内存泄漏，还能将两次内存申请（1：new Test 2：new shared中引用计数）整合一起增加效率  
		* 所有的栈对象追踪的同一个堆内存均死亡后才会将堆内存进行释放  
		* 但是存在循环引用问题  
	3. weak_ptr: 弱指针，配合shared_ptr一起使用，解决循环引用问题。  
		* 可以被赋值为shared指针对象  
		* 不会增加shared中引用技术，生命周期到了也不会释放其内存  

## C++的拷贝和拷贝构造函数  
* 让其什么时候工作，是效率高的体现。-C++拷贝  
* 每当编写一个变量被赋值另一个变量的代码时，你总是在复制（引用除外）  
* class String类实现，引入拷贝和拷贝构造函数  

* 拷贝构造函数：
	- 默认拷贝构造函数：内置类型值拷贝（浅拷贝），自定义类型调用默认拷贝构造  
	- 程序员实现拷贝构造函数的时候，一般是要进行深拷贝的时候  

## C++的箭头操作符  
* 在指针对象调用属性和方法的时候，使用箭头操作符代替其解引用然后调用。者本身只是一个快捷方式  
* 其他情况：在智能指针实现过程中，需要运算符重载箭头符号让智能指针指向的指针正常访问属性和方法  
* 还有一种可以使用箭头运算符来获取内存中某个值的偏移量  
```c++
	struct Vector3
	{
        float x, y, z;
	};
	
	int main()
	{
        int offset = (int)(&((Vector3*)nullptr)->y); // nullptr为一个空指针，是一个指针类型  
        // "指针->属性"访问属性的方法实际上是通过把指针的值和属性的偏移量相加，得到属性的内存地址进而实现访问。而把指针设为nullptr(0)，然后->属性就等于0+属性偏移量。编译器能知道你指定属性的偏移量是因为你把nullptr转换为类指针，而这个类的结构你已经写出来了(float x,y,z)，float4字节，所以它在编译的时候就知道偏移量(0,4,8)，所以无关对象是否创建
        // 因为只是取了地址，但是没有访问，所以不会发生错误
        return 0;
	}
```

## C++的动态数组  
* ``Vector``  
* vector动态数组最好尽量使用对象（而不是指针），这样对象内存会存在一条线上（连续分配），效率会很高。指针会分布在各个内存中，需要动态寻找。但是在复制、扩容等方面，指针更甚一筹  
* 基础操作：  
  1. push_back尾插  
  2. erase删除，参数传入迭代器  

* 更优化的方式使用vector类  
	- 优化目标：vector不断的申请空间，复制等操作  
1. 避免频繁扩容：一开始vector使用方法reserve()提前给出大小（或者直接扩容到对应大小）  
2. 避免传递复制：使用方法``emlace_back();`` 只是传递构造函数的参数列表，这样就可以避免频繁拷贝  

## C++静态链接库  
* C++库的典型分布：includes、library-目录和库目录  
* 静态链接编译器会优化链接，理论上速度是快于动态链接的，但是一旦出现问题，静态链接的程序需要修改后全部编译，但是动态库却不需要  
* 在工程目录下建立``Dependencies(依赖关系)``的文件夹，管理依赖项目（C++三方库）  
* windowsVs下建立链接：``项目属性->C/C++->General->指定包含目录include/Linker-input 增加xxx.lib文件名，General设置附加库目录-Add``  
	- 指定包含目录路径时，不可绝对路径（别人使用就无法了，需要根据当前工程文件确定相对路径）：``$(SolutionDir)`` -解决方案路径（此括号后面不需要加\表示该目录，需要写文件名（在解决方案路径下的）比如：``$(SolutionDir)Test\...``）  

## C++动态链接库  
* 彼此彼此 - 动态链接库需要可执行文件同时载入内存，和静态链接不同，不会在链接阶段和源代码的二进制文件链接在一起，而库也是和程序独立出来的二进制文件  
* 在vs下使用动态链接库：  
	- 项目属性->C/C++->General->Add include(和静态库一致) 头文件  
	- 项目属性->Linker->Input 将对应的lib可以修改为dll.lib(文件中一般存在：一堆指向dll文件的指针，一般需要和dll同时编译) 和dll（dll文件简单的做法是可以放在和可执行程序一样的路径下，就可以执行）  

## C++创建和使用库  
* 注意exe文件下General项目配置：**Configuration Type**为Application(.exe)，静态库项目配置设置为Static library(.lib),动态库：Dynamic Library(.dll)  
* 确保所有平台都存在设置  
* 定位头文件，推荐使用在项目配置中>C++->General->Add include 通过**``$(SolutionDir)``**设置绝对路径  
* 如果是在当前项目下链接我们自己编译好的静态库，不需要之前那样设置lib文件，而是在exe项目右键Add-Reference添加即可（这样全是自动化，无需关心依赖项是否编译（自动编译））  

## C++处理多返回值  
* 在一些情况下，一个方法可能会存在多个返回值，但是默认情况下，C++只允许返回一个  
	- 对于同一类型的可以使用数组或者容器进行解决  
	- 通过函数参数列表。通过引用或者指针，变成输出型参数也不失为解决此的一种选择，但是总感觉不是很优雅  

* ``std::tuple<类型列表>`` 头文件是functional  
	- 赋予值的时候可以使用make_pair返回  
	- 取出值的时候可以使用``get<0>(tuple变量)``进行获取，注意模板里面是索引  
	- 语法很特殊，观感不便（数字来代表变量名，不知道用途，需要查看函数实现），pair类似  

* 使用struct简单的结构体解决返回多值问题  

## C++的模板template  
* 让编译器替你写代码  
* ``template<typename T>、template<class T>`` 模板类型，``template<int N>`` 指定传参？编译时将调用通过``<5>``替换为模板中使用的全部N类型的地方  
* 调用时调用编译器根据传入的类型推导形成代码  
* 隐式推导和指定类型  
* 不可过度滥用（给团队带来不便于理解）  

## C++堆与栈内存的比较  
* 栈内存分配空间，根据栈变量的大小，栈顶指针每次移动对应的大小，从高地址向低地址移动，这也是为什么栈分配非常快的原因（返回指针的地址即可-反向移动的原因（高->低）返回变量一开始存储的地址）  
* 栈变量出了作用域后生命周期结束，栈指针移动到原来的位置，内存全部弹出释放 ，释放和申请没有任何的开销  
* new->malloc，对应数量的内存分配给你，底层维护一个空闲列表（free list），跟踪内存块（大小，使用情况等）。每次申请时都需要便利寻找对应的内存块并且返回  
* 在需要生命周期长或者变量内存较大时选择堆上申请，其余尽量保持栈上使用内存  

## C++的宏  
* 利用预处理器在某种程度上帮我们实现自动化  
* 在vs界面下，项目的属性C++->预处理定义中可以设置在不同环境下-debug、release下的宏设置  
* 宏相关语句使用  
	- ``#define 待替换的目标语句 C++目标代码``  
	- ``#ifdef`` 检查目标宏是否定义  
	- ``#ifndef`` 检查目标宏是否没定义  
	- ``#if、#else、#elif`` 检查宏条件，条件年中可以使用``defind(宏)``检查宏是否定义   
	- ``#endif`` 结束宏条件语句  
* 利用\反斜杠可以写多行宏表示的代码 ，不是局限于一行，每一行的后面需要加上反斜杠即可  


## C++的auto关键字  
* 自动推导类型赋值  
* 对于自动推导类型（弱类型）和强类型的区别，自动推导类型有时会很方便（有很长的一段类类型表示），并且避免一些修改api之后的麻烦，但是也会在被改变时破坏原本代码的结构（比如返回值接受的是auto类型，由原本的string变为了``char*``，此时原本变量外部的size方法就无法使用）  
* 对于长类型的解决可以使用typedef、using取别名进行解决  
* c++11、14可以利用auto做出类似py的事情？  
```c++
	auto GetName() -> char*
	{
		return "Hello word";
	}
```

## C++的静态数组std::array  
* 所谓静态，也就是无法改变其数组长度  
* 接受两个模板参数：类型参数、常量参数  
* 和C语言风格的静态数组相比的好处：存在一些方法，能够使用std中的算法库，自身维护数组长度，使用栈上的内存（和std::vector不同）、存在边界检查  
* 在内存中的情况和C风格的数组一致（Size只是一个模板参数，传入的是常量，并不会在栈内存中进行存储），并且在debug调试模式下存在边界检查功能，是一个不错的选择  

## C++函数变量  
* C风格的函数指针->原始指针函数（而不是后续C++的function等特性）  
* ``返回类型 (*函数指针名)(函数参数);``  

### lambda-C++11  
* 定义匿名函数的方式，不需要通过函数定义，就可以定义一个函数的方法  
* 希望能够将一个函数传递给一个api，便于api中调用我们想要让其调用的方法  
* 语法：``[捕捉对象](参数列表) mutable{函数体}``  
	- mutable表示如果是值拷贝捕捉，那么可以修改这些拷贝后的对象，可省略  
	- 捕捉对象：=值拷贝，不可修改 &引用传递  

## C++为何不用using namespace std  
* 函数签名会重复或者重载，这是一件非常危险的事情，追踪寻找bug会非常困难  
* 绝对不能对std使用using namespace std  

## C++的命名空间  
* ``namespace``  
* 存在的原因：避免命名冲突  
* 指定namespace的别名：``namespace STL = std;``  

## C++的线程  
* 多个执行流的时候就需要线程来参与了  
* 使用：头文件：``<thread>``  
	- ``thread worker(函数对象);``  
	- 等待线程退出：``workder.join();``  
	- 在当前线程下阻塞：``using namespace std::literals::chrono_literals;std::this_thread::sleep_for(1s)``  


* 线程的目的就是为了优化  


## C++计时  
* 当我们需要知道程序运行的具体时间时，就需要对程序片段进行计时  
* C++11提供了**chrono**库，不需要去操作系统库  
	- ``std::chrono::high_resolution_clock::now();``  
		* 转换为正确的时间：``std::chrono::time_point_cast<std::chrono::microseconds>(对象)``  
			* ``time_since_epoch().count()`` 算自时间起始点到现在的时长，然后计数给出微秒计数  
	- ``std::chrono:duration<float> duration = end - start;`` 高精度计时  
		 - ``duration.count();`` 获取s的时间  
	- 获取的now似乎也可以：``std::chrono::duration_cast<std::chrono::microseconds>(endTimepoint - m_StartTimePoint).count()``即可  
	* 可以高精度计时以及适合全平台  

## C++多维数组  
* 多维数组的本质：数组里的数组里的数组.... 无线套娃即可  
* 处理数组的数组，容易造成内存泄露问题，并且new出来的还是存在内存碎片的问题（实际数组申请的内存并不会靠在一起），这样会导致cache miss(缓存不命中)，浪费时间影响性能  
* **可优化的重要事情之一：优化你的内存访问**  
* 多维数组就可以被优化为1维数组（连续存储，不会出现碎片问题，cache hit几率上升）  


## C++类型双关  
* 用来在C++中绕开类型系统  
```c++	
struct Verctor{
	int x, y;
	int* GetPositions()
	{
		return &x;
	}
};
int main()
{
	int a = 8;
	double b = *(double*)&a;
	Verctor vector = {4, 8};
	std::cout << vector.GetPositions()[0] << std::endl;
	return 0;
}
```

## C++的联合体  
* union-联合体、共用体  
* 一次只能占用一个成员的内存，可以添加函数等，不能使用虚方法。通常使用联合体是为了类型双关紧密相关的时候  
* 匿名union，不可添加函数  
```c++
	union{
		float a;
		int b;
	};
	a = 2.0f;
	std::cout << b << std::endl;
```

* 下面基本是基于类型双关的应用（同一个内存地址的不同解析），利用union可以更加方便的进行表示  
```c++
// C++联合体
#include <iostream>

struct Vector2
{
	float x, y;
};

struct Vector4
{
	//float x, y, z, w;

	//Vector2* GetA()
	//{
	//	return  (Vector2*)(&x);  // 类型双关
	//}

	//Vector2* GetB()
	//{
	//	return  (Vector2*)(&z);
	//}

	union
	{
		struct
		{
			float x, y, z, w;
		};

		struct
		{
			Vector2 a, b;
		};
	};

};


inline void PrintVector2(const Vector2& vector2d)
{
	std::cout << "(" << vector2d.x << ", " << vector2d.y << ")" << std::endl;
}

int main()
{
	Vector2 vector2 = { 1.2f, 3.4f };
	PrintVector2(vector2);
	Vector4 vector4 = { 1.0f, 2.0f, 3.0f, 4.0f };
	//PrintVector2(*vector4.GetA());
	//PrintVector2(*vector4.GetB());
	PrintVector2(vector4.a);
	PrintVector2(vector4.b);

	vector4.y = 2.5f;
	vector4.z = 6.8f;
	//PrintVector2(*vector4.GetA());
	//PrintVector2(*vector4.GetB());
	PrintVector2(vector4.a);
	PrintVector2(vector4.b);

	std::cin.get();
	return 0;
}
```

## C++析构虚函数  
* 虚函数和析构函数的结合，处理多态非常有用  
* 多态调用时，因为会将子类指针传递给父类指针，那么父类如果dele，则不会调用对应子类域的析构函数（要调用需要构成多态）  
* 多态方法：在基类析构函数前面加上virtual，子类析构函数()后面加上override(C++11)即可  


## C++类型转换  
* C++系统可用的类型系统中进行的类型转换，将内存对应的内容拿出来该如何解析的系统  
* 执行类型转换：隐式类型转换、显示类型转换（C风格(更简单，比如这个括号的风格)、C++风格）  
* C++的显示类型转换：  
	- ``static_cast<静态类型>(需要转换静态类型的变量)`` 静态类型转换,用于进行比较“自然”和低风险的转换，如整型和浮点型、字符型之间的互相转换,不能用于指针类型的强制转换  
	- ``reinterpret_cast<>()`` 将这段内存重新解释为别的东西,不同类之间的地址指针转换？用于进行各种不同类型的指针之间强制转换  
	- ``const_cast`` 移除或者添加变量的const限定  
	- ``dynamic_cast`` 不检查转换安全性，仅**运行时检查**，如果不能转换，返回null  

* 区分：并没有新增功能，只是增加了语法糖，并且做了额外的事情（比如转换不成功返回null）  
	- 可能收集到的编译时检查、帮助我们减少尝试强制转换时候犯下意外的错误-类型不兼容  
	- 做项目应该使用C++风格的类型转换，因为这样能够让代码更加的坚实  


## C++条件和操作断点  
* 在vs集成开发环境下，进行调试的时候更加实用的断点操作（特定条件下）  
* 条件断点（内存中某些东西满足了设定的条件，就触发此断点）、操作断点（一般是碰到断点时打印一些东西到控制台）  
* 面对一些我们已经编译了的程序，调试时也可以不需要重新编译、停止运行进行调试  
* VS上右键断点：操作、条件（也可以同时使用）
	- 操作：要打印的变量值放入``{}``中即可，或者使用一些关键字（其他随便写，最终是打印输入的全部内容）。同时可以设置程序是否继续运行还是中断暂停  

## C++的预编译头文件  
* 预编译的头文件：让你抓取一堆头文件，并将它们转换成编译器可以使用的格式，不必一遍又一遍地编译这些头文件（**只编译一次**）  
* 是为了避免一些常用的库，并不需要每次编译，只需要一次编译就行-避免频繁编译造成编译时间过长-浪费时间  、
* 一般使用于外部库（STL、操作系统接口），在pch文件中，此时每个cpp文件不需要去指定包含他们，会自动编译链接到  
* 需要注意：  
	1. 不要将频繁修改的文件放入预编译头文件中去（每次预编译头文件是一个，是一个整体，其中发生变化整个都需要重新编译）  
	2. 注意依赖关系问题。对于一些库包含在较少的CPP源文件下，对于include可能对代码阅读较为友好，因为全在pch文件中的话，不宜阅读  

* 在vs中使用：  
	- 一般命名为：``pch.h``  
	- 在pch文件下进行include包含其他常用不修改的头文件  
	- 创建一个对应包含include->pch.h的cpp文件，为pch.cpp,右键属性->c/c++->预编译头->创建预编译头。  
	- 项目属性下->C/C++->预编译头->使用预编译头、名字设置为对应的文件名``pch.h``  

* 在g++中使用：  
	- g++ -std=c++11 pch.h 即可，生成了h.gch文件  


## C++的dynamic_cast  
* 动态转换类型-更像是一个函数  
* 是专门用于沿继承层次结构进行的强制类型转换：基->派生、派生->基  。失败的转换会返回NULL。并且只用于**多态类类型**（基类存在一个虚函数表即可）  
* 如何做到检查的？  
	- 运行时类型信息（RTTI）：类型存储关于自己的信息  
	- 在vs中可以关闭，项目属性->C/C++->语言那里  
* 使用运行时类型检查，还可以做到检查一个对象是否是对应的实例方可继续的检查：  
```c++
	if (dynamic_cast<Player*>(player))
	{
		// ... 
	}
	
	// 或者下面这种方式比较常见：  
	Player* p = dynamic_cast<Player*>(player);
	if (p)
	{
		// ...
	}
```
* 动态检查确实会损耗一定的性能，如果想要避免这种情况需要进行关闭RTTI，并且不可使用此特性  


## C++的基准测试  
* 测试一个程序的运行速度-实际测量C++代码的性能  
* 利用作用（RAII）域-C++计时简单测试：<a href="#C++计时">链接</a>  
```c++
// 计时器结构
class Timer
{
public:
	Timer()
	{
		// 开始计时
	}
	
	~Timer()
	{
		stop();  // 调用方法？
	}
	
	void stop()
	{
		// 结束计时  
		// 处理计时数据？
	}
}
```

* 需要确保，无论测试什么，都需要知道测试的做了什么事情，比如编译器在debug、relese模式下对于代码的优化（汇编代码）。测试性能阶段一般实在relese模式下进行测试  
* 其他知识：  
	- ``__debugbreak();`` vs下的windows函数，中断编译，和设置断点类似  

## C++的结构化绑定C++17  
* 结构化绑定是一个新特性，让我们更好的处理多返回值的问题  
* ``std::tuple<std::string, int, ...>`` 元组和pair类似，只不过可以定义多个返回值，使用``{}`` 即可构造返回。详细请看: <a href="#C++处理多返回值">vcr</a>  
* 但是对于tuple取出确实非常麻烦，并且不便于代码阅读  
* 解决方法：  
	1. 模式化匹配？  ``string name; int age; std::tie(name, age) = tuple<std::string, int>的返回函数;``  
	2. 结构化绑定  ``auto[name, age] = tuple<std::string, int>的返回函数;``  
* 注意使用新特性确保编译器使用C++版本大于等于17.项目属性->C/C++->语言->语言标准  
* 在一些返回多个值的场景下又使用次数很少（定义struc类型），为啥不用这个新特性呢？  

## C++处理optional数据C++17  
* 对于一些函数处理后的状态的返回，比如文件打开函数判断文件是否打开等操作。传统的做法是对于返回值下手，或者一些输出型的参数  
* 对于返回值可以使用``std::optional<类型>``进行修饰，返回时如果存在对于类型列表中的值直接返回即可，不返回直接返回一个空列表``{}``即可。使用此函数接受时使用对应类型进行接收，可选的已经被设定  
* ``optional``相关方法  
	- ``has_value`` 为true表示可选的被设置。当然自身也可以直接当作表达式参与判断是否被设置中（bool运算符）  
	- 使用类似于指针，``->``使用，或者``*``进行解引用即可  
	- ``value_or(xxx)`` 如果其中数据存在返回数据，不存在返回xxx  

* 简单来讲，对于一个返回值，利用optional可以做到返回一个空的值，来表示本次函数的一种状态，接受使用和指针一样。可能存在或者不存在的值  

## C++中单一变量存放多种类型的数据C++17  
* ``std::variant<类型列表>`` 让我们不用担心处理数据的确切类型，列出可能的数据类型，决定成为什么类型  
* 访问此多类型变量：``std::get<类型>(多类型对象)``  
* 如果错误的访问了对应类型，会抛出``bad variant access``异常  
* ``data.index()``方法会返回当前多类型变量存储的实际类型索引（依据初始化时的类型列表的顺序）  
* ``std::get_if<类型>(多类型对象地址)`` 会返回一个指针，如果返回为空，则表明以当前类型取出对应值是不对的  
* 本质上是类型安全的union联合体吗？不是，联合体是以最大类型为内存，但是variant是将所有可能的数据类型存储为单独的变量做为单独的成员  

## C++如何存储任意类型的数据C++17  
* 头文件：``<any>``  
* ``std::any`` 用来表示任意类型  
* 取出比较麻烦：  
	- ``std::any_cast<类型>(变量名)``  
	- 如果不是想要转换的类型，这里将会抛出一个类型转换的异常  
* 大于一定内存大小会进行动态分配内存，不利于效率  
* 不好用，不如上面的  

## 如何让C++运行的更快  
* 一个项目通过多线程来提高性能  
* 对于一些独立的事情使用并行提高效率  
* 比如for循环串行执行的事情。（``C#``中存在并行for版本）  
* 头文件：``future``，``std::async(启动类型, 函数对象, 参数列表...)``  
	- 启动类型：``std::launch::async`` - 在一个单独的线程上完成（不设置这个虽然是异步，但不一定是多线程的）  
	- 注意访问临界资源的时候需要增加互斥锁，利用库中的RAII  （``std::lock_guard<std::mutex> lock(互斥锁对象)``）  
	- 返回值``std::future<返回值类型>``必须保留，否则会被摧毁（返回值类型为传入async中函数的返回值）  

* 注意：在多线程中，不可传递左值引用，因为线程函数按值移动或者复制，如果引用参数需要传递给线程函数，它必须被包装为（``std::ref, std::cref``）,或者通过指针传值？  
* 充分利用硬件来提高速度  

## 如何让C++字符串更块  
* 一堆字符串实际上会让程序变的很慢->string  
* string的性能问题：堆上分配、字符串格式化、字符串操作  
* 跟踪内存分配：``operator new(size_t size)``  
* 优化程序的时候，很多情况下只需要看看数据就能找到优化它的方法  
* **``string_view``**C++17中的新类，本质是一个指向现有内存的指针，也就是const char指针，也就是字符串内部存储的东西和大小。不在创建自己的字符串，而是在观察一个已有的字符串  
	- 函数的使用，传入原始字符串指针，传入自己想要观察的长度即可（可以直接输入长度，默认从0开始，否则 +x, len）  
	- 值传递即可  
* 完全优化，可以不使用string，因为有些时候存在的静态字符串没有利用让堆内存动态的分配空间导致性能下降  
	- 包括传参中的``const string&`` 这样即使你传递的是栈上的静态字符串，还是会被默认类型转换转换为string导致堆上分配从而减缓程序运行效率  

## C++的可视化基准测试  
* 使用谷歌chrome进行数据可视化分析：``chrome://tracing``  
	- 工作方式：加载一个包含所有数据的json文件  
* 创建``Instrumentor``类，格式化出一个json文件  
* 写入文件头，是chrome tracing需要的特定格式：  
```json
{"otherData": {},"traceEvents":[
```

* 写入文件尾：  
```json
]}
```

* WriteProfile - 单独写时间分析数据  
```json
{"cat":"function","dur":<持续时间>,"name":"<名字>","ph":"X","pid":0,"tid":线程id,"ts": <开始的时间-时间戳？>}
```


* 注意：  
	- 每次写文件的时候，为了防止程序崩溃不想丢失所有的数据，需要为ofstream.flush()刷新缓冲区到存储的文件空间内  
	- 持续时间的统计应该返回的是us结果  
	- 细节，如果name中存在诸如test"123" 需要将"替换为'  

* 可以使用宏定义包装InstrumentationTimer调用，将行号赋予进去，这样可以得到在当前代码中的唯一名字，让写入json文件中时name并不会发生冲突  
	- ``#define PROFILE_SCOPE(name) InstrumentationTimer timer##__LINE__(name)``  
		* ``##__LINE__``让Timer实例名在同一个作用域变得不一样，否则就命名冲突了-重定义  
	- 另外对于传入name也是一个非常烦人的想法，可以再次定义一个宏：``#define PROFILE_FUNCTION() PROFILE_SCOPE(__FUNCTION__)``  
		* ``__FUNCTION__``只是函数名称  
		* ``__FUNCSIG__`` 函数签名  

* C++使用下面的获取线程id  
	- ``std::hash<std::thread::id>{}(std::this_thread::get_id());``  

## C++的单例模式  
* 这是一种设计模式。当我们不打算实例化一个特定类的多个实例，我们想要他们自己的数据版本，这些操作将在这些数据上执行  
* 单例可以简单的理解为：``一个外表为类形式的命名空间``  
* 单例可以简单的从下面几条思路设计：  
	1. 饿汉模式。定义静态自身对象，在外定义  
	2. 懒汉模式。使用指针进行new，然后返回。``->需要注意线程安全``,特别注意双if语句也会造成未定义行为（同时访问临界资源）  
	3. 直接在对外访问接口的地方定义局部静态变量。c++11以上已经实现了静态变量的线程安全，所以借助特性在不需要加锁的情况下就能实现简单的单例模式  

## C++的小字符优化  
* 简称SSO  
* 不喜欢字符串的原因：内存分配问题->因为是在堆上导致速度很慢  
* C++标准库中，对于字符串在一定长度之内，会进行优化到在栈上分配内存，vs2019中，不超过15字符的字符串就可以得到优化  
* Debug模式下特殊编译，会导致内存分配  

## 跟踪内存分配的简单方法  
* 重载操作符new,return malloc 分配的字节返回``void*``指针  
* 然后可以在这个函数中插入断点，精确的追踪这些内存分配的来源  
* 重载操作符delete也可，在函数内部调用free，掉``void*``内存地址即可  
	- 当然参数还可以传入size_t size这样也可以获取即将被delete掉的内存有多大  

* 创建一个AllocationMetrics结构体，增加一个32位的无符号的整型TotalAllocated，表示总共分配的内存、TotalFreed总共释放的内存、写一个小函数：CurrentUsage，返回两者之间的差值  

## C++的左值与右值  
* 左值大多数在左边，右值在右边  
* 左值：有实际的存储空间  
* 右值：字面量、将亡值  
* 左值引用：int& 只接受左值，当然前面加上修饰符const即可以接受左值又可以接受右值  
* 右值引用：int&& 只接受右值  
* 右值引用的作用是非常强大的，可以优化很多事情，因为右边值是临时值，可以对他干任意的事情  

## C++的参数计算顺序  
* 比如函数传参的时候，参数的计算顺序（传递顺序如何？）  
* C++存在一些未定义行为，未定义行为的出现取决于编译器的不同而变化，这意味着不是可靠的  
* C++17增加了一项新规则：后缀表达式必须在别的表达式之前被计算，但是顺序仍然是没有确定的  

## C++的移动语义  
* 允许我们移动对象。这在C++11之前是不可能的，因为C++11开始引入了右值引用  
* 在一些传递临时值的场景下，实际new内存分配调用了多次，这是我们不想看到的，因为性能就浪费在了这种上面  
* 使用右值进行构造函数的时候，外部传入的必须是一个右值，使用右值引用进来其实会被识别为左值，需要使用``std::move()``或者(..&&)转换过来  

## std::move和移动赋值操作符  
* 左值需要转换为右值得时候，可以以标准库提供得优雅得方式:``std::move()``进行转换  
* 除了设置移动构造拷贝函数，还可以设置移动赋值函数``operator =(..&&) noexcept``  

## C++模板基础
* 高级语言 -> 机器语言 [编译]<编译器>  
* 汇编指令 -> 机器码 [汇编]<汇编器>
* 程序生成/运行过程:
> 源文件 -编译器-> 程序  编译期
> 程序 -执行-> 			运行期

* 参数 + 模板 -实例化-> 代码
* 流程位置
> 预处理-><模板实例化>->编译->链接->运行

* 使用模板的目的，可以让一个在运行期实时计算的东西放在编译前，通过模板的实例化算成(从运行时去计算变到了编译时去计算)不变值，从而节省运行时开销
* 资源前置，能提前算好的就在构建机上完成。达成效果：构建时间增加，运行时间减少
* 模板只能是编译期就能决定的事情
* 倾向于构建效率换取运行效率(前置优化)

## C++模板函数&模板全局常量
* 模板参数: 类型参数(typename/class) 非类型参数(具体类型, 比如 int N)
* ``template <>`` 模板的特化，模板参数具体的写出来
* 模板全局常量
```c++
	template<xxx>
	constexpr static xxx...
```

## C++模板元编程 - 模板类与类型萃取  
* 通过模板类进行类型萃取  
> 通过在模板类里进行using type, 针对特殊的类型可以单独特化  
> 后续使用的时候使用此工具模板类中的type->处理后的类型进行处理 
> 小例子: 编写个模板函数，在传参过程中, 对于基础类型(int double char)以及指针类型进行**值拷贝**, 对于自定义类型进行**const T&**    
```c++
// 例子: 实现一个模板函数, 对基础类型值拷贝, 对自定义类型引用传递

// 类型定义模板类
template <typename T>
struct ArgsType
{
    using type = const T&;
};
// 指针类型
template <typename T>
struct ArgsType<T*>
{
    using type = T*;
};
// int
template <>
struct ArgsType<int>
{
    using type = int;
};
// double
template <>
struct ArgsType<double>
{
    using type = double;
};
// char
template <>
struct ArgsType<char>
{
    using type = char;
};
// 模板类型定义
template <typename T>
using type_t = typename ArgsType<T>::type;

template <typename T>
void Func(type_t<T> arg)
{
    fmt::println("{}", boost::typeindex::type_id_with_cvr<type_t<T>>().pretty_name());
}
```

* 注意在模板类中引用其中的公开类属性或者公开类型时，需要使用到``typename``来标识此引用是类型还是属性(解决二义性问题)  
* 除开之前的全特化外，还有一种偏特化，偏特化并没有像全特化那样直接固定类型，而还是模板，只不过在之前的原始模板基础上，对类型的范围做了一定的限制。比如之前例子中的指针类型的偏特化  
* 使用模板类型定义将之前用于类型处理的模板类封装一下，方便需要对类型进行处理的相关方法使用
> ``template<typename T> using type_t = typename xxx<T>::type;``  

## C++模板元编程 - Substitution Failure Is Not An Error(SFINAE)
* 例子和上一个模块类似，但是现在不利用之前的偏特化进行解决
* 简单例子: 对于小于指针大小的类型采用值传递，其他采用引用传递
* **enable_if**
```c++
	template<bool Cond, typename T>
	struct enable_if {};

	template<typename T>
	struct enable_if<true, T> {
		using type = T;
	};
```

* C++模板匹配失败(Substitution Failure) 不会立刻结束，而是会去找其他的实现  
* C++模板会努力去匹配，直到所有都匹配失败
* 为了可读性，一般会为模板类中的类型定义一个模板类型(模板下直接using)，和一个模板的全局变量  

```c++
// 例子: 实现一个模板函数, 对基小于指针大小进行值拷贝, 否则进行引用
template <bool Cond, typename T>
using type_t = typename enable_if<Cond, T>::type;

template <typename T>
constexpr bool isLess_v = sizeof(T) <= sizeof(void*);

template <typename T>
constexpr bool isGreater_v = !isLess_v<T>;

template <typename T>
void Func(type_t<isLess_v<T>, T> t)
{
    fmt::println("值拷贝");
}

template <typename T>
void Func(const type_t<isGreater_v<T>, T>& t)
{
    fmt::println("引用传递");
}
```

## C++模板元编程 - 变参模板
* 一组相同类型的模板参数``**<typename... Args>**``
* 变参的展开方式:
  1. 直接展开``const T& t, Args... args`` -> 用逗号展开
  2. 嵌套展开``const T& t, const Args &... args`` -> 先嵌套, 后用逗号展开
  3. 按照自定义运算符展开 (折叠表达式)  
* 原理：在模板进行实例化的时候进行了递归  

## C++模板元编程 - 折叠表达式
* 之前的逗号展开，实际上相当于是一个一个传入进去依次匹配前面的。在自定义运算符展开的目的是定义一个二元运算符，让参数包内的每个参数使用此运算符计算从而达成展开的效果  
* 展开类型: 一元展开(展开过程中只使用了变参本身) 二元展开(使用了别的参数) ->只能使用二元运算符进行展开
* 对于使用二元运算符，存在两种展开方向(一元展开)  
  * 从左往右展开 -> ``(... - args)``
  * 从右往左展开 -> ``(args - ...)``
> 对于上述展开，对于 1 2 3 4变参的传入, 可以理解为如下:
> 从左往右展开： ((1 - 2) - 3) - 4 = -8
> 从右往左展开： 1 - (2 - (3 - 4)) = -2

* 二元展开, 实际上就是除了变参参数以外还引入了额外的参数，比如我连续将变参打印出来
  * ``(std::cout << ... << args) << std::endl;`` 


## C++模板元编程 - 类型限定与萃取
* 类型限定很重要，方便我们后续合法参数判断  
* C++20提供了一套参数限定(concept)  
* C++17以前需要使用SFINAE
* 例子: 要求函数传入的类型必须实现method()函数  
  * 思路：构造一个T类型的对象，调用对应的方法，可以正常调用就可以匹配了
  * 注意, 实在编译期(模板实例化期间)调用的  
  * 不能直接构造T对象(因为必须存在无参构造), 只需要将空指针转换为对应类型，然后调用方法就完事
    * ``decltype(static_cast<T*>(nullptr)->xxx)``
    * 标准库存在对应的方法 : ``std::declval<T>().xxx``  
    * 只能用作类型推导使用
    * 注意当同一个函数可以，但是类型返回不同时该如何去做？
      * 可以直接将任意类型映射为一种类型, 然后去比较此类型即可(一般类型设置为void即可) -> ``std::void_t<xxx>``  
      * std::is_void_v 可以直接将类型和void类型进行比较
    * 如果出现传入指针和引用，判断可能会存在误差，所以需要将类型擦除变成原本的类型(类型萃取一下即可->偏特化去匹配)
      * std::remove_reference_t<xxx>  

* 需要用到 ``std::enable_if_t -> typename std::enable_if::type`` 和 ``std::is_same_v<type, type>``  
  * 对于函数返回值类型 -> ``decltype(xxx)``  

* ``is_same_v`` 原理
  * 实际上很简单就是通过偏特化 判断两个模板参数是否一致，然后设置里面的常数变量为true or false即可

```c++
// 例子: 实现一个传参限制实现函数print的对象
template <typename T1, typename T2>
struct IsSame
{
    constexpr static bool value = false;
};

template <typename T>
struct IsSame<T, T>
{
    constexpr static bool value = true;
};

// std::is_same_v
template <typename T1, typename T2>
constexpr static bool IsSameV = IsSame<T1, T2>::value;

// std::is_void_v
template <typename T>
constexpr static bool IsVoidV = IsSame<T, void>::value;

// 将任意类型映射为void类型 std::void_t
template <typename T>
using VoidT = void;

// 类型擦除
template <typename T>
struct RemoveReference
{
    using type = T;
};
template <typename T>
struct RemoveReference<T&>
{
    using type = T;
};
template <typename T>
struct RemoveReference<const T&>
{
    using type = T;
};
template <typename T>
struct RemoveReference<T&&>
{
    using type = T;
};
template <typename T>
struct RemoveReference<T*>
{
    using type = T;
};
// std::remove_reference_t
template <typename T>
using RemoveReferenceT = typename RemoveReference<T>::type;

template <typename T>
void func(std::enable_if_t<IsVoidV<VoidT<decltype(std::declval<RemoveReferenceT<T>>().print())>>, T> t)
{
}
```