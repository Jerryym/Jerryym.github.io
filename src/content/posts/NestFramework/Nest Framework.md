---
title: Nest Framework
published: 2024-01-28
category: Nest Framework
tags: [Qt, C++]
draft: false
---
当开发EchoEngine项目时，我的设计理念是采用**分层架构**，将渲染引擎层和应用软件框架分开，因此我创建了**Nest Framework**。**Nest Framework**是一个基于Qt 6.5.3版本的通用软件开发框架。该项目是一个学习项目，旨在深入了解Qt相关特性和软件开发过程。

## 开发环境

* **IDE**: Visual Studio 2019、Visual Studio 2022
* **Qt Version**: Qt6.5.3

---

## 开发日志(持续更新)

### Nest Framework v0.1.0

本框架计划使用**分层架构**设计，实现框架核心与UI分离。

* **Nest**：此项目为框架核心，是一个 `.lib`项目。内部包含Application(应用程序类)、基于[spdlog](https://github.com/gabime/spdlog)库封装的Log日志模块。
* **NestUI**：此项目为框架UI库，是一个 `.dll`项目。内部包含MainWindow(纯虚类)、DockWidget(纯虚类)、DockWidgetManager(DockWidget窗口管理类)。

---

## 依赖

本项目用到以下开源库：

* [Qt6](https://www.qt.io/)
* [spdlog](https://github.com/gabime/spdlog) - C++ logging library.
