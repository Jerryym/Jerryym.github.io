---
title: EchoEngine开发笔记-GraphicsContext
published: 2025-03-05
description: ''
image: ''
tags: [Render Engine]
category: 'EchoEngine'
draft: false 
lang: ''
---
在[LearnOpenGL](https://learnopengl-cn.github.io/)中，有这么一段话是描述OpenGL的：
> OpenGL自身是一个巨大的**状态机(State Machine)**，一系列的变量描述OpenGL此刻应当如何运行。OpenGL的状态通常被称为**OpenGL上下文(Context)**。我们通常使用如下途径去更改OpenGL状态：设置选项，操作缓冲。最后，我们使用**当前OpenGL上下文**来渲染。
> 当使用OpenGL的时候，我们会遇到一些状态设置函数(State-changing Function)，这类函数将会**改变上下文**。以及状态使用函数(State-using Function)，这类函数会根据当前OpenGL的状态执行一些操作。

我学习OpenGL的时候，看到这段话我是完全没有理解，后来我在另一篇博客中看到一个形象的比喻，让我理解了什么是OpenGL上下文。
:::note
这就好比一个画家作图，OpenGL就是这个画家，而画家作画需要的画笔、画布等东西就是Context。Context的切换就像画家同时在作多幅画，当要到另一幅画前绘画时，画家需要放下原来的画笔，拿起这幅画所需的画笔。
:::

## 图形上下文

基于对于OpenGL上下文的理解，我对于图形上下文的理解如下：

* 图形上下文可以理解为图形渲染过程中的“工作空间”，即画家的画布和工具
* 图形上下文一般只能由创建它的线程使用
* 图形上下文会保存当前线程中所设置的所有渲染相关的状态、资源和配置信息
* 在一个图形上下文中，所有的渲染命令都是依据当前的上下文的状态和资源执行
* 图形上下文通常与一个窗口关联，如: 在GLFW中，通常会编写以下代码来创建一个窗口和图形上下文:

```cpp
/* Create a windowed mode window and its OpenGL context */
window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
if (!window)
{
    glfwTerminate();
    return -1;
}

/* Make the window's context current */
glfwMakeContextCurrent(window);
```

---

## EchoEngine中的GraphicsContext

在EchoEngine中，GraphicsContext类用于管理图形上下文，并根据不同的图形API，派生出不同的子类，如：OpenGLContext、VulkanContext等。由于在当前开发过程中，仅专注对于OpenGL的学习与应用，所以只派生创建了OpenGLContext子类。

### GraphicsContext

在GraphicsContext类中，定义了抽象的Init和SwapBuffers方法，分别用于初始化和交换缓冲区。

```cpp
/// @brief 渲染上下文：此类为抽象类，需根据渲染平台实现具体子类
class GraphicsContext
{
public:
    /// @brief 初始化
    virtual void Init() = 0;
    /// @brief 交换缓冲区
    virtual void SwapBuffers() = 0;
};
```

### OpenGLContext

* 通过其构造函数，传入当前创建的窗口句柄
* 在Init方法中，通过glfwMakeContextCurrent方法设置当前窗口上下文，并初始化Glad
* 在SwapBuffers方法中，通过glfwSwapBuffers方法交换缓冲区

```cpp
// OpenGLContext.h
/// @brief OpenGL上下文
class OpenGLContext : public GraphicsContext
{
public:
    OpenGLContext(GLFWwindow* windowHandle);
    virtual ~OpenGLContext() {}

public:
    virtual void Init() override;
    virtual void SwapBuffers() override;

private:
    GLFWwindow* m_pWindowHandle;
};

// OpenGLContext.cpp
OpenGLContext::OpenGLContext(GLFWwindow* windowHandle)
    : m_pWindowHandle(windowHandle)
{
    ECHO_CORE_ASSERT(windowhandle, "Window Handle is null!");
}

void OpenGLContext::Init()
{
    glfwMakeContextCurrent(m_pWindowHandle); //设置当前窗口上下文
    int status = gladLoadGLLoader(GLADloadproc(glfwGetProcAddress));
    ECHO_CORE_ASSERT(status, "Failed to initiazlize Glad!");

    ECHO_CORE_INFO("OpenGL Info: ");
    ECHO_CORE_INFO(" Vendor: {0}", reinterpret_cast<const char*>(glGetString(GL_VENDOR)));
    ECHO_CORE_INFO(" Renderer: {0}", reinterpret_cast<const char*>(glGetString(GL_RENDERER)));
    ECHO_CORE_INFO(" Version: {0}", reinterpret_cast<const char*>(glGetString(GL_VERSION)));
}

void OpenGLContext::SwapBuffers()
{
    glfwSwapBuffers(m_pWindowHandle);
}
```

---

## Reference

* [LearnOpenGL](https://learnopengl-cn.github.io/)
* [OpenGL Context（渲染上下文）及状态值查询](https://mozhi1729.blog.csdn.net/article/details/89315921?fromshare=blogdetail&sharetype=blogdetail&sharerId=89315921&sharerefer=PC&sharesource=Jerry_ym&sharefrom=from_link)
* [OpenGL 渲染上下文-context](https://www.cnblogs.com/xin-lover/p/9453760.html)
