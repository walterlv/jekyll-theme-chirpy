---
title: "在 CMD 里根据进程名杀掉进程"
date: 2020-04-30 11:31:27 +0800
tags: windows
position: knowledge
permalink: /post/kill-process-by-name-using-cmd.html
---

任务管理器杀进程大家都会，不过如果你的系统被卡到任务管理器都无法操作了，怎么办？拿控制台 CMD 吧！

---

系列博客：

- [在 CMD 里根据进程名杀掉进程 - walterlv](/post/process-by-name-using-cmd)
- [在 PowerShell 里根据进程名杀掉进程 - walterlv](/post/process-by-name-using-powershell)

<div id="toc"></div>

## taskkill 命令

```powershell
taskkill /f /t /im chrome.exe
```

`/f` 表示强制终止进程。如果不指定，那么 taskkill 会给进程发送终止信号，但进程可以阻止退出（例如提示文档需要保存）。指定了，就会强杀进程。

`/t` 表示结束此进程和其子进程。

`/im` 用来指定进程的影映像名称（有 .exe 后缀）。

更多使用方法可以直接输入 `taskkill /?`。

```powershell
❯ taskkill /?

TASKKILL [/S system [/U username [/P [password]]]]
         { [/FI filter] [/PID processid | /IM imagename] } [/T] [/F]

描述:
    使用该工具按照进程 ID (PID) 或映像名称终止任务。

参数列表:
    /S    system           指定要连接的远程系统。

    /U    [domain\]user    指定应该在哪个用户上下文执行这个命令。

    /P    [password]       为提供的用户上下文指定密码。如果忽略，提示
                           输入。

    /FI   filter           应用筛选器以选择一组任务。
                           允许使用 "*"。例如，映像名称 eq acme*

    /PID  processid        指定要终止的进程的 PID。
                           使用 TaskList 取得 PID。

    /IM   imagename        指定要终止的进程的映像名称。通配符 '*'可用来
                           指定所有任务或映像名称。

    /T                     终止指定的进程和由它启用的子进程。

    /F                     指定强制终止进程。

    /?                     显示帮助消息。

筛选器:
    筛选器名      有效运算符                有效值
    -----------   ---------------           -------------------------
    STATUS        eq, ne                    RUNNING |
                                            NOT RESPONDING | UNKNOWN
    IMAGENAME     eq, ne                    映像名称
    PID           eq, ne, gt, lt, ge, le    PID 值
    SESSION       eq, ne, gt, lt, ge, le    会话编号。
    CPUTIME       eq, ne, gt, lt, ge, le    CPU 时间，格式为
                                            hh:mm:ss。
                                            hh - 时，
                                            mm - 分，ss - 秒
    MEMUSAGE      eq, ne, gt, lt, ge, le    内存使用量，单位为 KB
    USERNAME      eq, ne                    用户名，格式为 [domain\]user
    MODULES       eq, ne                    DLL 名称
    SERVICES      eq, ne                    服务名称
    WINDOWTITLE   eq, ne                    窗口标题

    说明
    ----
    1) 只有在应用筛选器的情况下，/IM 切换才能使用通配符 '*'。
    2) 远程进程总是要强行 (/F) 终止。
    3) 当指定远程机器时，不支持 "WINDOWTITLE" 和 "STATUS" 筛选器。

例如:
    TASKKILL /IM notepad.exe
    TASKKILL /PID 1230 /PID 1241 /PID 1253 /T
    TASKKILL /F /IM cmd.exe /T
    TASKKILL /F /FI "PID ge 1000" /FI "WINDOWTITLE ne untitle*"
    TASKKILL /F /FI "USERNAME eq NT AUTHORITY\SYSTEM" /IM notepad.exe
    TASKKILL /S system /U 域\用户名 /FI "用户名 ne NT*" /IM *
    TASKKILL /S system /U username /P password /FI "IMAGENAME eq note*"
```

---

**参考资料**

- [how to kill a program by process name using cmd in windows 8? - Stack Overflow](https://stackoverflow.com/a/29233238/6233938)

