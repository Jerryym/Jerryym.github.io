---
title: Unity备忘录
published: 2025-01-12
description: ''
image: ''
tags: [Unity]
category: 'Memorandum'
draft: false 
lang: ''
---

## 基础

### Monobehaviour类

* `Monobehaviour`是Unity中最重要的类，基本上编写的Unity脚本都是继承自`Monobehaviour`。
* `Start()`方法
  * 在第一帧时调用。
  * **作用**：一般用来进行一些初始化操作(e.g: 初始化变量等)，或者是在首次执行`Update()`之前需要执行的逻辑。
* `Update()`方法
  * 每一帧都会调用。
  * **作用**：一般用来进行一些逻辑判断、逻辑更新操作(e.g: 移动，动画播放等)，以及监听键盘、鼠标等事件。

---

## 动画

### 在Animation中使用状态机脚本控制动画状态

* **使用步骤**
  * 创建一个状态机脚本，此脚本中的类继承自 `StateMachineBehaviour`。然后进入Animator面板，选择动画状态机里的任意状态，点击Add Behaviour挂载脚本。
* **Example**

    ```csharp
    /// <summary>
    /// 受伤动画状态机
    /// </summary>
    public class InjuryAnimation : StateMachineBehaviour
    {
        // OnStateEnter is called when a transition starts and the state machine starts to evaluate this state
        override public void OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
        {
        
        }

        // OnStateUpdate is called on each Update frame between OnStateEnter and OnStateExit callbacks
        override public void OnStateUpdate(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
        {
        
        }

        // OnStateExit is called when a transition ends and the state machine finishes evaluating this state
        /// <summary>
        /// 状态退出时，重置角色受伤状态
        /// </summary>
        /// <param name="animator"></param>
        /// <param name="stateInfo"></param>
        /// <param name="layerIndex"></param>
        override public void OnStateExit(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
        {
            //状态退出时，重置角色受伤状态
            animator.GetComponent<PlayerController>().IsInjured = false;
        }

        // OnStateMove is called right after Animator.OnAnimatorMove()
        override public void OnStateMove(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
        {
            // Implement code that processes and affects root motion
        }

        // OnStateIK is called right after Animator.OnAnimatorIK()
        override public void OnStateIK(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
        {
            // Implement code that sets up animation IK (inverse kinematics)
        }
    }
    ```

* **可重写的消息函数**

  | 方法名         | 作用 |
  | -------------- | ---- |
  | `OnStateEnter` | 当状态机评估该状态时，会在**第一帧**更新时调用。 |
  | `OnStateUpdate` |当状态机评估该状态时，**除去第一帧和最后一帧**，都会在每一帧更新时调用。 |
  | `OnStateExit`  | 当状态机评估该状态时，会在**最后一帧**更新时调用。 |
  | `OnStateMove`  | 在动画根运动过程(Animator Root Motion pass)中被调用。 执行此信息可按状态修改动画根运动的结果。 |
  | `OnStateIK`    | 在动画的IK传递过程(Animator IK pass)中被调用。 执行此信息可在逐个状态评估状态机后修改动画结果。 |
