---
title: 框架篇：NestUI v0.1.0-DockWidget
published: 2024-04-06
category: Nest Framework
tags: [Qt, C++]
draft: false
---
在本篇中，我们将设计Nest框架UI库中的**停靠窗口**和**DockWidget窗口管理类**。

### 设计思路

> 在Qt中，QDockWidget提供了停靠窗口部件的概念，也称为工具选项板或实用程序窗口。停靠窗口是放置在QMainWindow中中央部件周围的停靠小部件区域中的辅助窗口。
> QDockWidget由标题栏和内容区域组成。 标题栏显示停靠小部件窗口标题、浮动按钮和关闭按钮。 根据 QDockWidget 的状态，浮动和关闭按钮可能被禁用或根本不显示。
> 标题栏和按钮的视觉外观取决于所使用的样式。
> QDockWidget充当其子窗口小部件的包装器，通过setWidget()设置。自定义尺寸提示、最小和最大尺寸以及尺寸策略应在子窗口小部件中实现。QDockWidget将尊重它们，调整其自身的约束以包括框架和标题。 不应在QDockWidget本身上设置大小约束，因为它们会根据它是否停靠而变化； 停靠的 QDockWidget 没有框架和较小的标题栏。

#### DockWidget

- **功能设计**
  作为框架UI库中的DockWidget，我们希望其具有以下功能和特性：

  - 由于整个Nest框架是基于Qt开发的，所以主窗口类一定要能满足Qt相关特性（e.g：信号和槽机制）
  - 支持窗口设计，即设置窗口初始大小，窗口标题，窗口图标等。
  - 支持可扩展，能够根据不同应用场景进行对应的继承扩展。
- **实现**

  - 设计一个名为 `DockWidget`的类，此类继承于 `QDockWidget`，使其满足Qt停靠窗口相关功能和特性。
  - 设计一个名为 `clearWidget`方法，用于清空DockWidget内容。
  - 设计一个名为 `refreshWidget`方法，用于刷新DockWidget内容。
    - DockWidgeet.h

      ```cpp
      class NESTUI_API DockWidget : public QDockWidget
      {
          Q_OBJECT
      public:
          DockWidget(QWidget* parent = nullptr);
          ~DockWidget();

      public:
          /// @brief 清空DockWidget内容
          virtual void clearWidget() = 0;
          /// @brief 刷新DockWidget
          virtual void refreshWidget() = 0;
      };
      ```

#### DockWidgetManager

在框架中，我们希望有一个对象能管理应用程序主窗口中的所有停靠窗口，且能够提供相关增删改查的方法。

- **功能设计**

  - 这个类的对象在应用程序主窗口生命周期内当前仅存在一个，因此我们可以使用**单例模式**设计这个类。
  - 提供相关增删改查的方法。
- **实现**

  - 设计一个名为 `DockWidgetManager`的类，使用单例模式设计。
  - 在类中定义一个 `QMap`类型的成员变量，实现DockWidget对应的窗口标题与对应的DockWidget对象对应。
    - DockWidgetManager.h

      ```cpp
      /// @brief DockWidget窗口管理类
      class NESTUI_API DockWidgetManager
      {
      public:
          /// @brief 获取当前DockManager
          /// @return 
          static DockWidgetManager* GetDockManager(QMainWindow* mainWindow);

          /// @brief 添加DockWidget
          /// @param STitle DockWidget对应的窗口标题
          /// @param dockWidget 
          /// @return 若对应窗口标题的dockwidget存在，返回false
          bool AddDockWidget(const QString& STitle, QDockWidget* dockWidget, Qt::DockWidgetArea area);

          /// @brief 获取DockWidget
          /// @param STitle DockWidget对应的窗口标题
          /// @return 
          QDockWidget* GetDockWidget(const QString& STitle);

          /// @brief 移除DockWidget
          /// @param STitle DockWidget对应的窗口标题
          void RemoveDockWidget(const QString& STitle);

          /// @brief 获取当前主窗口中的dockWidget个数
          /// @return 
          int GetDockWidgetNum() const { return m_DockWidgetMap.size(); }

      protected:
          DockWidgetManager(QMainWindow* mainWindow);
          ~DockWidgetManager();

      private:
          /// @brief DockWidget窗口管理单例: 程序运行时，每个主窗口当且只能有一个DockManager存在
          static DockWidgetManager* s_Instance;
          QMainWindow* m_mainWindow;    //应用程序主窗口
          //dockWidget Map
          QMap<QString, QDockWidget*> m_DockWidgetMap;
      };
      ```

      在 `AddDockWidget`函数中有一个 `Qt::DockWidgetArea`类型的变量。

      - `Qt::DockWidgetArea`是一个枚举类型，其作用是设置停靠窗口在主窗口中的位置。
      - 在默认情况下，停靠窗口在上下左右都可停靠。
      - 定义：

        ```cpp
        enum DockWidgetArea {
            LeftDockWidgetArea = 0x1,//主窗口左侧停靠区
            RightDockWidgetArea = 0x2,//主窗口右侧停靠区
            TopDockWidgetArea = 0x4,//主窗口顶部停靠区
            BottomDockWidgetArea = 0x8,//主窗口底部停靠区

            DockWidgetArea_Mask = 0xf,
            AllDockWidgetAreas = DockWidgetArea_Mask,
            NoDockWidgetArea = 0
        };
        ```
    - DockWidgetManager.cpp

      ```cpp
      DockWidgetManager* DockWidgetManager::s_Instance = nullptr;

      DockWidgetManager::DockWidgetManager(QMainWindow* mainWindow)
      {
          NEST_CORE_ASSERT(s_Instance != nullptr, "DockWidgetManager already exists!");
          m_mainWindow = mainWindow;
      }

      DockWidgetManager::~DockWidgetManager()
      {
          m_mainWindow = nullptr;
      }

      DockWidgetManager* DockWidgetManager::GetDockManager(QMainWindow* mainWindow)
      {
          // FIXED: 在此处插入 return 语句
          s_Instance = new DockWidgetManager(mainWindow);
          return s_Instance;
      }

      bool DockWidgetManager::AddDockWidget(const QString& STitle, QDockWidget* dockWidget, Qt::DockWidgetArea area)
      {
          if (m_DockWidgetMap.contains(STitle) == true)
          {
              NEST_CORE_ERROR(STitle.toStdString() + " already exists!");
              return false;
          }
          m_DockWidgetMap.insert(STitle, dockWidget);
          m_mainWindow->addDockWidget(area, dockWidget);
          return true;
      }

      QDockWidget* DockWidgetManager::GetDockWidget(const QString& STitle)
      {
          if (m_DockWidgetMap.contains(STitle) == true)
          {
              return m_DockWidgetMap[STitle];
          }
          return nullptr;
      }

      void DockWidgetManager::RemoveDockWidget(const QString& STitle)
      {
          if (m_DockWidgetMap.contains(STitle) == true)
          {
              m_mainWindow->removeDockWidget(m_DockWidgetMap[STitle]);
              m_DockWidgetMap.remove(STitle);
          }
      }
      ```

---

### Reference

[1] [QT之QDockWidget使用详解](https://blog.csdn.net/qq_39295354/article/details/104036697)
[2] [Qt6.5 官方文档](https://doc.qt.io/qt-6.5/)
