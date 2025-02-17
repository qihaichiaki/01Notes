## 设计游戏引擎  
* name：Game Engine  
1. 入口点：启动应用时，会发生什么？控制了什么？  
2. 应用层  
	- 运行循环  
	- 渲染帧、事件  
	- ...  
3. 窗口层  
	- 输入、输出事件  
4. 渲染器  
5. 渲染接口抽象  
	- OpenGL  
	- DirectX  
6. Debug-日志系统  
7. 脚本语言：Lua、C#、..  
8. 内存系统 - 对性能是很大的问题  
9. ECS实体组件系统  
10. 物理系统  
11. File、I/O、VFS  
12. build System  

## 项目设置  
* 游戏引擎名：Hazel  
* 小技巧，在clone远程仓库的时候可以先clone到一个目录下，然后对目录中的内容拷贝到实际根目录下  
* 因为当前引擎需要链接很多依赖，并且引擎本身也要作为依赖提供给外部应用，所以选择只作为dll-Windows下的动态库，而其他依赖作为静态库链接到引擎动态库本身上  
* 属性设置  
	- 仅支持x64  
	- 设置当前为dll  
	- 输出目录：$(SolutionDir)bin\$(Configuration)-$(Platform)\$(ProjectName)\   
	- 中间目录：$(SolutionDir)bin-int\$(Configuration)-$(Platform)\$(ProjectName)\   

* 新建项目构建应用程序使用我们的引擎：Sandbox  
	- 同样支持x64  
	- 输出目录：$(SolutionDir)bin\$(Configuration)-$(Platform)\$(ProjectName)\   
	- 中间目录：$(SolutionDir)bin-int\$(Configuration)-$(Platform)\$(ProjectName)\   
	- 设置其为启动项目  
* 另外编辑sln项目，将其中Project的Sanbox设置到前面  
* 右键Sandbox、添加、引用：Hazel  
* 两个项目均创建src目录  
* 设置测试代码  
	- 动态库中函数声明前：``__declspec(dllexport)``  
	- 在应用中使用时的函数声明：``__declspec(dllimport)``  


## 入口点  
* dll也存在主函数，目前的想法是将主函数控制交给引擎engine那边处理  
* 引擎中添加Hazel，这将是引擎的核心库  
* 思路：  
	1. 解决动态库自身头文件声明export和对外声明import的情况，使用宏在dll和exe文件之间分开解决，并且目前限制在windows平台下  
	2. 想要在dll文件中实现main函数，在对外声明的头文件中进行处理，根据平台定义的宏，设置一个main函数，声明出CreateApp创建应用类似的东西，此接口在引擎库声明，实现却是在应用层，从而达到在引擎这边控制主函数的效果  
	3. 主函数这边对应用程序抽象出一个类，对其中的方法虚拟指定，子类需要重写实现  
* 细分：  
	1. Hazel/Core.h 预处理来适配dll  
		- HZ_PLATFORM_WINDOWS  引擎名-平台-win 表示windows1平台我们即将定义的宏  
		- HZ_BUILD_DLL 表示在当前dll解决项目中，接口前面需要声明``__declspec(dllexport)``  
		- 如果没有定义BUILD，说当前使用此头文件是应用程序，那么需要声明：``__declspec(dllimport)``  
		- 使用宏HAZEL_API 替换这些dll声明  
		- 可以使用``#error`` 适当说明一下错误  
		- 注意在不同的解决方案添加上述的宏  
	2. 在引擎层定义Application.cpp/h 定义类Application，虚拟化析构函数，实现一个函数run，之后引擎维持不退出的状态使用此函数。目前暴力实现死循环即可,目前流出一个接口CreateApplication方便用户层实现后通过此函数提供继承后的子类，这样在dll层使用此接口在main调用run即可  
	3. 在Hazel同级下创建Hazel.h 用来提供给应用程序调用相关接口的声明，注意在Sandbox添加头文件  
	4. 定义文件EntryPoint.h，在此内部实现main，通过声明CreateApplication达到调用run的目的  


## 添加日志  
* 采用第三方日志库作为我们引擎的日志依赖，因为需要支持格式化不同的类型，并且需要一种好的方式来格式化字符串，并加强格式化，写出一种好的C#风格  
* 使用第三方库<a href="https://github.com/gabime/spdlog">spdlog</a>，采样git clone子模块的形式，以便得到及时的更新  
	- git submodule add url Hazel/vender/spdlog  

* 添加第三方依赖库的头文件  
* 我们需要为他们制作一个包装器，我们需要一些东西，封装他们的接口变成我们自己引擎的接口  
* 目的：封装我们的接口为静态函数，提供一些应该提供的参数-基本就是搞包装器。并且设置宏更加方便我们的使用  

* 操作：  
	1. 在引擎模块(Hazel)下增加Log.cpp、Log.h用来包装三方库spdlog  
	2. 在Log.h中引用：spdlog/spdlog.h  
	3. 声明定义类Log，根据spdlog的使用进行封装  
		- spdlog使用的基本流程：1.首先创建多线程控制台来创建一个新的控制台，每个控制台用于输出日志，控制台基于的name就是每条消息开头的提示词。2.设置自己的格式选项3.根据日志级别输出内容，目前日志级别大致是：  
		- 思路：创建两个控制台，一个为引擎输出日志内容，一个为应用输出日志内容，分别是HAZEL、APP  
	4. 实现Log中的函数(下面全是静态成员函数、属性,类名实际作用就是起了一个命名空间的作用)  
		- void Init(); 初始化日志控制台，控制格式等操作。整个系统使用日志时，需要先调用此函数  
			* 参考格式：``set_pattern("%^[%T] %n: %v%$")`` 使用正确的颜色，时间戳、日志名、实际的日志消息  
			* 使用``stdout_color_mt(name)`` 返回智能指针对象  
			* 使用``set_level(spdlog::level::trace)``成员方法来设置日志级别  
		- 根据创建控制台返回的是一个共享指针对象，那么创建静态的这些属性，来分别定义Core-引擎日志、APP-应用日志  
			* ``std::shared_ptr<spdlog::logger> s_CoreLogger;``
			* ``std::shared_ptr<spdlog::logger> s_AppLogger;``
		- 创建对应的返回函数获取到他们  
	5. 使用宏包装一些不同级别方法的调用，使其更加好用  
		- 例子：``#define HZ_CORE_ERROR(...) ::Hazel::Log::GetCoreLogger()->error(__VA__ARGS__)``  
		- 参考的设置不同日志等级：fatal、error、warn、info、trace  
		- 注意不在引擎的命名空间内容  


## premake  
* 使用构建系统  
* 使用premake作为我们的构建系统，其语法简单，相较于CMake，Cmake更为复杂。其本身是用lua脚本语言编写  
* <a href="https://github.com/premake/premake-core">premake</a>  
* 将下载好的premake保存在项目目录下的vender/bin/premake 保存好许可证，即可使用  
* 在项目目录下新建premake5.lua，写配置文件：  
```lua
workspace "Hazel"  -- 解决方案  
	architecture "x64"  -- 架构
	
	configurations
	{
		"Debug",    -- 完全调试
		"Release",  -- 部分调试
		"Dist"		-- 发布版本
	}
	
outputdir = "%{cfg.buildcfg}-%{cfg.system}-%{cfg.architecture}"  -- debug/release-windows-x64

-- Hazel Engine配置
project "Hazel"  -- 项目  
	location "Hazel"  -- 相对于项目根目录，指定引擎库的位置
	kind "ShareLib"
	language "C++"
	
	targetdir ("bin/" .. outputdir .. "/%{prj.name}")  -- 二进制文件生成路径  
	objdir ("bin-int/" .. outputdir .. "/%{prj.name}")  -- 中间文件生成路径  
	files	-- 项目需要包含的文件
	{
		"%{prj.name}/src/**.h",    -- ** 为递归包含
		"%{prj.name}/src/**.cpp"
	}
	
	includedirs  -- 项目需要引入的头文件
	{
		"%{prj.name}/src",
		"%{prj.name}/vendor/spdlog/include"
	}
	
	filter "system:windows"  -- 使用过滤器来定义不同平台的规则宏  
		cppdialect "C++17"
		staticruntime "On"  -- 与链接运行时库相关，静态的链接他们  
		systemversion "latest"  -- 复制windows SDK Version  或者直接设置latest使用系统最新版本  
		
		defines  -- 宏定义
		{
			"HZ_PLATFORM_WINDOWS",
			"HZ_BUILD_DLL"
		}
		
		postbuildcommands  -- 指令操作，能方便将生成的dll文件放在应用程序目录下
		{
			("{COPY} %{cfg.buildtarget.relpath} ../bin/" .. outputdir .. "/Sandbox")
		}
		
	filter "configurations:Debug"  -- 适用于调试配置
		defines "HZ_DEBUG"
		symbols "On"  -- 调试符号打开

	filter "configurations:Release"  -- 适用于调试配置
		defines "HZ_RELEASE"
		optimize "On"  -- 优化符号打开

	filter "configurations:Dist"  -- 适用于调试配置
		defines "HZ_DIST"
		optimize "On"  -- 优化符号打开

-- Sandbox 配置
project "Sandbox"  -- 项目  
	location "Sandbox"  -- 相对于项目根目录，指定引擎库的位置
	kind "ConsoleApp"
	language "C++"
	
	targetdir ("bin/" .. outputdir .. "/%{prj.name}")  -- 二进制文件生成路径  
	objdir ("bin-int/" .. outputdir .. "/%{prj.name}")  -- 中间文件生成路径  
	files	-- 项目需要包含的文件
	{
		"%{prj.name}/src/**.h",    -- ** 为递归包含
		"%{prj.name}/src/**.cpp"
	}
	
	includedirs  -- 项目需要引入的头文件
	{
		"Hazel/vendor/spdlog/include",
		"Hazel/src"
	}
	
	links  -- 链接，内写项目名称即可  
	{
		"Hazel"
	}
	
	filter "system:windows"  -- 使用过滤器来定义不同平台的规则宏  
		cppdialect "C++17"
		staticruntime "On"  -- 与链接运行时库相关，静态的链接他们  
		systemversion "latest"  -- 复制windows SDK Version  
		
		defines  -- 宏定义
		{
			"HZ_PLATFORM_WINDOWS"
		}
		
	filter "configurations:Debug"  -- 适用于调试配置
		defines "HZ_DEBUG"
		symbols "On"  -- 调试符号打开

	filter "configurations:Release"  -- 适用于调试配置
		defines "HZ_RELEASE"
		optimize "On"  -- 优化符号打开

	filter "configurations:Dist"  -- 适用于调试配置
		defines "HZ_DIST"
		optimize "On"  -- 优化符号打开
```


* lua配置文件写完后，使用相对路径使用exe文件执行build构建脚本  
	- ``premake5.exe vs2022``  

* 可以编写GenerateProjects.bat优化在windows构建项目的体验：  
```bat
call vendor\bin\premake\premake5.exe vs2022
PAUSE
```

## 事件系统  
* 关闭窗口、调整窗口大小、输入事件：鼠标事件、键盘事件...  这些事件需要能够让引擎、应用系统知晓，并且能够去处理他们。背后的设计就是整个事件系统  
* 处理事件->窗口系统。这样更加合理，从底层到上层实现会更加的丝滑  
* 在Hazal引擎的应用程序表示层中：
	- Layers （最终会将事件传递到层）  
	- Application：事件的中心类  
		* 需要创建Hazel事件，方便Window返回对应类型事件  
			- 鼠标按下事件（x、y、按钮）  
			- ...
		* 为Window提供一个回调。每次创建一个window，都要提供一个回调，每次循环的时候检查回调，如果回调不为空，那么就回用对应的事件数据调用回调函数  
		* OnEvent：接受一些比如事件引用，回从Window调用这个函数  
		* 上面在抽象一层IEventListener的接口  
		* 另外除了窗口事件，Application还存在本地事件，比如：OnUpdate、OnTick、OnRender...  
	- Window：实际窗口的表示  
		* 一旦在窗口触发的事件发生，需要一种方式将它通信回Application  
		* 但是Window类不应该知道Application类的存在（不想将Application绑定到Window类）  
		* 创建一些方法，将所有这些事件发送回Application（App-create->window） 
		* 事件传递过程：窗口触发window回调-> 构造Hazel事件->Application  

* 操作：  
	1. 在Hazel目录下创建了名为Events的文件夹，其中包含：ApplicationEvent.h、Event.h、KeyEvent.h、MouseEvent.h  
	2. Event.h: 整个事件系统的主文件  
		- 包含头文件：Core.h、string、functional  
		- 设计思路：当前设计方式是阻塞事件，现在它并没有被缓冲，事件没有被延迟，只要被发生，整个应用就会停止，处理这个事件。未来可以维护一个事件队列，将它们缓冲到某种事件总线中  
		- **enum class EventType** 事件类型枚举类，用来描述事件类型：  
			1. None = 0,  
			2. 窗口事件：WindowClose(窗口关闭), WindowResize(窗口调整大小)、WindowFocus(窗口成为焦点), WindowLostFocus(窗口失去焦点), WindowMoved(窗口移动),  
			3. Application本地事件：AppTick, AppUpdate, AppRender,  
			4. 按键事件：KeyPressed(键按下), KeyReleased(键松开),  
			5. 鼠标事件：MouseButtonPressed(按下鼠标按钮), MouseButtonReleased(松开鼠标按钮), MouseMoved(移动鼠标)， MouseScrolled(鼠标滚动)  
		- **enum EventCategory** 枚举，将一个事件分成多个分类，使用BIT(Core.h中定义宏)运算(1 << x 左移一位)来创建位字段，利用此来查看一个事件实际上属于什么分类  
			1. None = 0,
			2. EventCategoryApplication = BIT(0), 应用类别(1)  
			3. ...Input = BIT(1), 输入类别(10)  
			4. ...Keyboard = BIT(2), 键盘类别(100)  
			5. ...Mouse = BIT(3), 鼠标类别(1000)  
			6. ...MouseButton = BIT(4), 鼠标按钮类别(10000)  
		- **宏定义实现override** 方便实现派生类需要重写的方法  
			1. EVENT_CLASS_TYPE(type) 重写实现返回事件类型、返回name、新增静态函数直接返回type(方便调度器判断的时候直接使用类型::的方式进行判断)  
				```static EventType GetStaticType() { return EventType::##type; } \  ```
				```virtual EventType GetEventType() const overried { return GetStaticType(); } \  ```
				```virtual const char* GetName() const overried { return #type; }```  
			2. EVENT_CLASS_CATEGORY(category)  重写实现获取事件分类类别标志，使用的是EventCategory枚举中的常量  
				virtual int GetCategoryFlags() const overried { return category; }  
		- **class Event** 事件的基类  
			1. protected保护对象：  
				- m_Handled = false;  需要查看一个事件是否被处理了  
			2. public：  
				- 纯虚函数：  
					1. ``EventType GetEventType() const``  返回事件类型  
					2. ``const char* GetName() const`` 返回name  
					3. ``int GetCategoryFlags() const``  返回事件类型标志？  
				- 虚函数：  
					1. ``std::string ToString() const `` 这里可直接实现，返回GetName()即可，只是一个调试的东西，后续如果需要打印其他东西，可以重写实现即可。调试的东西暂时不会考虑性能因素  
				- inline bool IsInCategory(EventCategory category) 工具函数，通过给定的枚举事件分类，通过&与算符检查事件是否在这个分类里的方法。可以利用这个快速过滤掉某些事件  
					* return GetCategoryFlags() & category;  
			3. friend 友元类：  
				- class EventDispatcher; 事件调度器类，下面会进行实现  
		- **class EventDispatcher** 事件调度器类  
			1. using:  
				* using EventFn = std::function<bool(T&)>;  T应该为Event的派生类类型  
			2. public:  
				* 构造函数，传递Event& event, m_Event接收  
				* ``bool Dispatch(EventFn<T> func)`` 调度  
					检查当前事件类型是否等于T::GetStaticType，如果是，则``mEvent.m_Handled = func(*(T*)&m_Event); return true;``否则return false;  
			3. private:  
				* Event& m_Event;  
		- **operator<<** 实现将Event e 打印出来的函数，寄ostream << e.ToString(); 即可  
	3. ApplicationEvent.h： application本地系统的事件  
		- 包含窗口操作和app循环机制的事件  
		- ``class WindowResizeEvent`` 窗口调整大小类  
			* 初始化unsigned int width、height，存在Get方法  
			* 重写ToString显示调整大小的宽度和高度  
			* 重写宏：WindowResize、Application  
		- ``class WindowCloseEvent`` 窗口关闭事件类  
			* 构造函数  
			* 重写宏：WindowCose、Application  
		- ``class AppTickEvent``  
            * 构造函数  
			* 重写宏：AppTick、Application  
		- ``class AppUpdateEvent``  
		- ``class AppRenderEvent``  
			* 类似  
		
	4. KeyEvent.h: 键盘事件  
		- 包含头文件：Event.h sstream  
		- 包含在命名空间Hazel内  
		- 思路分析：首先按键事件之间的按键代码是通用的，所以存在基类KeyEvent，其次，因为按键的特性，一直按下一个按键，首先第一次会立即触发，暂停一会儿后就是连续的触发，所以可以设计为第一个为KeyPressedEvent(按键按下事件), 其他基本上就是KeyRepeatEvent(按键重复事件)   
		- **class KeyEvent : public Event**  
			1. public：  
				- inline int GetKeyCode() const { return m_KeyCode; }  
				- EVENT_CLASS_CATEGORY(EventCategoryKeyboard | EventCategoryInput)宏  
			2. protected:  
				- KeyEvent(int keyCode) : m_KeyCode(keycode) {}   受保护的构造函数，外部对象无法对其构造  
				- int m_KeyCode; 按键代码  
		- **class KeyPressedEvent : public KeyEvent**  
			1. public:  
				- 构造函数：传入int 按键代码(keycode)和统计次数(repeatCount)？  
				- inline int GetRepeatCount 返回按键按下次数  
				- 重写ToString函数，使用stringstream，返回键盘按下事件的代码和repeats重复次数  
				- EVENT_CLASS_TYPE(keyPressed) 宏  
			2. private:  
				- int m_RepeatCount;  按键按下次数  
		- **class KeyReleasedEvent : public KeyEvent**  
			* 构造函数传入keycode到基类初始化、ToString重写一下说明释放的keyCode，利用宏EVENT_CLASS_TYPE指定一下事件分类为KeyReleased  
	5. MouseEvent.h: 鼠标事件  
		- ``class MouseMovedEvent``  鼠标移动事件类  
			* 构造函数传入x、y -> MouseX、MouseY定位鼠标的位置  
			* Get函数返回x、y的值。类型为float  
			* 重载实现ToString返回当前鼠标移动事件的x、y位置  
			* 重写虚函数的两个宏，传入type(MouseMoved)和事件分类类型(Mouse | Input)  
		- ``class MouseScrolledEvent`` 鼠标滚动事件类  
            * 构造函数传入xOffset、yOffset -> XOffset、YOffset滚动了x、y  
			* Get函数返回xOffset、yOffset的值。类型为float  
			* 重载实现ToString返回当前鼠标滚动事件的偏移值  
			* 重写虚函数的两个宏，传入type(MouseScrolled)和事件分类类型(Mouse | Input)   
		- ``class MouseButtonEvent`` 鼠标按钮事件基类（类似于按键事件基类）  
            * GetMouseButton() 返回鼠标的按键，m_Button,int类型  
			* 重写虚函数的一个宏：事件分类类型(Mouse | Input)  
			* 保护属性中增加属性和构造方法  
		- ``class MouseButtonPressedEvent`` 鼠标按键按下类  
			* 构造函数传入按下按钮button表示int  
			* ToString打印输出鼠标按下事件 button  
			* 重写虚函数宏：传入此时鼠标按下按钮事件类型  
		- ``class MouseButtonReleasedEvent`` 鼠标按键松开类  
			* 同理如上  
	* 注意：一些C++标准库的文件，这些应该首先出现在预编译头文件中。引擎越往下发展，越难集成到预编译头文件中，目前方便使用还未集成到预编译头文件中  
	* 其他修改：在premake中增加了一些头文件路径和sdk系统版本使用最新的设置。最后需要在log.h中增加头文件``spdlog/fmt/ostr.h``让日志系统支持输出流操作，更加的支持自定义类型的输出  


## 添加预编译头  
* 在src下创建hzpch.h 和hzpch.cpp  
* .h 下包括一切不需要重复每次编译的头文件  
	- iostream、memory、utility、algorithm、functional  
	- string、sstream、vector、unordered_map、unordered_set  
	- 判断当前是否在WINDOWS下：添加Windows.h  
* 将其添加到premake中：  
	- 在project Hazel中，objdir下面：  ``pchheader "hzpch.h" 指定pch头文件``，windows上需要加（不需要添加到过滤器中，其他平台会自动忽略）``pchsource "Hazel/src/hzpch.cpp"``  

* 每个cpp第一个头文件需要包含它  


## 窗口抽象和GLFW  
* 使用GLFW第三方库创建窗口，并且能够做到跨平台  
* 目前阶段只是支持OpenGL，但是日后会支持DirectX、Valkan等图形渲染，所以对每个平台抽象一份Window类  
* 首先可fork将GLFW为子模块放在自己的github上，随后由主项目添加子模块添加进去即可  
	- 拷贝GLFW的master分支，然后添加premake5.lua  
	- premake5.lua中添加的是Windows的专用文件  
* git submodule add url  添加创建好的glfw的url  Hazel/vendor/GLFW  

* 在主的premake5.lua 做如下的修改：  
	- 在outputdir下创建了一个IncludeDir = {} 创建了一个表，IncludeDir["GLFW"] = "Hazel/vendor/GLFW/include",include "Hazel/vendor/GLFW" -包含我们的premake文件，类似CPPinclude的包含  
	- Hazel项目中includedirs中，需要包含新的include："%{IncludeDir.GLFW}"  
	- links: "GLFW", "opengl32.lib"  

* 目前能够将GLFW项目成功编译构建  
* 操作：  
	- 在Hazel目录下创建Window.h,这里存放的是Window的抽象表示，和平台无关，也是Application中所要使用的  
		1. ``using EventCallbackFn = std::function<void(Event&)>``  
		2. 虚函数-析构函数  
		3. 纯虚函数-void OnUpdate  
		4. 纯虚函数-unsigned int GetWidth、GetHeight  
		5. 纯虚函数-void SetEventCallback(const EventCallbackFn& callback)  
		6. 纯虚函数-void SetVSync(bool enabled)  
		7. 纯虚函数-bool IsVSync()  
		8. ``static Window* Create(const WindowProps& props WindowProps());``  
	- struct WindowProps 基本的窗口属性  
		* 标题、Width、Height。标题默认为Hazel Engine，宽度默认为1280，高度默认为720  
	- class Window类，纯虚函数，只是声明接口相关，要求派生类如何实现他们  
	- 在Hazel同级目录创建了Platform/Windows/ 创建了windows平台下具体实现window的相关文件：WindowsWindow.h、cpp  
	- WindowsWindow类继承Window类，override一些虚函数..  
		* 头文件包含：``GLFW/glfw3.h``  
		* public:  
			1. WindowsWindow(const WindowProps& props);  
				- Init(props)  
			2. virtual ~WindowsWindow();  
				- Shutdown();  
			3. void OnUpdate() override;  会更新GLFW、缓冲区会轮询输入事件...  
				- glfwPollEvents();
				- glfwSwapBuffers(m_Window)  
			4. inline GetWidth、GetHeight - 直接实现即可  
			5. inline void SetEventCallback(const EventCallbackFn& callback) override 直接实现即可：m_Date.EventCallback = callback  
			6. void SetVSync(bool enabled) override;  
				- 如果为true：glfwSwapInterval(1)，否则为glfwSwapInterval(0), 同时设置m_data中的VSync  
			7. bool IsVSync() const override;  
				- return Data中的VSync即可  
		* private:  
			- virtual void Init(const WindowProps& props);  
				* 初始化Data、打一条内核日志显示属性  
				* 判断s_GLFWInitialized是否为false，随后调用int success = glfwInit(); 初始化函数，然后内核断言其是否初始化成功，设置s_GLFWInitialized为true即可  
				* 创建窗口：m_Window = glfwCreateWindow(属性... nullptr, nullptr)、glfwMakeContextCurrent(m_Window)、glfwSetWindowUserPointer(m_Window, &m_Data)、SetVSync(true)  
			- virtual void Shutdown();  
				* glfwDestroyWindow(m_Window);  
		* private:  
			- ``GLFWwindow* m_Window;``  
			- struct WindowData结构，需要将其传递给GLFW  
				- string Title  
				- unsigned int Width、Height  
				- bool VSync;  
				- EventCallbackFn EventCallback;
			- WindowData m_Data ;  
		* 实现中首行增加了static bool s_GLFWInitialized = false(防止出现多次调用glfwInit); Create直接返回new WindowsWindow对象即可，传入props  
	- 在Core.h 文件中，定义断言宏  
	> ``#ifdef HZ_ENABLE_ASSERTS``
	>   ``#define HZ_ASSERT(x, ...) { if(!x) { HZ_ERROR("Assertion Faild: {0}", __VA_ARGS__); __debugBreak();}}``  
	>   ``#define HZ_CORE_ASSERT(x, ...) { if(!x) { HZ_CORE_ERROR("Assertion Faild: {0}", __VA_ARGS__); __debugBreak();}}``
	> ``#else``
	>   ``#define HZ_ASSERT(x, ...)``  
	>   ``#define HZ_CORE_ASSERT(x, ...)``    
	
	- 在Application中 下：  
		* Application类新增unique_ptr唯一指针Window m_Window  和m_Running控制循环  
		* 实现中在构造函数中使用unique_ptr Window(Window::Create());进行创建  
		* 在Run的循环中：运行Window的OnUpdate即可  
		* 可以试着在前面增加opengl相关调用：``glClearColor(1, 0, 1, 1);glClear(GL_COLOR_BUFFER_BIT);``  
* pch文件改动：增加#include "Hazel/Log.h"  


## 窗口事件  
* 将事件添加到窗口中，窗口就会生成事件  
* 为了实现在Application端只需要设置事件回调，让window感知不到Application的情况下完成事件的传递，也就是通过SetEventCallback进行实现  
* 在Application添加唯一指针保存Window对象，并且设置回调为自身的OnEvent函数（利用绑定）  
* WindowsWindow中，需要利用GLFW也是设置回调来获取窗口相关的事件：  
	- glfwGetWindowUserPointer将用户存入的数据取出  
	- glfwSetWindowSizeCallback 改变窗口大小时触发  
	- glfwSetWindowCloseCallback 关闭窗口时触发  
	- glfwSetKeyCallback 按键操作时，接受四个额外的参数：key(按键代码)、scancode、action(GLFW_PRESS - 按键按下、GLFW_RELEASE- 按键松开、GLFW_REPEAT - 按键重复)、mods    
	- glfwSetMouseButtonCallback 鼠标按钮触发 button、action（没有重复事件）、mods  
	- glfwSetScrollCallback 鼠标滚动触发  
	- glfwSetCursorPosCallback 鼠标移动触发  
	- 额外的：在init的时候最后新增glfwSetErrorCallback（设置一个类外的静态函数 GLFWError int、字符指针 ），用来接受错误消息  

## Layers  
* 层对于游戏引擎来说：1.决定渲染的顺序。2.根据渲染的顺序->决定接收事件的层数是第几个  
* 在Hazel游戏目录下新增：Layer.h 和 Layer.cpp  
* 对于类Layer:  
	- public:  
	- 构造函数，给予一个name - m_DebugName(保护);  
	- 虚函数 析构函数
	- 虚函数 OnAttach() {}  
	- 虚函数 OnDetach() {}  
	- 虚函数 OnUpdate() {}  
	- 虚函数 OnEvent(Event& event) {}  
	- inline const string& GetName const {...}  
	* CPP中的实现也只是对构造和析构进行了实现  
* 新增文件：LayerStack.h/cpp  
* 对于类LayerStack  
	- public：  
	- 构造、析构  
		* Insert = m_Layers.begin();  
		* 遍历m_Layers,对每一个Layer进行delete  
	- ``PushLayer(Layer* layer);``  
		* Insert = m_layers.emplace(Insert, layer);  
	- ``PushOverlay(Layer* overlay);``  
		* m_layers.emplace_back(overlay);  
	- ``PopLayer(Layer* layer);``  
		* find找到对应的it迭代器，判断是否存在，随后erase，并且Insert--  
	- ``PopOverLay(Layer* overlay);``  
		* find找到判断，然后erase(it)即可  
	- ``std::vector<Layer*>::iterator begin() { return m_Layers.begin(); }``  
	- 类似，返回end  
	- private:  
	- ``vector<Layer*> m_Layers;``  
	- ``vector<Layer*>::iterator m_LayerInsert;``  

* Application.h/cpp 文件的变动：  
	- ``void PushLayer(Layer* layer);``  
		* Push  
	- ``void PushOverlay(layer* overlay);``  
		* PushOverlay  
	- private:  
	- ``LayerStack m_LayerStack;``  
	- cpp中OnEvent函数变化：  
		* 遍历LayerStack的迭代器，从后往前进行迭代，调用每个迭代器的OnEvent函数，判断传入事件是否被掉过了，如果被调用则break  
	- cpp中Run函数变化：  
		* 从LayerStack遍历每个Layer，调用其OnUpdate，最后在调用window的Update  
* Hazel.h 中包含：Layer.h 文件  
* 在premake.lua文件中：  
	- 构建选项设置包含了多线程调试dll、多线程dll  
	- buildoptions "/MDd" - debug 其余不加d  

## 现代OpenGL和Glad  
* 现在迫切的可以实现IMGUI实时GUI程序在我们的程序中跑出来  
* 目前需要做的是将所有现代OpenGL函数从GPU加载到C++的方法-使用GLAD（基于官方规范的多语言gl加载生成器）  
* <a href="https://glad.dav1d.de">网址</a>  C/C++ OpenGL Core  gl=4.6 然后生成，下载glad.zip即可  
	- 放在Hazel/vendor/Glad  

* premake5.lua 中：  
	- include包含在对应Glad项目下的premake5.lua  
	- 上面的IncludeDir增加对应头文件的索引（下面includedirs需要进行包含）  
	- links链接上Glad  
* 在Glad下设置premake5.lua 将其设置为静态库  
	- 和GLFW类似  
	- files中包含全部头文件和对应的源文件  
	- 包含win的删除，构建选项删除即可  

* 构建好项目后需要对其进行加载：  
	- 在WindowsWindow.cpp中：包含glad/glad.h  
		* 在init函数中，初始化好OpengGL上下文后，使用：``int status = gladLoadGLLoader((GLADloadproc)glfwGetProcAddress); HZ_CORE_ASSERT(status, "Failed to initialize Glad!")``  
	- 在premake5.lua中，对windows筛选中定义宏：``GLFW_INCLUDE_NONE`` 不让GLFW生成opengl相关，使用glad/glad.h来生成  

## 引入ImGui  
* 先引入ImGui，方便后续实现渲染器的时候可以更加的方便调试  
* 首先需要将ImGui fork到本地，增加premake5.lua 文件  
* ``git submodule add url 路径``  
* 增加premake5.lua文件，将imgui编译为对应的静态库  
* 静态库引入项目后，开始添加imgui层，开始调试代码：  
	- 在引擎的文件夹下创建ImGui文件夹，并且创建对应的两个文件：ImGuiLayer：.h 和 .cpp  
	- 创建类ImGuiLayer，继承Layer  
	- cpp 文件包含imgui.h 和Platform中的Renderer.h  
		* 构造、析构  
			- 构造给基类赋予一个ImGuiLayer  
		* OnAttach函数重写  
			- ImGui::CreateContext();  
			- 
		* OnDetach函数重写  
		* OnUpdate函数重写  
		* OnEvent函数重写  
	- 在Platform文件夹下创建OpenGL，然后将imgui-opengl3的实现和头文件拷贝于此  
		* 重命名：ImGuiOpenGLRenderer.h/cpp  
		* 注意CPP需要包含预编译头  

