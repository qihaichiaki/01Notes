## 前言
* SDL(Simple DirectMedia Layer) 跨平台的媒体层封装（音媒体、线程支持、输入设备）
* <a href="http://wiki.libsdl.org/SDL2/FrontPage">SDL2_wiki</a>
* 使用显卡->加速2D渲染纹理
* 可以在不同平台上完成代码

* SDL
* SDL_ttf 文本（.ttf - > ）
* SDL_image 图片
* SDL_mixer 音媒体
* SDL_gfx 简单图元的绘制

## EasyX->SDL
### 绘制区别
* SDL
  1. 可以使用BlitSurface或创建软渲染Renderer绘图（不考虑OpenGL等接入下，如果希望使用硬件加速的渲染，需要使用带有硬件加速标志(SDL_RENDERER_ACCELERATED)创建的Renderer进行绘图）
  2. 绘制流程：load->SDL_Surface->SDL_Texture(图片纹理-显存结构-硬件加速GPU)->Renderer渲染
  3. 文本类似，通过load生成文本的SDL_Surface->...

* EasyX
  1. 没有renderer的概念，绘图只需要源image和目标image（窗口缓冲区或者SetWorkingImage函数设置目标image）
  2. 绘制流程：loadimage -> putimage

## SDL
* 窗口和输入相关
### SDL_Init
* 传入flag，来初始化子系统
* SDL_INIT_EVERYTHING - 初始化全部的子系统
### SDL_MAIN_HANDLED
* 在引入SDL.h头文件前定义其宏，可以使用原本的main函数
### SDL_Quit

## 渲染
### 初始化-IMG_Init
* 传入flag进行格式的支持
* IMG_INIT_JPG | IMG_INIT_PNG
### 加载图像为surface IMG_Load
```c++
  SDL_Surface* surfImg = IMG_Load(path)
```
* SDL_Surface 为解码后的像素数据
* surface可以直接获取图像相关的数据

### 通过SDL_Surface* 创建纹理 - SDL_CreateTextureFromSurface
```c++
SDL_Texture* texture = SDL_CreateTextureFromSurface(SDL_Renderer*, SDL_Surface*);
```

* 从内存的图像数据，上传到gpu->纹理数据

### SDL_FreeSurface

### 直接加载纹理-IMG_LoadTexture
```c++
    SDL_Texture* texture = IMG_LoadTexture(SDL_Renderer*, const char*);
```

* SDL_image.h 中的``IMG_LoadTexture`` 一步到位将图片加载为纹理

### 释放纹理-SDL_DestroyTexture
```c++
    SDL_DestroyTexture(SDL_Texture*);
```

* 释放纹理资源

### 渲染纹理-SDL_RenderCopy
```c++
SDL_RenderCopy(SDL_Renderer*, SDL_Texture*, SDL_Rect* src, SDL_Rect* dst);
```

* 不想截取图片，src-> nullptr
* dst->nullptr --拉伸到整个绘制纹理上

### 渲染纹理-SDL_RenderCopyExF(可变换)
```c++
SDL_RenderCopyExF(SDL_Renderer*, SDL_Texture*, SDL_Rect* src, SDL_Rect* dst, double angle, SDL_FPoint* center, SDL_RendererFlip);
```

* angle 角度(逆时针) - 注意不是弧度
* center 旋转中心(纹理的坐标 - 旋转纹理的相对坐标系，左上角为00)
* SDL_RendererFlip 翻转效果

### 查询纹理宽高
* 将图片加载为SDL_Surface后直接访问其wh即可
* 对于显存结构纹理，使用``SDL_QueryTexture(SDL_Texture*, nullptr, nullptr, &w, &h)``进行获取
  * 前两个空指针也是输出参数：纹理格式、可访问性

### 创建程序窗口 - SDL_CreateWindow
```c++
SDL_Window* window = SDL_CreateWindow(title, x, y, w, h, flags)
```

* utf-8 窗口标题（在C++中可以指定u8"xxx"设置）
* x、y可以自己设置或者设置宏WINDOWS_CENTERED/UNDEFINED 中心或系统分配
* w、h宽高
* 标志存在全屏标志/后端标志位/窗口是否隐藏/窗口无边框/最大最小化/SDL_WINDOW_SHOWN - 显示即可
### SDL_DestroyWindow
### 创建渲染器 - SDL_CreateRenderer
```c++
SDL_Renderer* = SDL_CreateRenderer(window, index, flags);
```

* window - SDL_Window* 窗口
* index - 初始化的驱动索引(-1 通用的设置，自己选择)
* 标志位：SDL_RENDERER_ACCELERATED(硬件加速-GPU) SDL_RENDERER_SOFTWARE(软渲染-CPU) SDL_RENDERER_PRESENTVSYNC(垂直同步) SDL_RENDERER_TARGETTEXURE(可渲染到纹理)
### 设置渲染颜色-SDL_SetRenderDrawColor
```c++
SDL_SetRenderDrawColor(renderer, r, g, b, a);
```
### 使用绘图颜色清空渲染-SDL_RenderClear
```c++
SDL_RenderClear(renderer);
```
* 一般每次刷新前，使用``SDL_SetRenderDrawColor(renderer, 0, 0, 0, 255);SDL_RenderClear(renderer);`` 清屏然后绘制下一帧
### 绘制更新-SDL_RenderPresent
* 类似双缓冲，传入renderer参数
### SDL_DestoryRenderer
### IMG_Quit

## 事件
### 处理事件-SDL_PollEvent
### SDL_Event
* SDL_Event为事件结构体
#### 消息类型Type
* 各种消息事件
* ``SDL_QUIT`` 窗口退出

## 音频
### 初始化-Mix_Init
* 传入flag进行格式的支持
* MIX_INIT_MP3 | ...
### Mix_OpenAudio
* frequency 44100 - 音频采样率
* format MIX_DEFAULT_FORMAT(默认通用，和系统的字节序相关) - 解码的音频格式
* channels 声道数（立体声 - 2，单通道1）
* chunksize 2048 - 块大小(音频缓冲区大小 - 较小的缓冲区，减少音频的播放延迟但是会增加音频的处理负担)
### Mix_AllocateChannels
* 在SDL_Mixer中所有音频的播放都依赖``通道``，一个通道在同一时刻只能播放一个音频资源对象，但是一个音频资源可以同时在不同的通道中进行播放
* 申请通道个数(32)
### 加载音乐-Mix_LoadMUS
```c++
Mix_Music*  = Mix_LoadMUS(path)
```
### 加载音效-Mix_LoadWAV
### 播放音效-Mix_PlayChannel
```c++
    Mix_PlayChannel(int channel, Mix_Chunk*)
```

* channel 音效播放的通道, -1表示在所有通道中的第一个空闲的通道进行播放
  * 对于-1，如果通道被占满，那么最先开始播放的通道就会被停止，优先播放最新请求的资源

### 播放音乐-Mix_PlayMusic
```c++
Mix_PlayMusic(Mix_Music*, loop)
``` 

* loop表示循环多少次，-1表示永久循环

### 淡入播放-Mix_FadeinMusic
```c++
Mix_FadeinMusic(Mix_Music*, loop, ms)
```

* ms 淡入效果的毫秒数
### Mix_FreeMusic
### Mix_FreeChunk
### Mix_Quit


## 字体
### TTF_Init
* 初始化字体，不需要传flag
### 加载字体 - TTF_OpenFont
```c++
    TTF_Font* =  TTF_OpenFont(const char*, size);
```

* 根据对应字簇加载对应字号的字体
### TTF_RenderUTF8_Blended -> SDL_Surface*
```c++
TTF_RenderUTF8_Blended(TTF_Font*, text, color)
```
### 释放字体 - TTF_CloseFont
### TTF_Quit


## 基本图元 - SDL2_gfx
### filledCircleRGBA


## 其他方法
### SDL_ShowCursor
* SDL_DISABLE 窗口中默认的鼠标光标不可见
### SDL_GetPerformanceCounter
* 获得高性能计时器的计数，返回值为uint64
### SDL_GetPerformanceFrequency
* 获得频率，计数除以频率，可以得到对应多少s的时间
### SDL_Delay
* 类似于sleep - cpu时间片不会被占用

## 其他数据结构
### SDL_Point 2D坐标
### SDL_Rect 矩形数据结构
### SDL_Color 颜色数据