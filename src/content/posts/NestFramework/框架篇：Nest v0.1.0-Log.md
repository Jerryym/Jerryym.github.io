---
title: 框架篇：Nest v0.1.0-Log
published: 2024-04-05
category: Nest Framework
tags: [Qt, C++]
draft: false
---
根据{% post_link EchoEngine开发笔记-软件设计 %}中的设计思路，在完成 `Applicaiton`类的初步设计后，我们将进行Log日志模块的设计与开发。

### 设计思路

- **问题引入**
  说起Log，我们在软件开发过程中，经常会遇到需要跟踪和记录应用程序的状态和行为的情况。当应用程序变得复杂，出现了更多的错误、警告和信息时，手动输出到控制台或文件就不够高效和方便了。因此，有一个可靠的日志系统是非常重要的。
- **功能设计**
  对于Log来说，我们希望其具有以下功能和特性：

  - 支持多种日志级别，如调试、信息、警告、错误等

    > 原文链接：[C++ 实现简易 log 日志库](https://blog.csdn.net/K346K346/article/details/46908167)
    > 日志级别一般分为以下几种（从高到低排序）：
    > **FATAL（致命）**： FATAL级别的日志用于记录严重的错误，通常表示应用程序遇到了无法继续运行的情况，可能会导致应用程序崩溃。这些日志级别用于标识需要立即处理的严重问题。
    > **ERROR（错误）**：ERROR级别的日志用于记录错误情况，例如操作失败、异常抛出等。这些日志表明应用程序遇到了一些问题，但通常不会导致应用程序崩溃。ERROR级别的日志在生产环境中非常重要，以及时发现和修复错误。
    > **WARN（警告）**：WARNING级别的日志用于记录警告信息，例如潜在的问题或不正常的情况，但不会导致应用程序终止。这些日志可以帮助监控和排查问题。
    > **INFO（通知）**：INFO级别的日志用于记录应用程序的一般信息，例如应用程序启动、关键事件、操作完成等。这些日志对于了解应用程序的运行状态和性能是有用的，通常会在生产环境中启用。
    > **DEBUG（调试）**：DEBUG级别的日志用于记录应用程序的调试信息，例如变量的值、函数的执行路径等。它有助于开发人员查找和修复问题。DEBUG级别的日志在开发和测试阶段启用，但在生产环境中通常禁用，因为它可能会产生大量的输出。
    > **TRACE（跟踪）**：这是最详细的日志级别，通常用于记录应用程序中最细微的操作和事件，以帮助调试和追踪问题。TRACE级别的日志包含非常详细的信息，通常只在开发和调试阶段启用。
    >
  - 支持日志格式化输出，允许自定义日志消息的格式
- **常用C++ Log库**

  - spdlog：一个快速，灵活且易于使用的日志库。它支持多线程，并且可以定制日志格式。
  - boost.log：是 boost 库的一部分，提供了强大的日志功能，包括多级日志，日志记录过滤器和异步日志。
  - log4cxx：是 Apache 的一个项目，是一个功能强大的日志库，支持多级日志，日志记录过滤器，异步日志和 XML 配置。
  - glog：是 Google 开发的日志库，支持多线程，提供了命令行选项来配置日志记录，并支持在运行时动态更改日志级别。
  - Poco Logging Library：是 Poco C++ 库的一部分，提供了简单的日志记录功能，并且可以很容易地扩展。
- **实现**
  结合Cheron的[Game Engine](https://youtube.com/playlist?list=PLlrATfBNZ98dC-V-N3m0Go4deliWHPFwT&si=uWt66l688XuxNesQ)系列视频，最终选用[spdlog](https://github.com/gabime/spdlog)作为框架中Log模块的基础。

  - 设计一个类名为 `Log`，在类中声明两个静态指针分配管理框架日志对象和客户端日志对象。

    - Log.h

      ```cpp
      #include "spdlog/spdlog.h"
      #include "spdlog/sinks/stdout_color_sinks.h"
      #include "spdlog/fmt/ostr.h"
      class NEST_API Log
      {
      public:
          /// @brief 初始化Log
          /// @param SCoreName 框架段日志名称
          /// @param SClientName 客户端日志名称
          static void InitLogger(const QString& SCoreName = "Nest", const QString& SClientName = "SandBox");

          /// @brief 静态函数：获取框架端日志
          /// @return 
          inline static std::shared_ptr<spdlog::logger>& GetCoreLogger() { return s_CoreLogger; }
          /// @brief 静态函数：获取客户端日志
          /// @return 
          inline static std::shared_ptr<spdlog::logger>& GetClientLogger() { return s_ClientLogger; }

      private:
          static std::shared_ptr<spdlog::logger> s_CoreLogger;    // 框架端日志对象(智能指针)
          static std::shared_ptr<spdlog::logger> s_ClientLogger;  // 客户端日志对象(智能指针)
      };
      ```
    - Log.cpp

      ```cpp
      std::shared_ptr<spdlog::logger> Log::s_CoreLogger;
      std::shared_ptr<spdlog::logger> Log::s_ClientLogger;

      void Log::InitLogger(const QString& SCoreName, const QString& SClientName)
      {
          // 更改日志模式
          spdlog::set_pattern("%^[%T] %n: %v%$");

          // 初始化引擎端日志对象的颜色和等级
          s_CoreLogger = spdlog::stdout_color_mt(SCoreName.toStdString());
          s_CoreLogger->set_level(spdlog::level::trace);

          // 初始化客户端日志对象的颜色和等级
          s_ClientLogger = spdlog::stdout_color_mt(SClientName.toStdString());
          s_ClientLogger->set_level(spdlog::level::trace);
      }
      ```
  - 根据日志级别，分别定义了框架和客户端的**跟踪、信息、警告、错误、致命**等级的宏。根据宏定义，我们可以看出spdlog对应的日志级别所调用的函数分别为 `trace()`、`info()`、`warn()`、`error()`和 `critical()`。

    ```cpp
    // Core log macros
    #define NEST_CORE_TRACE(...)        ::Nest::Log::GetCoreLogger()->trace(__VA_ARGS__)
    #define NEST_CORE_INFO(...)         ::Nest::Log::GetCoreLogger()->info(__VA_ARGS__)
    #define NEST_CORE_WARN(...)         ::Nest::Log::GetCoreLogger()->warn(__VA_ARGS__)
    #define NEST_CORE_ERROR(...)        ::Nest::Log::GetCoreLogger()->error(__VA_ARGS__)
    #define NEST_CORE_CRITICAL(...)     ::Nest::Log::GetCoreLogger()->critical(__VA_ARGS__)

    // Client log macros
    #define NEST_CLIENT_TRACE(...)      ::Nest::Log::GetClientLogger()->trace(__VA_ARGS__)
    #define NEST_CLIENT_INFO(...)       ::Nest::Log::GetClientLogger()->info(__VA_ARGS__)
    #define NEST_CLIENT_WARN(...)       ::Nest::Log::GetClientLogger()->warn(__VA_ARGS__)
    #define NEST_CLIENT_ERROR(...)      ::Nest::Log::GetClientLogger()->error(__VA_ARGS__)
    #define NEST_CLIENT_CRITICAL(...)   ::Nest::Log::GetClientLogger()->critical(__VA_ARGS__)
    ```
  - 在 `Application`的构造函数中初始化Log，以实现Log模块启动和应用。

    ```cpp
    //Application.cpp
    Application::Application(int& argc, char** argv)
        : QApplication(argc, argv)
    {
        NEST_CORE_ASSERT(s_Instance != nullptr, "Application already exists!");
        s_Instance = this;

        // 初始化框架的Log
        Nest::Log::InitLogger();
        NEST_CORE_WARN("Initialiazed Log!");
        NEST_CORE_INFO("Hello Nest FrameWork!");
    }
    ```
- **运行效果**
  ![Log模块](/Log模块.jpg)
  从图上可以看到，`NEST_CORE_WARN("Initialiazed Log!");`和 `NEST_CORE_INFO("Hello Nest FrameWork!");`已正确输出到控制台中了。

---

### Reference

[1] [Game Engine](https://youtube.com/playlist?list=PLlrATfBNZ98dC-V-N3m0Go4deliWHPFwT&si=uWt66l688XuxNesQ)
[2] [spdlog](https://github.com/gabime/spdlog)
[3] [spdlog wiki](https://github.com/gabime/spdlog/wiki)
[4] [C++ 实现简易 log 日志库](https://blog.csdn.net/K346K346/article/details/46908167)
