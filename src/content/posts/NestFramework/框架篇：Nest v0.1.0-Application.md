---
title: 框架篇：Nest v0.1.0-Application
published: 2024-04-04
category: Nest Framework
tags: [Qt, C++]
draft: false
---
在平时使用应用程序时，我们经常会注意到应用在运行期间能够有效地管理资源，并在结束后进行适当的资源释放。这引发了我对于代码中是否存在一个类来管理应用程序资源的思考。在学习Qt开发时，我注意到每当涉及窗口程序时都会涉及到 `QApplication`类。通过研究文档，我了解到 `QApplication`类用于管理Qt应用程序，并提供相应的功能和职责。因此，在设计Nest框架时，我希望有一个类来管理程序运行时的所有应用相关资源。为此，我选择设计一个名为 `Application`的类来实现这一需求。

---

### 设计思路

1. 由于整个框架是基于Qt框架的开发，在设计Application时应继承Qt中的相关Application类。

   1. 在Qt中，一共有三种Application类：QCoreApplication、QGuiApplication和QApplication
   2. 三者之间的关系如图所示
      ![QApplication关系](QApplication.png)
   3. 三者的区别
      - `QCoreApplication`定义在**core模块**中，为应用程序提供了一个非gui的事件循环。

        ```cpp
        #include <QCoreApplication>
        #include <QDebug>

        int main(int argc, char *argv[]) 
        {
            QCoreApplication app(argc, argv);
            qDebug() << "Hello, World!";
            return app.exec();
        }
        ```

      - `QGuiApplication`定义在**gui模块**中，提供了额外的gui相关的设置，比如桌面设置，风格，字体，调色板，剪切板，光标等。

        ```cpp
        #include <QGuiApplication>
        #include <QLabel>

        int main(int argc, char *argv[]) 
        {
            QGuiApplication app(argc, argv);
            QLabel label("Hello, World!");
            label.show();
            return app.exec();
        }
        ```

      - `QApplication`定义在**widgets模块**中，是QWidget相关的，能设置双击间隔，按键间隔，拖拽距离和时间，滚轮滚动行数等，能获取桌面，激活的窗口，模式控件，弹跳控件等。

        ```cpp
        #include <QGuiApplication>
        #include <QLabel>

        int main(int argc, char *argv[]) 
        {
            QApplication app(argc, argv);
            QLabel label("Hello, World!");
            label.show();
            return app.exec();
        }
        ```

2. 对于Application来说，我希望它能够在应用程序启动时都保证当且只有一个应用程序存在，因此使用**单例模式**来设计Application。
3. 在类中，设计一个 `bool`类型的变量用于控制应用程序运行状态，设置一个 `QMainWindow`的指针用于管理应用程序主窗口。

### 代码

- Application.h

  ```cpp
  /// @brief 应用程序类
  class NEST_API Application : public QApplication
  {
      Q_OBJECT
  public:
      Application(int& argc, char** argv);
      virtual ~Application() {}

  public:
      /// @brief 运行应用程序
      void Run();
      /// @brief 设置主窗口
      /// @param pMainWindow 
      void SetMainWindow(QMainWindow* pMainWindow);
      /// @brief 获取主窗口
      /// @return 当前活动窗口指针
      inline QMainWindow* GetMainWindow() { return m_Window.get(); }
      /// @brief 获取当前应用程序
      /// @return 
      inline static Application& GetApplication() { return *s_Instance; }

  private:
      /// @brief 应用程序单例: 程序运行时，当且只能一个应用程序存在
      static Application* s_Instance;
      /// @brief 运行状态的控制
      bool m_bRunning = false;
      /// @brief 应用程序主窗口
      std::unique_ptr<QMainWindow> m_Window;
  };
  ```

  - `Application`继承于Qt的 `QApplication`，使其可以管理Qt的应用程序。
  - 使用宏 `Q_OBJECT`，使得Qt的元对象系统支持 `Application`，能够使用信号和槽机制。
  - `static Application* s_Instance`：使用单例模式，定义了一个应用程序单例 ，确保程序运行时，当且只有一个应用程序存在。
  - `std::unique_ptr<QMainWindow> m_Window`：通过智能指针管理应用程序的主窗口，且通过 `std::unique_ptr`独占所管理对象的所有权，保证应用程序运行时只有一个主窗口。
  - `bool m_bRunning`：用于控制应用程序的运行状态。
- Application.cpp

  ```cpp
  Application* Application::s_Instance = nullptr;

  Application::Application(int& argc, char** argv)
  : QApplication(argc, argv)
  {
      NEST_CORE_ASSERT(s_Instance != nullptr, "Application already exists!");
      s_Instance = this;
  }

  void Application::Run()
  {
      m_bRunning = true;
      m_Window->show();
  }

  void Application::SetMainWindow(QMainWindow* pMainWindow)
  {
      m_Window = std::unique_ptr<QMainWindow>(pMainWindow);
  }
  ```
