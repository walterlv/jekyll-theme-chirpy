---
title: "如何删除 Windows 10 系统生成的 WindowsApps 文件夹"
date: 2018-02-28 00:03:43 +0800
tags: windows sysprep
coverImage: /static/posts/2018-02-27-23-27-02.png
permalink: /post/how-to-delete-windows-apps-folder.html
---

如果曾经修改过 Windows 10 应用安装路径到非系统盘，那么那个盘下就会生成一些文件夹。如果以后重装了系统或者应用删除了，挪位置了，那些文件夹依然在那里——删不掉！

大家都知道这是权限问题，然而如何修改权限以便成功删除呢？

---

![更改应用的保存位置](/static/posts/2018-02-27-23-27-02.png)  
▲ 更改应用的保存位置

那么，现在开始解决删不掉的问题吧！

![删不掉](/static/posts/2018-02-27-23-35-59.png)

![进不去](/static/posts/2018-02-27-23-42-28.png)

---

## 第一步：属性→安全→高级

![属性→安全→高级](/static/posts/2018-02-27-23-43-28.png)

![高级安全设置](/static/posts/2018-02-27-23-44-09.png)

## 第二步：更改所有者

![更改所有者](/static/posts/2018-02-27-23-44-57.png)  
▲ 更改所有者

![输入自己的用户名](/static/posts/2018-02-27-23-45-24.png)  
▲ 在这里输入自己的用户名（如果是在线账户，则是邮箱；如果是本地账户，则是本地用户名）

![检查名称](/static/posts/2018-02-27-23-46-34.png)  
▲ 检查名称（点击之后会显示自己的名称）

![替换子容器和对象的所有者](/static/posts/2018-02-27-23-48-17.png)  
▲ 确定之后记得勾选“替换子容器和对象的所有者”

![确定](/static/posts/2018-02-27-23-49-29.png)  
▲ 确定以关掉这个对话框

于是就会有一个等待窗口，等待即可：

![更改所有权](/static/posts/2018-02-27-23-58-56.png)  
▲ 更改所有权

### 第三步：更改权限

这时再次点击高级，“高级安全设置”对话框中的“更改权限”按钮可以点了：

![更改权限](/static/posts/2018-02-27-23-51-16.png)  
▲ 更改权限

![添加权限](/static/posts/2018-02-27-23-52-47.png)  
▲ 现在可以添加权限了

![选择主体](/static/posts/2018-02-27-23-53-35.png)  
▲ 选择主体

![检查名称](/static/posts/2018-02-27-23-46-34.png)  
▲ 用同样的方式检查名称

![完全控制](/static/posts/2018-02-27-23-55-20.png)  
▲ 完全控制

![查看已添加的账号](/static/posts/2018-02-27-23-55-54.png)  
▲ 发现自己已被添加

一路点击确认，就设置好啦：

![设置安全信息](/static/posts/2018-02-27-23-56-33.png)  
▲ 设置安全信息

### 享受成果

现在删除，即可完成！

![删除](/static/posts/2018-02-28-00-02-59.png)  
▲ 删除


