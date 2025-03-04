---
title: EchoEngine开发笔记-Render Architecture
published: 2025-03-04
description: ''
image: ''
tags: [Render Engine]
category: 'EchoEngine'
draft: false 
lang: ''
---
## 近期开发进度

在最近的开发中，EchoEngine相关基础模块不断完善，如：主窗口(MainWindow)、事件系统(Event System)、层(Layer)。因此，在此先简要回顾一下之前关键模块的设计思想，并为接下来的渲染模块开发提供重要基础。

### 主窗口(MainWindow)

在EchoEngine的开发中，主窗口作为渲染现实喝用户交互的核心组件，并且为了实现跨平台支持以及灵活的窗口管理，我采用了抽象接口和平台相关实现相结合的设计模式。

* 定义了一个窗口属性结构体sWindowProp，用于描述窗口的基本属性

```cpp
/// @brief 窗口属性
typedef struct sWindowProp 
{
    uint32_t m_iWidth;  //窗口宽
    uint32_t m_iHeight;  //窗口高
    std::string m_sTitle; //窗口标题

    sWindowProp(const std::string& sTitle = "Echo Engine", uint32_t iWidth = 1600, uint32_t iHeight = 900)
        : m_iWidth(iWidth), m_iHeight(iHeight), m_sTitle(sTitle)
    {}

}sWindowProp;
```

* 设计了抽象类`MainWindow`，定义统一的窗口接口，包括窗口更新、获取窗口尺寸与标题、设置垂直同步（VSync）、以及事件回调等功能

```cpp
/// @brief 主窗口类，此类为抽象类，需要继承并实现其纯虚函数
class MainWindow 
{
public:
    /// @brief 事件回调函数
    using EventCallBackFun = std::function<void(Event&)>;

    virtual ~MainWindow() {}

    /// @brief 创建主窗口
    /// @param prop 窗口属性
    /// @return 
    static Scope<MainWindow> CreateMainWindow(const sWindowProp& prop = sWindowProp());

public:
    /// @brief 窗口更新
    virtual void OnUpdate() = 0;

    /// @brief 获取窗口宽度
    virtual uint32_t GetWidth() const = 0;
    /// @brief 获取窗口高度
    virtual uint32_t GetHeight() const = 0;
    /// @brief 获取窗口标题
    virtual const std::string& GetTitle() const = 0;

    /// @brief 设置垂直同步
    /// @param bEnable true-开启，false-关闭
    virtual void SetVSync(bool bEnable) = 0;
    /// @brief 获取垂直同步状态
    /// @return true-开启，false-关闭
    virtual bool IsVSync() const = 0;

    /// @brief 获取当前活动窗口
    /// @return 当前活动窗口指针
    virtual void* GetNativeWindow() const = 0;

    /// @brief 设置事件回调函数
    /// @param callback 
    virtual void SetEventCallBack(const EventCallBackFun& callback) = 0;
};

Scope<MainWindow> MainWindow::CreateMainWindow(const sWindowProp& prop)
{
#ifdef ECHO_PLATFORM_WINDOWS
    return CreateScope<MainWindow_Windows>(prop);
#else
    ECHO_CORE_ASSERT(false, "Unknown platform!");
    return nullptr;
#endif // ECHO_PLATFORM_WINDOWS
}
```

### 事件系统(Event System)

事件系统是EchoEngine实现响应用户输入和内部通信的关键模块。在EchoEngine中，我设计了三类事件类型：

* **应用程序事件**：用于处理应用程序相关的事件，如窗口大小变化、窗口关闭、App渲染、App更新等。
* **键盘事件**：用于处理键盘输入事件，如按键按下、按键松开等。
* **鼠标事件**：用于处理鼠标输入事件，如鼠标移动、鼠标点击等。

### 层(Layer)

层（Layer）的概念在EchoEngine中用于组织和管理渲染顺序，使得场景渲染更加高效。每个Layer作为独立的渲染单元，具有自己的渲染状态和资源管理策略。设计Layer系统的主要优势包括：

* **渲染顺序控制**：通过将不同的渲染元素分配到不同的层次，能够灵活控制绘制顺序和混合模式。
* **性能优化**：每个Layer可以单独优化渲染流程，提升整体性能，尤其在复杂场景中表现更为明显。
* **资源管理**：层与层之间相互独立，有助于更好地管理资源加载和释放，减少内存占用和重复绘制。

---

## Render Architecture-渲染架构

### 渲染是什么？

在开始渲染模块设计之前，先回答一个最基础的问题：**渲染是什么的？**
在我看来，渲染即是在屏幕上绘制出图像，并根据外部交互进行相应的渲染调整。

### 渲染架构设计

![Render Architecture](./Render%20Architecture/Render%20Architecture.png)

* 整个渲染架构分为两部分：渲染器(Renderer)和渲染API(Render API)
  * **渲染器(Renderer)**：与操作系统和图形API**无关**
  * **渲染API(Render API)**：与操作系统和图形API**相关**

#### 渲染器(Renderer)

在渲染器中，包含以下内容：

* 渲染器：2D渲染器、3D渲染器（用于实现前向渲染、延迟渲染等）
* 渲染管理
  * ⭐场景图：管理渲染场景中的所有渲染元素，如：模型、灯光、相机等
  * ⭐排序：基于Layer进行渲染元素的排序
  * 剔除：剔除掉不需要渲染的元素，减少绘制次数
* 材质
* LOD
* 动画
* ⭐相机系统
* 视觉系统：粒子系统、后期效果、反射等

#### 渲染API(Render API)

在渲染API中，包含以下内容：

* ⭐图形上下文(Graphics Context)：用于管理渲染相关的状态、资源和配置等信息
* 交换链：交换缓冲(Swap Buffer)
* ⭐缓冲对象：根据不同图形API封装顶点缓冲(VAO)、索引缓冲(IBO)、帧缓冲(FBO)
* 纹理和着色器：根据不同图形API封装纹理和着色器API
* 渲染状态管理
* 渲染管线
* 渲染通道
