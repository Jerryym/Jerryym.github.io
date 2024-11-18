---
title: EchoEngine v0.2.0开发笔记-窗口设计
published: 2024-11-17
description: 'EchoEngine v0.2.0开发笔记-窗口设计'
image: ''
tags: [Render Engine]
category: 'EchoEngine'
draft: false 
lang: ''
---
本篇将记录EchoEngine中窗口相关开发内容，文中项目文件结构可参考下方项目仓库。

**项目仓库：**

::github{repo="Jerryym/EchoEngine"}

---

## 窗口设计

在EchoEngine开发过程中，当前我们选择了OpenGL作为图形API，以利用其跨平台的特性和强大的图形处理能力。为了更好地管理图形上下文和窗口管理等任务，我们引入了`GLFW`库。[GLFW](https://github.com/glfw/glfw)是一个开源的、跨平台的库，它提供了一个简单易用的接口来创建窗口、管理输入和处理事件，从而让我们能够专注于对于OpenGL的开发和使用。
:::note
除了GLFW，还有其他一些库也提供了类似的功能，适用于不同的平台和需求。例如，SDL(Simple DirectMedia Layer)是一个更为全面的库，它不仅支持窗口和图形上下文的管理，还提供了音频处理、文件I/O等额外的功能。对于Android平台来说，EGL(Embedded System Graphics Library)是处理图形上下文和窗口管理的标准库。而对于Apple平台，EAGL(Embedded Apple Graphics Library)则是一个专为iOS和macOS设计的图形库，它提供了对OpenGL ES的支持。
:::

### 窗口属性

在进行主窗口设计时，需要对窗口的部分基本要素进行设计，如：窗口大小、窗口标题、图标等。为此我们设计了名为`sWindowProp`的结构体，实现对于窗口属性的有效管理，代码如下。

```c++
/// @brief 窗口属性
typedef struct sWindowProp 
{
    unsigned int m_iWidth;  //窗口宽
    unsigned int m_iHeight;  //窗口高
    std::string m_sTitle;  //窗口标题

    sWindowProp(unsigned int iWidth = 1600, unsigned int iHeight = 900, const std::string& sTitle = "Echo Engine")
    : m_iWidth(iWidth), m_iHeight(iHeight), m_sTitle(sTitle)
    {}
}sWindowProp;
```

### 主窗口类设计

#### MainWindow类抽象

在EchoEngine中，因为希望未来能实现跨平台的能力（开坑+2...），所以将`MainWindow`设计为一个抽象类，这种设计不仅为引擎提供了一个统一的接口，而且确保了所有派生类都必须实现一系列核心功能，保证了跨平台的一致性和可扩展性， `MainWindow`类代码如下。

```c++
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
    static MainWindow* CreateMainWindow(const sWindowProp& prop = sWindowProp());

public:
    /// @brief 窗口更新
    virtual void OnUpdate() = 0;

    /// @brief 获取窗口宽度
    virtual unsigned int GetWidth() const = 0;
    /// @brief 获取窗口高度
    virtual unsigned int GetHeight() const = 0;
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
```

:::note
**垂直同步(VSync)**

* 定义
  1. 垂直同步（VSync）是一种显示技术，用于**同步显示器的刷新周期与GPU的渲染输出**，以避免**屏幕撕裂现象**。
  2. 屏幕撕裂：在图形渲染过程中，由于GPU渲染速度与显示器刷新率不同步，导致显示器在一次刷新周期内显示了两个或多个不同帧的部分图像，从而产生图像撕裂的效果。

* 作用
  1. 同步帧率：VSync确保GPU在显示器的垂直同步脉冲（VBlanking interval）期间完成帧的渲染，即在显示器开始新的一帧之前，GPU已经准备好了下一帧的图像数据。
  2. 消除撕裂：通过同步GPU的渲染输出和显示器的刷新，VSync可以有效地消除屏幕撕裂现象，提供更平滑和连贯的视觉体验。
  3. 减少视觉伪影：除了撕裂之外，不同步还可能导致其他视觉伪影，如重影（ghosting）和抖动（judder）。VSync有助于减少这些伪影。

:::

#### 面向Windows平台的MainWindow

* MainWindow_Windows.h

```c++
#pragma once
#include "GUI/MainWindow.h"

#include <GLFW/glfw3.h>

namespace Echo {

    class MainWindow_Windows : public MainWindow
    {
    public:
        MainWindow_Windows(const sWindowProp& sProp);
        virtual ~MainWindow_Windows();

    public:
        virtual void OnUpdate() override;

        virtual unsigned int GetWidth() const override { return m_sData.m_nWidth; }
        virtual unsigned int GetHeight() const override { return m_sData.m_nHeight; }
        virtual const std::string& GetTitle() const override { return m_sData.m_strTitle; }

        virtual void SetVSync(bool bEnable) override;
        virtual bool IsVSync() const override;

        virtual void* GetNativeWindow() const { return m_pGLFWWindow; }

    private:
        /// @brief 初始化窗口
        virtual void Initialize(const sWindowProp& props);
        /// @brief 窗口关闭
        virtual void ShutDown();

    private:
        /// @brief 窗口数据
        typedef struct sWindowData
        {
            unsigned int m_nWidth = 0;
            unsigned int m_nHeight = 0;
            std::string m_strTitle;
            bool m_bVSync = true;  //是否开启垂直同步
        }sWindowData;

        sWindowData m_sData;
        GLFWwindow* m_pGLFWWindow;
    };

}
```

* MainWindow_Windows.cpp

```c++
#include "echopch.h"
#include "MainWindow_Windows.h"

namespace Echo {

    static bool s_bGLFWInitialiazed = false;

    static void GLFWErrorCallback(int error, const char* description)
    {
        ECHO_CORE_ERROR("GLFW Error ({0}): {1}", error, description);
    }

    ///////////////////////////////////////////////////////////////////
    // MainWindow_Windows
    MainWindow* MainWindow::CreateMainWindow(const sWindowProp& prop)
    {
        return new MainWindow_Windows(prop);
    }

    MainWindow_Windows::MainWindow_Windows(const sWindowProp& sProp)
    {
        Initialize(sProp);
    }

    MainWindow_Windows::~MainWindow_Windows()
    {
        ShutDown();
    }

    void MainWindow_Windows::OnUpdate()
    {
        // 检查是否有事件触发
        glfwPollEvents();
        // 交换缓冲区
        glfwSwapBuffers(m_pGLFWWindow);
    }

    void MainWindow_Windows::SetVSync(bool bEnable)
    {
        if (bEnable == true)
        {
            glfwSwapInterval(1);
        }
        else
        {
            glfwSwapInterval(0);
        }
        m_sData.m_bVSync = bEnable;
    }

    bool MainWindow_Windows::IsVSync() const
    {
        return m_sData.m_bVSync;
    }

    void MainWindow_Windows::Initialize(const sWindowProp& props)
    {
        m_sData.m_nWidth = props.m_iWidth;
        m_sData.m_nHeight = props.m_iHeight;
        m_sData.m_strTitle = props.m_sTitle;

        ECHO_CORE_INFO("Create window {0} ({1}, {2})", props.m_sTitle, props.m_iWidth, props.m_iHeight);

        if (!s_bGLFWInitialiazed)
        {
            // glfwTerminate on system shutdown
            int success = glfwInit();
            ECHO_CORE_ASSERT(success, "Could not intialiaz GLFW!");

            glfwSetErrorCallback(GLFWErrorCallback);
            s_bGLFWInitialiazed = true;
        }
        m_pGLFWWindow = glfwCreateWindow((int)props.m_iWidth, (int)props.m_iHeight, props.m_sTitle.c_str(), nullptr, nullptr);
        glfwMakeContextCurrent(m_pGLFWWindow); //设置当前窗口上下文
        glfwSetWindowUserPointer(m_pGLFWWindow, &m_sData);
        SetVSync(true); //设置垂直同步
    }

    void MainWindow_Windows::ShutDown()
    {
        //释放GLFW资源
        glfwDestroyWindow(m_pGLFWWindow);
    }

}
```
