---
title: 利用 iDRAC 为服务器远程安装系统
tags:
  - iDRAC
---

## 背景

实验室的服务器需要换系统，由于服务器和我们维护人员不在同一个校区，物理安装会比较麻烦。还好之前服务器配置了 Integrated Dell Remote Access Controller (iDRAC) 远程管理系统，本次尝试利用它为服务器远程安装系统。

> The iDRAC is a piece of hardware that sits on the server motherboard that allows Systems Administrators to update and manage Dell systems, even when the server is turned off.
>
> The iDRAC also provides both a web interface and command line interface that allows administrators to perform remote management tasks. Almost all current Dell servers have the option for an iDRAC.
>
> An advantage of the iDRAC include being able to launch a remote console session (similar to Remote Desktop but works even when the OS is not booted up), remote reboot or power on/off your system, and map a virtual disk drive to a remote file share or OS deployment.
>
> [source](https://www.dell.com/support/kbdoc/en-us/000179517/dell-poweredge-how-to-configure-the-idrac-system-management-options-on-servers)



## 记录

### 下载系统镜像

我们选择的系统是 Ubuntu Server 20.04，将镜像文件下载到本地并校验。

![](https://img.yusanshi.com/upload/20220910131716638140.png)


```bash
yu@yu-ubuntu:~/Downloads$ sha256sum ubuntu-20.04.5-live-server-amd64.iso | grep 5035be37a7e9abbdc09f0d257f3e33416c1a0fb322ba860d42d74aa75c3468d4
5035be37a7e9abbdc09f0d257f3e33416c1a0fb322ba860d42d74aa75c3468d4  ubuntu-20.04.5-live-server-amd64.iso
```

### 进入 Lifecycle Controller

打开浏览器，登录进入 iDRAC Web 管理界面。

![](https://img.yusanshi.com/upload/20220910131510994594.png)


![](https://img.yusanshi.com/upload/20220910131511551856.png)

点击右侧的 Virtual Console，浏览器弹出来一个新窗口，我们接下来的操作都将在这个 Virtual Console 中进行。

![](https://img.yusanshi.com/upload/20220910131511736683.png)

由于该服务器系统坏得比较严重，常规方法已经难以重启机器以进入 Lifecycle Controller，于是我们直接选择硬重启：

![](https://img.yusanshi.com/upload/20220910131511520300.png)

重启时，根据这里的提示，按 F10 键进入 Lifecycle Controller（可以先点击上方的 Keyboard 调出键盘）。

![](https://img.yusanshi.com/upload/20220910131511513895.png)

![](https://img.yusanshi.com/upload/20220910131512380180.png)

### 配置 OS Deployment

直接用鼠标点击 OS Deployment，继续点击 Deploy OS，接下来跟着向导一步一步走。

![](https://img.yusanshi.com/upload/20220910131512407041.png)

选择直接进入系统安装。

![](https://img.yusanshi.com/upload/20220910131512816241.png)

这一步选择 Available Operating Systems 时，它已经给我们提供了一些可用的系统，但其中没有我们想用的 Ubuntu Server 20.04，所以我们选择 Any Other Operating System，以使用自己提供的 OS 镜像。

![](https://img.yusanshi.com/upload/20220910131512726022.png)

手动安装。

![](https://img.yusanshi.com/upload/20220910131513286229.png)

到了这一步，发现找不到 OS Media，毕竟我们的系统镜像文件还没有挂载。

![](https://img.yusanshi.com/upload/20220910131513956366.png)

我们点击上方的 Connect Virtual Media。

![](https://img.yusanshi.com/upload/20220910131513151679.png)

选择本地的系统安装镜像，映射上去。

![](https://img.yusanshi.com/upload/20220910131513789434.png)

点击 Back 再点击 Next 让它刷新下，就可以看到我们的虚拟 CD 了。

![](https://img.yusanshi.com/upload/20220910131513835670.png)

配置结束，点击 Finish 后即可重启进入 OS 安装。

![](https://img.yusanshi.com/upload/20220910131514726735.png)

### OS 安装

安装！

![](https://img.yusanshi.com/upload/20220910131514457115.png)

检查了好久……

![](https://img.yusanshi.com/upload/20220910131514601236.png)

选择语言、键盘布局之类的。

![](https://img.yusanshi.com/upload/20220910131514488361.png)

配置网络。点击第一个网卡（只有它的状态是 connected），按需编辑。

![](https://img.yusanshi.com/upload/20220910131514232188.png)

编辑完成。

![](https://img.yusanshi.com/upload/20220910131515121142.png)

没有代理，直接确定。

![](https://img.yusanshi.com/upload/20220910131515376840.png)

使用科大的镜像源（毕竟服务器就在科大 😆）。

![](https://img.yusanshi.com/upload/20220910131515846130.png)

硬盘配置，这里直接选择了 Use an entire disk。

![](https://img.yusanshi.com/upload/20220910131515690834.png)

发现它自动生成的配置只给 `ubuntu-lv`（ 挂载到 `/`）分配了 100GB，还剩 344GB 空余空间。

![](https://img.yusanshi.com/upload/20220910131515575269.png)

选择下方的 `ubuntu-lv`，编辑该分区，把它的容量从 100GB 增大至最大（444GB）。

![](https://img.yusanshi.com/upload/20220910131516874158.png)

修改完后的硬盘分区方案如下。

![](https://img.yusanshi.com/upload/20220910131516960283.png)

设定账号信息。

![](https://img.yusanshi.com/upload/20220910131516229092.png)

勾选上安装 SSH 服务。

![](https://img.yusanshi.com/upload/20220910131516691033.png)

跳过。

![](https://img.yusanshi.com/upload/20220910131516826655.png)

等待安装。

![](https://img.yusanshi.com/upload/20220910131517407908.png)

选择重启时，提示移除安装介质失败，我们进入前面操作过的 Virtual Media 界面，手动移除安装介质，然后按 Enter。

![](https://img.yusanshi.com/upload/20220910131517375443.png)

重启，安装完成 🚀🚀🚀。

![](https://img.yusanshi.com/upload/20220910131517146468.png)
