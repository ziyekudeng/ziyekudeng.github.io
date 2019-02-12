---
layout: post
title: 【Idea】中一个很有用的内存调试插件
category: tools
tags: [tools]
---

# 【Idea】中一个很有用的内存调试插件


![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/1.png)

JetBrains JVM Debugger Memory View plugin

在我最近的研发活动期间寻找新的工具，以提高我的开发经验，使Android Studio的生活更轻松，我发现一个有用的插件，我从来没有听说过。 这就是为什么，我决定写这个强大的工具，它如何帮助我与内存调试我的应用程序。

### What is the plugin about?

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/2.png)

此插件扩展了内置的JVM调试器，具有在调试会话期间观察JVM堆中的对象的功能。

内存视图按照类名称分组来显示 **堆中的对象总数** 。

当你一步步调试代码时， “Diff”列显示调试器停靠点(debugger stops也就是debug点)之间对象数量的变化 。 这种方式你可以很容易地看到你的步进代码如何影响堆。

双击类名称，打开一个包含该类实例的对话框。 该对话框允许您 **通过计算表达式过滤实例** 。 所有调试器操作（如检查，标记对象，评估表达式，添加到观察等）都可以应用于此对话框中的实例。

### How to install this wonderful plugin?

打开Android Studio **Plugins** 页面：

*   **快捷键：** 按 **command/ ctrl** + **shift** + **A，** 类型 **插件** 随后，按 **enter** 键：
*   或打开 **Preferences/Settings:** （Mac：Android Studio - >Preferences **/** Windows和Linux：File - >Settings）并找到 **Plugins** 页面：

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/3.png)

按 **Install jetBrains plugin…** 按钮，搜索 **JVM Debugger Memory View** 然后 **Install** 。

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/4.png)

装完重新启动Android Studio。

At first glance:

回到Android Studio后，您会发现 **Memory View Tool Window** 已经添加到工具栏的右侧。

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/5.png)

Memory View Tool Window

内存视图工具窗口

如果没有看到内存视图，打开工具窗口，使用主菜单： View → Tool Windows → Memory View。

首先，这个工具只有在打了调试断点并在 debug模式 运行期间才会显示数据。

其次，我要提到的是，我阅读了Android Studio可能会发生的一些警告和错误，不过，我并没有碰到过。

警告：Android Studio版本包含以下限制：

*   由于Android内存限制，获取大量的实例可能会失败，并会停止VM。

*   Android Studio可能会停止响应

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/6.png)

### Let’s debug!

在调试模式下运行应用程序并在BreakPoint上暂停后，您会看到很神奇的画面：

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/7.png)

这个表让我们最感兴趣的地方是 **Diff** ”列，当你一步步调试代码行时，你将看到会有多少新的对象实例被创建或销毁！

我想寻找我自己的对象 （即ProfileModel类） ，所以我搜索它：

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/8.png)

正如你可以看到我已经在这行代码更新了ProfileModel vairable，在GC删除旧对象之前我得到差异是+1 ，也可以访问之前不可能访问到的旧的对象。 通过双击这条记录，我将在窗口中获取ProfileModel类的实例：

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/9.png)

此窗口还允许你使用类方法通过计算的表达式过滤实例，例如，您可以使用 **OkHttp Response** 类的 **isSuccessful** 方法来过滤筛选在内存中加载不成功的响应：

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/10.png)

实例过滤器功能

另一个有用的功能是跟踪新实例，您可以通过Memory View Tool窗口中的右键菜单启用：

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/11.png)

此功能可帮助您跟踪已生成类的新实例的代码！

![](https://ziyekudeng.github.io/assets/images/2019/0212/jvm-debugger-memory-view-plugin/12.png)


 


 
