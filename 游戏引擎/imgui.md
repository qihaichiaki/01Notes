## 编译文件选择
* 首先根据后端，选择对应的后端：比如SDL2/3、OpenGL... 比如SDL2: imgui_impl_sdl.cpp/h
* 其次是渲染器：可以使用SDLRender，比如：imgui_impl_sdlrenderer.cpp/h

## IMGUI和SDL2-SDLRenderer
* 后续语言环境中, 前面有`*`标识的表示必要的, 其余是可选的(大概? 持续测试中ing...)
### 1. 初始化
#### 初始化窗口和渲染器
- SDL创建窗口和渲染器后
- *``ImGui::CreateContext()`` 创建imgui上下文
- *``ImGui_ImplSDLRenderer2_Init(renderer);`` 初始化SDL渲染器后端
- *``ImGui_ImplSDL2_InitForSDLRenderer(window, renderer);`` 根据SDL的窗口对象和渲染器对象初始化, 用于将IMGUI与SDL的窗口和渲染器相互连接起来
#### IMGUI IO配置
- ``ImGui::GetIO()`` imgui 输入系统的相关配置
  - ``ConfigFlags`` 配置参数
    - ``ImGuiConfigFlags_DockingEnable`` 拖拽式停靠
    - ``ImGuiConfigFlags_ViewportsEnable`` 多视口
  - ``Fonts`` 字体相关
    - ``AddFontFromFileTTF(R"(C:\Windows\Fonts\msyh.ttc)", 18.0f, nullptr, ioImGui.Fonts->GetGlyphRangesChineseFull())``
      - 字体路径、字体大小、使用默认的字体配置参数、用于加载完整的中文字符集(确保中文正常显示)
#### IMGUI 界面配置
- ``ImGui::StyleColorsDark();`` imgui的界面风格(常见的暗色/浅色风格)
- ``ImGui::GetStyle()`` imgui界面外观样式的相关配置
  - ``WindowRounding`` 窗口边角的圆角半径
  - ``FrameBorderSize`` 设置控件(按钮、输入框)的边框粗细程度，0像素表示没有
  - ``FrameRounding`` 设置控件的圆角半径
  - ``Colors[ImGuiCol_WindowBg]`` 窗口背景颜色，输入值为ImVec4 - RGBA

### 2. 主循环
#### 事件处理
- 使用SDL处理所有事件``SDL_PollEvent()``
- *``ImGui_ImplSDL2_ProcessEvent`` 事件循环内交给IMGUI处理
- ``SDL_GetWindowFlags(window) & SDL_WINDOW_MINIMIZED`` 可以检测到最小化, 方便停止绘制等操作
#### 启用IMGUI新帧
```c++
ImGui_ImplSDLRenderer2_NewFrame();
ImGui_ImplSDL2_NewFrame();
ImGui::NewFrame();
```

- 使用上述代码后，才能开始绘制UI控件
#### 自定义内容
- ``ImGui::ShowDemoWindow(&show_demo_window);`` 显示官方demo窗口
#### 渲染内容
```c++
ImVec4 clear_color = ImVec4(0.45f, 0.55f, 0.60f, 1.00f);
ImGui::Render();  // 生成绘制数据
SDL_RenderSetScale(renderer, io.DisplayFramebufferScale.x, io.DisplayFramebufferScale.y); // 设置 SDL 渲染缩放比例（用于高 DPI 支持）。
SDL_SetRenderDrawColor(renderer, (Uint8)(clear_color.x * 255), (Uint8)(clear_color.y * 255), (Uint8)(clear_color.z * 255), (Uint8)(clear_color.w * 255));  // 使用设置好的 clear_color 清屏。 可以是0， 0， 255
SDL_RenderClear(renderer);
ImGui_ImplSDLRenderer2_RenderDrawData(ImGui::GetDrawData(), renderer);  // 调用 ImGui 的 SDL 渲染器进行绘制。
SDL_RenderPresent(renderer);  // 把最终结果呈现在窗口中。
```