## EasyX->SDL
* SDL(Simple DirectMedia Layer) 跨平台的媒体层封装（音媒体、线程支持、输入设备）

### 绘制区别
* SDL
  1. 可以使用BlitSurface或创建软渲染Renderer绘图（不考虑OpenGL等接入下，如果希望使用硬件加速的渲染，需要使用带有硬件加速标志(SDL_RENDERER_ACCELERATED)创建的Renderer进行绘图）
  2. 绘制流程：load->SDL_Surface->SDL_Texture(图片纹理-显存结构-硬件加速GPU)->Renderer渲染
  3. 文本类似，通过load生成文本的SDL_Surface->...

* EasyX
  1. 没有renderer的概念，绘图只需要源image和目标image（窗口缓冲区或者SetWorkingImage函数设置目标image）
  2. 绘制流程：loadimage -> putimage


## 渲染
### 初始化-IMG_Init
### 加载纹理-IMG_LoadTexture
```c++
    SDL_Texture* texture = IMG_LoadTexture(SDL_Renderer*, const char*);
```

* SDL_image.h 中的``IMG_LoadTexture`` 一步到位将图片加载为纹理

### 释放纹理-SDL_DestroyTexture
```c++
    SDL_DestroyTexture(SDL_Texture*);
```

* 释放纹理资源

### 渲染纹理-SDL_RenderCopyExF
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

### SDL_CreateWindow
### SDL_DestroyWindow
### SDL_CreateRenderer
### SDL_RenderPresent
### SDL_DestoryRenderer
### IMG_Quit
### 通过SDL_Surface* 创建纹理 - SDL_CreateTextureFromSurface
### SDL_FreeSurface
### SDL_RenderCopy - 渲染
### SDL_FreeSurface

## 音频
### 初始化-Mix_Init
### Mix_OpenAudio
### Mix_AllocateChannels
* 在SDL_Mixer中所有音频的播放都依赖``通道``，一个通道在同一时刻只能播放一个音频资源对象，但是一个音频资源可以同时在不同的通道中进行播放
* 申请通道个数(32)
### 加载音乐-Mix_LoadMUS
### 加载音效-Mix_LoadWAV
### 播放音效-Mix_PlayChannel
```c++
    Mix_PlayChannel(int channel, )
```

* channel 音效播放的通道, -1表示在所有通道中的第一个空闲的通道进行播放
  * 对于-1，如果通道被占满，那么最先开始播放的通道就会被停止，优先播放最新请求的资源

### 播放音乐-Mix_PlayMusic
```c++
Mix_PlayMusic(Mix_Music*, -1)
``` 

* -1表示永久循环
### Mix_FreeMusic
### Mix_FreeChunk
### Mix_Quit


## 字体
### TTF_Init
### 加载字体 - TTF_OpenFont
```c++
    TTF_OpenFont(const char*, size);
```

* 根据对应字簇加载对应字号的字体
### TTF_RenderUTF8_Blended -> SDL_Surface*
### TTF_Quit


## 其他
### SDL_Init
### SDL_ShowCursor
* SDL_DISABLE 窗口中默认的鼠标光标不可见 
### SDL_PollEvent
* 处理消息事件
### SDL_Quit