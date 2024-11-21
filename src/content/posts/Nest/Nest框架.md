---
title: Nest框架
published: 2024-11-21
description: 'Nest框架简介'
image: ''
tags: [C++, Qt]
category: Nest
draft: false 
lang: ''
---

Nest是一个基于`Qt6.5.3`版本的通用软件开发框架。Nest将是一个长期维护项目，最终目标是实现一个可复用的Qt应用程序开发框架。

* **项目目标**：熟悉Qt相关特性、掌握CMake项目构建方法、提升软件架构设计思维。
* **项目仓库**：

::github{repo="Jerryym/Nest"}

---

### Environment

* **IDE**: Visual Studio 2022
* **Qt Version**: Qt6.5.3

### Modules

* NestApp：此项目为框架核心，是一个 `.dll`项目。内部包含 `Application`、基于[spdlog](https://github.com/gabime/spdlog)库封装的日志模块。
* NestUI：此项目为框架UI库，是一个 `.dll`项目。内部包含了第三方开源UI库[SARibbon](https://github.com/Jerryym/SARibbon)、 `MainWindow`、`DockWidget`、`DockWidgetManager`等。

### Dependencies

本项目用到以下开源库

* [Qt6](https://www.qt.io/)
* [spdlog](https://github.com/gabime/spdlog) - C++ logging library.
* [SARibbon](https://github.com/Jerryym/SARibbon) - Ribbon Control for Qt
