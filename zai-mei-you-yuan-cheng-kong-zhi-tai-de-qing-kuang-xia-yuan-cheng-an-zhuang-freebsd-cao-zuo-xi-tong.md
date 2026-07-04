# 在没有远程控制台的情况下远程安装 FreeBSD 操作系统

- 原文链接：[Remote Installation of the FreeBSD Operating System Without a Remote Console](https://docs.freebsd.org/en/articles/remote-install/)

## 摘要

本文记录了在远程系统的控制台不可用时，FreeBSD 操作系统的远程安装过程。本文的主要思想是与 Martin Matuska ([mm@FreeBSD.org](mailto:mm@FreeBSD.org)) 合作的结果，Paweł Jakub Dawidek ([pjd@FreeBSD.org](mailto:pjd@FreeBSD.org)) 提供了宝贵的意见。

## 1. 背景

世界上有许多服务器托管提供商，但很少有提供商官方支持 FreeBSD。他们通常支持在其提供的服务器上安装 Linux® 发行版。

在某些情况下，如果你提出请求，这些公司会为你安装你首选的 Linux® 发行版。通过这种方式，我们将尝试安装 FreeBSD。在其他情况下，他们可能提供救援系统，用于紧急情况。我们也可以借此达成目的。

本文介绍了远程安装具备 RAID-1 和 ZFS 功能的 FreeBSD 所需的基本安装和配置步骤。

## 2. 介绍

本节将总结本文的目的，并更好地解释其中的内容。本文中的说明将对使用不支持 FreeBSD 的托管服务的用户有所帮助。

1. 正如我们在 [背景](https://docs.freebsd.org/en/articles/remote-install/#background) 部分提到的，许多知名的服务器托管公司提供某种形式的救援系统，该系统通过他们的局域网启动，并可通过 SSH 访问。通常他们提供此类支持以帮助客户修复操作系统故障。正如本文所解释的，借助这些救援系统，实际上是可以安装 FreeBSD 的。
2. 本文的下一部分将描述如何在本地计算机上配置并构建最小化的 FreeBSD 系统。这个版本最终将通过 ramdisk 运行在远程计算机上，这将允许我们使用 sysinstall 工具从 FTP 镜像安装完整的 FreeBSD 操作系统。
3. 本文的其余部分将描述安装过程本身，以及 ZFS 文件系统的配置。

### 2.1. 要求

为了成功继续，你必须：

- 拥有可访问网络的操作系统，并且支持 SSH 访问
- 理解 FreeBSD 的安装过程
- 熟悉 [sysinstall(8)](https://man.freebsd.org/cgi/man.cgi?query=sysinstall&sektion=8&format=html) 工具
- 手头有 FreeBSD 安装 ISO 映像或 CD

## 3. 准备——mfsBSD

在 FreeBSD 可以安装到目标系统之前，必须构建最小化的 FreeBSD 操作系统映像，该映像将从硬盘启动。这样，新的系统就可以通过网络进行访问，并且可以在没有远程访问系统控制台的情况下完成剩余的安装过程。

可以使用 mfsBSD 工具集来构建小型的 FreeBSD 映像。正如 mfsBSD 名称所示（“mfs” 意味着 “内存文件系统”），生成的映像完全运行在 ramdisk 中。由于这个特性，硬盘的操作不受限制，因此可以安装完整的 FreeBSD 操作系统。mfsBSD 的[主页](http://mfsbsd.vx.sk/)包含指向工具集最新版本的链接。

请注意，mfsBSD 的内部实现及其如何运作超出了本文的范围。有兴趣的读者应该查阅 mfsBSD 的原始文档，以了解更多详细信息。

下载并解压最新的 mfsBSD 版本，并将你的工作目录切换到存放 mfsBSD 脚本的目录：

```sh
# fetch http://mfsbsd.vx.sk/release/mfsbsd-2.1.tar.gz
# tar xvzf mfsbsd-2.1.tar.gz
# cd mfsbsd-2.1/
```

### 3.1. mfsBSD 配置

在启动 mfsBSD 之前，需要设置一些重要的配置选项。最重要的配置是网络设置。配置网络选项的最佳方法取决于我们是否事先知道要使用的网络接口类型，以及需要为硬件加载的网络接口驱动程序。接下来，我们将介绍如何在这两种情况下配置 mfsBSD。

另一个需要设置的重要项是 `root` 密码。可以通过编辑 **conf/loader.conf** 来完成这项配置。请查看包含的注释内容。

#### 3.1.1. **conf/interfaces.conf** 方法

当已安装的网络接口卡未知时，可以使用 mfsBSD 的自动检测功能。mfsBSD 的启动脚本可以根据接口的 MAC 地址自动检测要使用的正确驱动程序。我们只需要在 **conf/interfaces.conf** 中设置以下选项：

```sh
mac_interfaces="ext1"
ifconfig_ext1_mac="00:00:00:00:00:00"
ifconfig_ext1="inet 192.168.0.2/24"
```

不要忘记在 **conf/rc.conf** 中添加 `defaultrouter` 信息：

```sh
defaultrouter="192.168.0.1"
```

#### 3.1.2. **conf/rc.conf** 方法

当网络接口驱动程序已知时，使用 **conf/rc.conf** 配置网络选项会更加方便。该文件的语法与 FreeBSD 标准的 [rc.conf(5)](https://man.freebsd.org/cgi/man.cgi?query=rc.conf&sektion=5&format=html) 文件相同。

例如，如果你知道将会使用 [re(4)](https://man.freebsd.org/cgi/man.cgi?query=re&sektion=4&format=html) 网络接口，可以在 **conf/rc.conf** 中设置以下选项：

```sh
defaultrouter="192.168.0.1"
ifconfig_re0="inet 192.168.0.2/24"
```

### 3.2. 构建 mfsBSD 映像

构建 mfsBSD 映像的过程非常简单。

第一步是挂载 FreeBSD 安装 CD 或安装 ISO 映像到 **/cdrom**。为了方便示例，本文假设你已经下载了 FreeBSD 10.1-RELEASE ISO。使用 [mdconfig(8)](https://man.freebsd.org/cgi/man.cgi?query=mdconfig&sektion=8&format=html) 工具挂载该 ISO 映像到 **/cdrom** 目录：

```sh
# mdconfig -a -t vnode -u 10 -f FreeBSD-10.1-RELEASE-amd64-disc1.iso
# mount_cd9660 /dev/md10 /cdrom
```

由于最近的 FreeBSD 发行版不包含常规的分发包，因此需要从 ISO 映像中的分发档案中提取 FreeBSD 分发文件：

```sh
# mkdir DIST
# tar -xvf /cdrom/usr/freebsd-dist/base.txz -C DIST
# tar -xvf /cdrom/usr/freebsd-dist/kernel.txz -C DIST
```

接下来，构建可启动的 mfsBSD 映像：

```sh
# make BASE=DIST
```

>**注意**
>
>必须从 mfsBSD 目录树的顶层目录（如 **~/mfsbsd-2.1/**）运行上述 `make` 命令。


### 3.3. 启动 mfsBSD

现在 mfsBSD 映像已经准备好，必须将其上传到运行实时救援系统或预安装 Linux® 发行版的远程系统。完成这项任务最合适的工具是 scp：

```sh
# scp disk.img root@192.168.0.2:.
```

为了正确启动 mfsBSD 映像，它必须放置在给定机器的第一个（可启动）设备上。假设 **sda** 是第一个可启动的磁盘设备，可以使用以下命令来完成：

```sh
# dd if=/root/disk.img of=/dev/sda bs=1m
```

如果一切顺利，映像应该已经写入第一个设备的 MBR，可以重新启动机器。使用 [ping(8)](https://man.freebsd.org/cgi/man.cgi?query=ping&sektion=8&format=html) 工具观察机器是否正确启动。系统重新上线后，应该可以使用配置的密码以 `root` 用户通过 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 进行访问。

## 4. 安装 FreeBSD 操作系统

mfsBSD 已成功启动，应该可以通过 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 登录。接下来的部分将描述如何创建和标记切片，设置 `gmirror` 以实现 RAID-1，以及如何使用 `sysinstall` 安装最小的 FreeBSD 操作系统发行版。

### 4.1. 硬盘准备

第一步是为 FreeBSD 分配磁盘空间，也就是创建切片和分区。显然，当前正在运行的系统完全加载到系统内存中，因此在操作硬盘时不会出现问题。可以使用 `sysinstall` 或 [fdisk(8)](https://man.freebsd.org/cgi/man.cgi?query=fdisk&sektion=8&format=html) 和 [bsdlabel(8)](https://man.freebsd.org/cgi/man.cgi?query=bsdlabel&sektion=8&format=html) 配合使用来完成这项任务。

首先，将所有系统磁盘标记为空。对每个硬盘重复以下命令：

```sh
# dd if=/dev/zero of=/dev/ad0 count=2
```

接下来，使用你喜欢的工具创建切片并标记它们。虽然使用 `sysinstall` 更为简便，但使用标准的基于文本的 UNIX® 工具（如 [fdisk(8)](https://man.freebsd.org/cgi/man.cgi?query=fdisk&sektion=8&format=html) 和 [bsdlabel(8)](https://man.freebsd.org/cgi/man.cgi?query=bsdlabel&sektion=8&format=html)）是一种更强大且可能更少出错的方法，本节将介绍这种方法。前者在 FreeBSD 手册的 [安装 FreeBSD](https://docs.freebsd.org/en/books/handbook/#install-steps) 章节中有详细的文档。正如本文引言中提到的，这篇文章将介绍如何设置具备 RAID-1 和 ZFS 功能的系统。我们的设置将包括小的 [gmirror(8)](https://man.freebsd.org/cgi/man.cgi?query=gmirror&sektion=8&format=html) 镜像 **/**（根）、**/usr** 和 **/var** 数据集，其余磁盘空间将分配给 [zpool(8)](https://man.freebsd.org/cgi/man.cgi?query=zpool&sektion=8&format=html) 镜像 ZFS 文件系统。请注意，ZFS 文件系统将在 FreeBSD 操作系统成功安装并启动后配置。

以下示例将描述如何创建切片和标签，如何在每个分区上初始化 [gmirror(8)](https://man.freebsd.org/cgi/man.cgi?query=gmirror&sektion=8&format=html)，以及如何在每个镜像分区上创建 UFS2 文件系统：

```sh
# fdisk -BI /dev/ad0 ①
# fdisk -BI /dev/ad1
# bsdlabel -wB /dev/ad0s1 ②
# bsdlabel -wB /dev/ad1s1
# bsdlabel -e /dev/ad0s1 ③
# bsdlabel /dev/ad0s1 > /tmp/bsdlabel.txt && bsdlabel -R /dev/ad1s1 /tmp/bsdlabel.txt ④
# gmirror label root /dev/ad[01]s1a ⑤
# gmirror label var /dev/ad[01]s1d
# gmirror label usr /dev/ad[01]s1e
# gmirror label -F swap /dev/ad[01]s1b ⑥
# newfs /dev/mirror/root ⑦
# newfs /dev/mirror/var
# newfs /dev/mirror/usr
```

- ① 创建覆盖整个磁盘的切片，并初始化给定磁盘的第 0 扇区中的引导代码。对系统中的所有硬盘重复此命令。
- ② 为每个磁盘写入标准标签，包括启动代码。
- ③ 现在，手动编辑给定磁盘的标签。参考 [bsdlabel(8)](https://man.freebsd.org/cgi/man.cgi?query=bsdlabel&sektion=8&format=html) 手册页面了解如何创建分区。创建分区 `a` 用于 **/**（根）文件系统，`b` 用于交换分区，`d` 用于 **/var**，`e` 用于 **/usr**，最后 `f` 将用于 ZFS。
- ④ 将最近创建的标签导入到第二个硬盘，使两个硬盘的标签相同。
- ⑤ 在每个分区上初始化 [gmirror(8)](https://man.freebsd.org/cgi/man.cgi?query=gmirror&sektion=8&format=html)。
- ⑥ 请注意，`-F` 用于交换分区。这告诉 [gmirror(8)](https://man.freebsd.org/cgi/man.cgi?query=gmirror&sektion=8&format=html) 假设设备在电源/系统故障后处于一致状态。
- ⑦ 在每个镜像分区上创建 UFS2 文件系统。

### 4.2. 系统安装

这是最重要的部分。本节将描述如何将 FreeBSD 的最小发行版实际安装到我们在上一节中准备好的硬盘上。为了完成此目标，需要将所有文件系统挂载，以便 `sysinstall` 可以将 FreeBSD 的内容写入硬盘：

```sh
# mount /dev/mirror/root /mnt
# mkdir /mnt/var /mnt/usr
# mount /dev/mirror/var /mnt/var
# mount /dev/mirror/usr /mnt/usr
```

完成后，启动 [sysinstall(8)](https://man.freebsd.org/cgi/man.cgi?query=sysinstall&sektion=8&format=html)。从主菜单选择 **Custom** 安装。选择 **Options** 并按回车。使用箭头键将光标移至 `Install Root` 项目，按空格键并将其更改为 **/mnt**。按回车键提交更改，并按 `q` 退出 **Options** 菜单。

>**警告**
>
>请注意，这一步非常重要，如果跳过此步骤，`sysinstall` 将无法安装 FreeBSD。

进入 **Distributions** 菜单，使用箭头键将光标移至 `Minimal`，按空格键选择它。本文使用最小发行版以节省网络流量，因为系统将通过 ftp 安装。通过选择 `Exit` 退出该菜单。

>**注意**
>
>**Partition** 和 **Label** 菜单将被跳过，因为它们现在无用。

在 **Media** 菜单中，选择 `FTP`。选择最近的镜像，并让 `sysinstall` 假定网络已配置好。你将返回到 **Custom** 菜单。

最后，通过选择最后一个选项 **Commit** 来执行系统安装。安装完成后，退出 `sysinstall`。

### 4.3. 安装后步骤

现在 FreeBSD 操作系统应该已经安装完成；然而，过程尚未结束。需要执行一些安装后步骤，以确保 FreeBSD 在未来能够启动，并且能够登录系统。

现在，你必须 [chroot(8)](https://man.freebsd.org/cgi/man.cgi?query=chroot&sektion=8&format=html) 到新安装的系统中，以完成安装。使用以下命令：

```
# chroot /mnt
```

完成我们的目标，执行以下步骤：

- 将 `GENERIC` 内核复制到 **/boot/kernel** 目录：

  ```sh
  # cp -Rp /boot/GENERIC/* /boot/kernel
  ```
- 创建 **/etc/rc.conf**、**/etc/resolv.conf** 和 **/etc/fstab** 文件。不要忘记正确设置网络信息，并在 **/etc/rc.conf** 中启用 sshd。**/etc/fstab** 的内容将类似于以下内容：

  ```sh
  # Device                Mountpoint      FStype  Options         Dump    Pass#
  /dev/mirror/swap        none            swap    sw              0       0
  /dev/mirror/root        /               ufs     rw              1       1
  /dev/mirror/usr         /usr            ufs     rw              2       2
  /dev/mirror/var         /var            ufs     rw              2       2
  /dev/cd0                /cdrom          cd9660  ro,noauto       0       0
  ```
- 创建 **/boot/loader.conf**，其内容如下：

  ```sh
  geom_mirror_load="YES"
  zfs_load="YES"
  ```
- 执行以下命令，使 ZFS 在下次启动时可用：

  ```sh
  # sysrc zfs_enable="YES"
  ```
- 使用 [adduser(8)](https://man.freebsd.org/cgi/man.cgi?query=adduser&sektion=8&format=html) 工具向系统添加额外的用户。不要忘记将用户添加到 `wheel` 组，这样你可以在重启后获取 root 访问权限。
- 仔细检查你的所有设置。

系统现在应该准备好进行下次启动。使用 [reboot(8)](https://man.freebsd.org/cgi/man.cgi?query=reboot&sektion=8&format=html) 命令重新启动系统。

## 5. ZFS

如果系统在重启后仍然能够正常启动，现在应该可以登录。欢迎进入刚刚完成的 FreeBSD 系统，整个过程都是通过远程操作完成的，没有使用远程控制台！

最后一步是配置 [zpool(8)](https://man.freebsd.org/cgi/man.cgi?query=zpool&sektion=8&format=html) 并创建一些 [zfs(8)](https://man.freebsd.org/cgi/man.cgi?query=zfs&sektion=8&format=html) 文件系统。创建和管理 ZFS 非常简单。首先，创建镜像池：

```sh
# zpool create tank mirror /dev/ad[01]s1f
```

接下来，创建一些文件系统：

```sh
# zfs create tank/ports
# zfs create tank/src
# zfs set compression=gzip tank/ports
# zfs set compression=on tank/src
# zfs set mountpoint=/usr/ports tank/ports
# zfs set mountpoint=/usr/src tank/src
```

就这样。如果你对 FreeBSD 上 ZFS 的更多细节感兴趣，请参考 FreeBSD Wiki 上的 [ZFS](https://wiki.freebsd.org/ZFS) 部分。
