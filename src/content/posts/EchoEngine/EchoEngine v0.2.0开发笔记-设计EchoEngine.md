---
title: EchoEngine v0.2.0开发笔记-Design EchoEngine
published: 2024-07-10
tags: [Render Engine]
category: EchoEngine
draft: false
---
在Cherno的[DESIGNING our GAME ENGINE](https://youtu.be/etdSXlVjXss?si=Sn9zBWNlenxqmtaE)视频中，他介绍了一个渲染引擎/游戏引擎应该具有哪些功能，拥有哪些功能模块，这些在前一篇博客中我已经阐述过了。而对于Cherno自己的Hazel引擎来说，他的设计思路如下图所示。

![EchoEngine架构](EchoEngine软件设计.png)

从图中我们可以看出，Cherno的设计十分清晰，功能模块的任务分工明确。因此，我在设计EchoEngine时基本上参考了Cherno的设计思路，并在后续项目开发中，我会尝试融入自己的设计风格来完善项目。

##### **EchoEngine设计思路**

* 本项目将基于**ImGUI、GLFW和GLAD**开发
  * ImGUI：为EchoEngine提供GUI支持；并且由于ImGUI采用立即模式渲染方式，因此常常用于渲染引擎/游戏引擎开发。
  * GLFW：用于管理渲染引擎中**渲染上下文（Graphic Context）**
  * GLAD：提供现代OpenGL的API
* 将实现渲染引擎中Layer（层）机制
* 抽象渲染API，并首先实现支持OpenGL的API
* 添加**GLM库**作为数学运算库
* 使用**ENTT库**实现**ECS架构**作为数据管理方法
* 使用**box2d库**实现物理系统
* 接入脚本语言系统

##### **EchoEngine项目结构**

![EchoEngine项目结构](EchoEngine项目结构.png)

:::note
PS：前面花了三个多月验证基于Qt、GLFW和GLAD开发EchoEngine的技术路径，验证过程中出现了各种坑、各种奇葩的Bug……最后果断放弃并使用和Cherno同样的技术路径/(ㄒoㄒ)/~~

PPS：目标：未来实现Qt作为项目的GUI（开坑+1）
:::

## 参考

[1] [The Cherno Game Engine Series](https://www.youtube.com/playlist?list=PLlrATfBNZ98dC-V-N3m0Go4deliWHPFwT)

[2] 《Game Engine Architecture, Second Edition》 Jason Gregory
