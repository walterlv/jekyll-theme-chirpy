---
title: "EFI 分区/恢复分区不可删除？你需要使用命令行了（配合鼠标操作）"
date: 2019-10-07 11:05:02 +0800
tags: windows
position: knowledge
coverImage: /static/posts/2019-10-07-10-02-13.png
permalink: /post/delete-efi-partition-that-cannot-be-deleted-1.html
---

Windows 系统在安装的时候，会自动为我们的磁盘划分一个恢复分区和一个 EFI 分区。如果后面不打算再用这些分区的时候，却发现无法删除。

本文将提供解决方法。

---

因为误操作会导致数据丢失，所以我将两种不同的解决方法分开成两篇文章以避免干扰：

- [EFI 分区/恢复分区不可删除？你需要使用命令行了（配合鼠标操作）](/post/delete-efi-partition-that-cannot-be-deleted-1)
- [EFI 分区/恢复分区不可删除？你需要使用命令行了（全命令行操作）](/post/delete-efi-partition-that-cannot-be-deleted-2)

<div id="toc"></div>

## 无法删除

看下图，有两种不同类型的无法删除：

1. 有完整菜单只是删除按钮不可用的 EFI 分区；
2. 仅有一个“帮助”菜单的恢复分区。

删除方法会略有不同，我会在合适的地方提示你使用正确的方法的。

![无法删除](/static/posts/2019-10-07-10-02-13.png)

我的磁盘 2 原本包含两个可见分区，一个是图中黑色色块，原来放的是旧操作系统，一个是图中的 D 盘，放大量文件。因为我新买了一个大容量 SSD 专门用来放操作系统，所以原来操作系统所在的磁盘就可以回收与 D 盘合并。

然而悲剧的是，中间隔着一个 820MB 的恢复分区，导致我没有办法为 D 分区扩容。

更麻烦的是，在磁盘管理中，这三个我不会再使用的恢复分区都不可删除。

PS. 吐槽一下，大版本升级一次 Windows 10 竟然会在后面给我多创建一个恢复分区……

## 解决办法

在实操之前，你必须清除地知道你每一步在做什么，否则你需要承担丢失数据的后果：

- 如果你在网上找到一些操作 disk 的命令，请不要相信——**因为此命令清除的是整个磁盘而不只是单个分区**

### 第一步：打开命令提示符

打开开始菜单，输入 `cmd` 然后回车确定，我们可以打开命令提示符

![cmd](/static/posts/2019-10-07-10-11-03.png)

### 第二步：打开 diskpart

在 `cmd` 中输入 `diskpart` 然后回车，你会看到一个 UAC 提示弹窗，点击“是”之后会启动一个新的管理员权限启动的命令提示符，这里运行着 Diskpart 程序。

![diskpart](/static/posts/2019-10-07-10-14-07.png)

### 第三步：找到要操作的磁盘

输入 `list disk` 回车，我们可以看到自己计算机中所有已经插入的磁盘。

```powershell
DISKPART> list disk

  磁盘 ###  状态           大小     可用     Dyn  Gpt
  --------  -------------  -------  -------  ---  ---
  磁盘 0    联机              238 GB      0 B
  磁盘 1    联机              931 GB  2048 KB
  磁盘 2    联机              489 GB   199 GB        *
  磁盘 3    联机              476 GB  1024 KB        *

```

请注意，这里看到的是磁盘，而不是平时在“计算机”中看到的分区——每一个磁盘都可以包含一个到多个分区哦！

你有两种方法来确认我们即将操作的是哪个磁盘：

1. 前面我们在磁盘管理中看到的那个界面，也就是本文一开始的那张图，可以直接看出我们要删除的分区在“磁盘 2”上。
2. 根据分区的大小去猜，相信分区少的时候你可以猜对。

好的，我们知道要操作的是“磁盘 2”，于是我们输入命令 `select disk 2`（如果你是其他磁盘请换成自己的数字）：

```powershell
DISKPART> select disk 2

磁盘 2 现在是所选磁盘。

```

紧接着，我们输入 `list partition` 列出此磁盘上的所有分区：

```powershell
DISKPART> list partition

  分区 ###       类型              大小     偏移量
  -------------  ----------------  -------  -------
  分区      1    恢复                 499 MB  1024 KB
  分区      2    系统                 100 MB   500 MB
  分区      3    保留                  16 MB   600 MB
  分区      5    恢复                 820 MB   199 GB
  分区      6    主要                 288 GB   200 GB

```

通过分区，我们也能再次确认我们找到了正确的要操作的磁盘。

截至目前，我们还没有对系统进行任何更改，所以你操作错了也不用担心。但接下来你就需要谨慎一些。

### 第 4.1 步：删除分区（仅适用于 EFI 分区）

![EFI 分区](/static/posts/2019-10-07-10-39-34.png)

因为我不再将此磁盘用作系统盘，所以里面除了那个 288GB 的数据部分不能动之外，其他系统生成的部分都是需要删除的，所以接下来我需要对分区 1 2 5 都进行一遍以下操作（你的目的不同可能需要删除的分区也不一样）。

先选中要操作的分区：

```powershell
DISKPART> select partition 1

分区 1 现在是所选分区。

```

然后更改其 ID：

```powershell
DISKPART> SET ID=ebd0a0a2-b9e5-4433-87c0-68b6b72699c7

DiskPart 成功设置了分区 ID。

```

然后操作其他的分区。

完整的截图如下：

![完整的截图](/static/posts/2019-10-07-10-30-17.png)

这个时候，回到磁盘管理中，F5 刷新，你可以看到原本不可删除的 EFI 分区，现在可以直接使用鼠标删除了。点击一下“删除卷”即可。

![在磁盘管理中删除卷](/static/posts/2019-10-07-10-50-47.png)

### 第 4.2 步：删除分区（适用于所有类型的分区）

![恢复分区](/static/posts/2019-10-07-10-39-46.png)

恢复分区不能使用上面 4.1 中的方法删除，如果你在 4.1 的操作之后还发现存在不可删除的恢复分区，请尝试使用我的另一篇博客：

- [EFI 分区/恢复分区不可删除？你需要使用命令行了（全命令行操作）](/post/delete-efi-partition-that-cannot-be-deleted-2)

---

**参考资料**

- [windows10删除EFI分区(绝对安全) - 修炼之路 - CSDN博客](https://blog.csdn.net/sinat_29957455/article/details/88726797)
- [How to Delete EFI System Partition in Windows 10/8.1/8/7/XP/Vista - EaseUS](https://www.easeus.com/partition-master/delete-efi-system-partition.html)


