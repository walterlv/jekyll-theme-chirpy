---
title: "神器如 dnSpy，无需源码也能修改 .NET 程序"
date: 2018-05-22 22:02:13 +0800
tags: dotnet
coverImage: /static/posts/2018-05-22-21-45-11.png
permalink: /post/edit-and-recompile-assembly-using-dnspy.html
---

[dnSpy](https://github.com/0xd4d/dnSpy) 是 [0xd4d](https://github.com/0xd4d) 开发的 .NET 程序调试神器。

说它是神器真的毫不为过！它能在完全没有源码的情况下即时调试程序，甚至还能修改程序！本文讲向大家介绍如何使用 dnSpy 修改 .NET 程序。

---

dnSpy 的主打功能是无需源码的调试，[林德熙](https://blog.lindexi.com/) 有一篇文章 [断点调试 Windows 源代码](https://blog.lindexi.com/post/%E6%96%AD%E7%82%B9%E8%B0%83%E8%AF%95-Windows-%E6%BA%90%E4%BB%A3%E7%A0%81.html) 介绍了这个方法。而本文主要说其另一项强大的功能 —— 修改程序集。

<div id="toc"></div>

## 看看 dnSpy

![dnSpy](/static/posts/2018-05-22-21-45-11.png)

dnSpy 长着一身 Visual Studio 一样的外观，调试的时候给你熟悉的感觉。

我们只需要讲我们需要调试或修改的程序集拖入左侧的程序集列表中即可（它会自动为我们把此程序集依赖的程序集也添加进来）。我把以前我写过的一个程序 `ManipulationDemo` 拖进来了。

## 实操修改程序集

现在我们来修改它，修改什么好呢？为了让效果明显一点，我决定在启动时弹一个窗口。于是我们展开进入到 `App` 类中。

![App 类](/static/posts/2018-05-22-21-48-55.png)

然后在类中右键“Edit class (C#)”：

![右键 -> 编辑类](/static/posts/2018-05-22-21-50-05.png)

在里面重写 `OnStartup` 方法。发现，它竟然连智能感知提示都做了！

![重写 OnStartup 方法](/static/posts/2018-05-22-21-51-37.png)

![MessageBox.Show](/static/posts/2018-05-22-21-52-52.png)

改完只需要点击一下右下角的编译，即可讲修改应用到我们刚刚打开的程序集中。

![编译](/static/posts/2018-05-22-21-53-43.png)

## 保存修改的程序集

如果只是修改了可以立刻运行，那么充其量只是可以辅助调试。但是 dnSpy 是可以将程序集另存到本地的。

点击“File”->“Save Module”：

![保存模块](/static/posts/2018-05-22-21-56-00.png)

为了以示区分，我写了一个新的名字：

![保存](/static/posts/2018-05-22-21-56-51.png)

保存完之后，运行：

![运行](/static/posts/2018-05-22-21-59-53.png)

我们会发现，我们刚刚新增的对话框已经弹出来了。“OK”之后原来的窗口才会显示出来。

## 发挥想象力的时候到了

既然有如此简单的修改程序集的方法，那么我们可以用来做什么事儿呢？用来做什么事儿呢？做什么事儿呢？什么事儿呢？事儿呢？呢？

**想象力***时间*

顺便说一下，就算程序集被混淆了也难不倒它。


