# vm-bhyve Wiki

- 原文：[vm-bhyve wiki](https://github.com/churchers/vm-bhyve/wiki)
- 作者：Matt Churchyard

欢迎来到 vm-bhyve 的 wiki。

希望你能在这里找到有关使用 vm-bhyve 的信息、示例，以及我们当前支持和将来计划支持的详细内容。

感谢你的到访！

—— Matt

## 功能

**Apr 22, 2016**

这是一份（相当）完整的 vm-bhyve 当前支持功能清单。除非另有说明，所有功能均适用于一切类型的 guest。

* 配置文件简单，可轻松手动编辑
* FreeBSD guest（含基于 FreeBSD 的衍生版本，如 pfSense）
* Linux / OpenBSD / NetBSD guest
* Windows guest
* SmartOS guest
* guest 模板，可用于为新建 guest 指定配置、磁盘、网络等
* 支持运行于 ZFS 或其他任意文件系统上
* 多数据存储——可创建多个位置来存放虚拟机数据
* 支持稀疏文件、zvol、sparse zvol 和自定义磁盘设备
* 在 `nmdm` 控制台和标准输入输出上运行 guest 及安装程序
* 动态分配控制台用的 nmdm 设备
* 动态分配网络用的 tap 设备
* 支持任意数量的虚拟交换机，其接口可动态创建并桥接
* 虚拟交换机的 VLAN 支持
* 虚拟交换机的 NAT 支持
* 能够将 guest 连接到手动配置的桥接（`vm switch import myswitch bridgeX`）
* guest 与虚拟交换机的详细信息输出（`vm info` 与 `vm switch info`）
* guest 支持多块磁盘设备（UEFI guest 最多可达 3 个）
* 多网络适配器支持
* PCI 直通
* 可为 guest 指定 UTC 时间
* 可通过配置文件为 guest 启用设备 `virtio-rnd`
* 所有需要 grub 命令的 `grub-bhyve` guest 使用自定义 grub 配置文件。这些 guest 会在控制台显示可访问的启动菜单。
* 对基于 ZFS 的 guest 可进行快照、回滚与克隆
* 可创建基于 ZFS 的 guest 镜像，并据此部署新 guest
* 支持设置 rctl 限制

## 路线图

**Aug 16, 2018**

>**技巧**
>
>在 2025.11.9，Ports 中 [sysutils/vm-bhyve](https://www.freshports.org/sysutils/vm-bhyve) 的版本是 1.6.2。

### 1.3 版本（开发版本）

* [ ] 用户可控制用于直通设备的 slot/function，同时可以配置 vm-bhyve 在自动设备中应从哪个 slot 开始。
* [ ] 通过 `wired_memory="yes"` 控制联动内存（wired memory）的功能。注意，已无法再通过 `bhyve_options="-S"` 实现该目的。`bhyve_options` 仍然存在，但我们不再扫描其中的 `-S` 参数来传递相同选项给 bhyveload。
* [ ] 使用新的 bhyve CPU 语法控制 CPU 拓扑。据我所知，主机需要是 FreeBSD 12。
* [ ] 选项 `graphics_vga=on|off|io`，用于控制 fbuf 参数 `vga`。在 UEFI 下的 OpenBSD 配置中使用此设置。
* [ ] 若安装介质文件名不以 `.iso` 结尾，vm-bhyve 将自动切换为 `ahci-hd`，并将此设备设置为只读。

### 1.2 版本（当前 FreeBSD Ports 版本）

这是当前的开发版本，有以下新特性：

* [ ] 支持在单个控制器上使用高达 32 个 AHCI 设备。此前，bhyve 会为每个设备添加一个新的 AHCI 控制器，这在 UEFI guest 中严重限制了可附加设备的数量。（仅限 FreeBSD 12）
* [ ] 新增命令 `vm set` 与 `vm get` 来管理全局配置变量。目前仅支持变量 `console`，但这为将来更多基本功能的可配置化铺平了道路。
* [ ] “交互模式”支持：在 `tmux` 会话中启动 guest。与普通模式不同，`tmux` 会话不会分离，这提供了类似前台模式的体验；优势在于仍可手动分离 `tmux` 会话，让 guest 在后台继续运行。
* [ ] 命令部分匹配支持，例如可使用 `vm sw l` 列出虚拟交换机。
* [ ] 1.2 默认对所有 guest 使用 UTC 时钟，除非明确禁用。这种行为更一致、更可预期。
* [ ] “media” 数据存储支持。如果你已有存放 ISO 镜像的目录，就没必要把它们复制到 **$vm_dir/.iso**。现在你可以直接把该目录添加为媒体数据存储，我们会在此目录中查找 ISO 文件。运行安装命令时也可直接使用 ISO 的完整路径，我们会在当前目录中查找。
* [ ] 命令 `vm start` 现在能同时启动多个 guest。
* [ ] 重构虚拟交换机。现在桥接以交换机命名，代码模块化。
* [ ] 更好地支持 virtio-console 设备。


### 1.1 版本


* [ ] 新增概念 “datastore”（数据存储）与命令集 `vm datastore`。可创建多个虚拟机存储位置。可在特定数据存储上创建 guest，我们会在所有位置中查找用户指定的 guest。
* [ ] 将“global”（全局）配置移入 **$vm_dir/.config/system.conf**。之前有独立的配置文件 `switch` 和 `datastore`，现已合并。我们添加了新的函数 `__config` 用于加载与访问该文件，无需频繁调用 `sysrc`。
* [ ] 移除了配置项 `guest`。用户需确保为所需的 guest 指定了 `loader="grub"` 或 `loader="bhyveload"`。
* [ ] 支持通过 VNC 实现 UEFI 图形界面。
* [ ] 支持使用 `tmux` 替代 `cu` 作为 guest 的串行控制台。


### 1.0 版本


当前计划是将现有代码库（截至 0.12.5）稳定为 1.0 版本，待确认无明显错误即可发布。我对当前功能很满意。测试了 FreeBSD / Linux / NetBSD / OpenBSD / Windows 多种 guest，运行正常。我尝试设置了在主机启动时自动启动的 guest（5 个），均无问题。

截至 4 月 15 日，当前代码升级为 1.0-beta。我计划保留一到两周，此期间只递增修订号/构建号（现在为独立整数）。之后我将发布 1.0 并提交 Port 更新。其他功能变更将纳入 1.1，并暂时仅在 GitHub 发布。我不希望频繁提交 Port 更新到 FreeBSD；想要最新功能的用户可从 GitHub 获取，其他用户则可在 Ports 获取稳定的 1.0。

该版本分支为 1-0.stable。待确认脱离 beta 状态，Ports 将更新此版本。版本号简化为 1.0，并附加构建号。如需修复，将在此分支完成，并在必要时以 1.0-pX 形式更新 Ports。


### 0.12.5 版本（已完成）


* [ ] guest grub 默认使用分区 1，因此若分区 1 正确（通常如此），配置文件中无需再指定分区选项。
* [ ] 新增 `-f` 前台运行选项，可在前台运行 guest（及安装程序）。这对安装过程非常有用，无需运行 `vm install` 后再连接控制台。

### 0.12 版本（已完成）


* [ ] 新增对 rctl 限制的支持。


### 0.11 版本（已完成）


更新了 `grub-bhyve` 支持，现在使用主机上的自动生成 `grub.cfg` 文件，而不再直接向加载器传递启动命令。这带来以下好处：

* [ ] 加载器始终在 guest 控制台运行，因此若 guest 启动失败，用户可通过 `vm console` 命令访问加载器。
* [ ] 如果用户从控制台重启 guest，加载器会出现，用户可以选择启动默认选项（会自动启动），或进入 grub 命令行。
* [ ] 在旧版本中，若启动命令错误，`grub-bhyve` 可能挂起并占用 100% CPU。此问题解决完成。
* [ ] 配置文件中不再需要 `boot` 命令，简化了配置。


### 0.10 版本（已完成）

完全重写了 `grub-bhyve` 支持。用户可通过选项 `grub_installX` 与 `grub_runX` 完全控制传递给加载器的启动与安装命令。

## 简单开始

**Jul 26, 2022**

安装 vm-bhyve 并启动 FreeBSD guest 所需命令的简单概览：

```sh
1. pkg install vm-bhyve [grub2-bhyve sysutils/bhyve-firmware]
2. zfs create pool/vm
3. sysrc vm_enable="YES"
4. sysrc vm_dir="zfs:pool/vm"
5. vm init
6. cp /usr/local/share/examples/vm-bhyve/* /mountpoint/for/pool/vm/.templates/
7. vm switch create public
8. vm switch add public em0
9. vm iso ftp://ftp.freebsd.org/pub/FreeBSD/releases/ISO-IMAGES/10.3/FreeBSD-10.3-RELEASE-amd64-bootonly.iso
10. vm create myguest
11. vm install [-f] myguest FreeBSD-10.3-RELEASE-amd64-bootonly.iso
12. vm console myguest
```

**第 1 行**

安装 `vm-bhyve`。

如果需要支持 Linux grub，可安装软件包 `grub2-bhyve`；如需运行包括 Windows 在内的 UEFI 启动系统，请安装 UEFI 相关 Port。

**第 2 行**

为虚拟机创建数据集。若不使用 ZFS，只需创建一个普通目录即可。

**第 3–4 行**

在 **/etc/rc.conf** 中启用 `vm-bhyve` 并设置要使用的数据集。如果不使用 ZFS，只需设置 `$vm_dir="/my/vm/folder"`。

**第 5 行**

运行 `vm init` 命令，在 **$vm_dir** 下创建所需目录并加载内核模块。

**第 6 行**

安装随 `vm-bhyve` 提供的示例模板。

**第 7–8 行**

创建虚拟交换机 “public”，并将你的网络接口附加到 “public”。将 `em0` 替换为实际连接网络的接口。

**第 9 行**

从 FTP 站点下载 FreeBSD 镜像。

**第 10–12 行**

使用 FreeBSD 的 `default.conf` 模板创建新的 guest，运行安装程序并连接到其控制台。此时可像普通系统一样进行安装。如果在安装命令前添加选项 `-f`，guest 将直接在你的终端上运行，因此不需要再执行 `vm console` 命令（请注意，选项 `-f` 仅在 `vm-bhyve-0.12.5+` 中可用）。

## 完整示例模板

**Oct 11, 2025**

以下是运行 FreeBSD guest 的完整模板示例。

其他操作系统的配置示例请参见 [Supported Guest Examples](https://github.com/churchers/vm-bhyve/wiki/Supported-Guest-Examples) 页面。

所有模板均存放在目录 **$vm_dir/.templates** 下，文件名应为 `[我的模板名].conf`。你可以创建任意数量的模板，并在创建 guest 时指定，例如：

```sh
vm create -t 我的模板名 myguest
```

若在创建 guest 时未指定模板名称，则默认使用 `default.conf`。

### 模板示例（FreeBSD guest）

```ini
loader="bhyveload"
cpu=2
memory=512M
disk0_type="virtio-blk"
disk0_name="disk0.img"
disk0_dev="file"
network0_type="virtio-net"
network0_switch="public"
```

**第 1 行**

FreeBSD guest 需要使用 `bhyveload` 作为加载器。

**第 2-3 行**

指定该 guest 的 CPU 核心数和内存大小。内存选项可以以 `M` 或 `G` 结尾。

**第 4-6 行**

第一块磁盘的选项，即 guest 启动的磁盘。`disk0_dev` 不是必需的，默认值为 `file`。也可以使用 ZVOL 或自定义磁盘，详情请参考磁盘示例页面。

**第 7-8 行**

第一块网络适配器的选项。如果不需要网络，可以删除这些选项。目前唯一支持的接口类型是 `virtio-net`。该接口会自动连接到 `public` 虚拟交换机（若有）。

## 使用 tmux

**Oct 3, 2022**

在默认情况下，`vm-bhyve` 使用 `cu`/`nmdm` 作为控制台会话管理工具。如果你希望虚拟机运行在 `tmux` 会话中，从 v1.1+ 开始支持此功能。只需在 **$vm_dir/.config/system.conf** 中设置选项 `console`：

```sh
vm set console=tmux
```

设置完成后，启动的所有新 guest 都会关联到一个 tmux 会话。会话名称与 guest 名称相同，同时也可以使用标准命令 `vm console guest` 连接到该 tmux 会话。

示例：

```sh
# vm list | grep fbsd
fbsd            default         bhyveload   2      256M      -                    No           Running (88761)
# tmux ls
fbsd: 1 windows (created Tue Jun 28 10:42:22 2016) [168x46]
```

## 受支持的 Guest 示例

该文件列出了测试过的 guest，并展示了安装器和 guest 启动时可使用的配置选项。目标是建立配置清单，供大家在自己的模板中选用。在大多数情况下，guest 都是使用其默认安装选项进行测试的。

如果你成功使用了这里未列出的 guest 操作系统，请告知我，以便更新列表。

请注意，我在这些示例中都省略了网络和磁盘设置，因为所有 guest 的格式都是相同的。不同配置请参见网络或磁盘示例页面。

所有 guest 都需要 cpu 和内存选项，非常简单：

```ini
cpu=1
memory=512M
```

如果你使用的是 `vm-bhyve-0.11` 及更新版本，则 grub guest 不再需要 `boot` 命令，因此安装和运行时的 guest 配置文件中可以去掉该选项。

在 `vm-bhyve-0.12.5` 中，如果分区 1（msdos1/openbsd1）正确，任何 `grub-bhyve` guest 都不需要选项 `grub_run_partition`。

完整的可用配置选项列表及说明，请参见手册页或 sample-templates 目录下的 `config.sample`。

### FreeBSD / pfSense / MidnightBSD

该配置文件应支持所有 FreeBSD 和 MidnightBSD 版本。对于 pfSense，我使用了嵌入式内核选项，效果良好。

```ini
guest="freebsd"
loader="bhyveload"
```

### Windows

建议至少为 Windows 准备 1G 内存，磁盘仿真需要使用 `ahci-hd`。

```ini
guest="windows"
uefi="yes"
disk0_type="ahci-hd"
# 若低于 Windows 10，则需要设置 sectorsize
disk0_opts="sectorsize=512"
```

### NetBSD

```ini
guest="generic"
loader="grub"
grub_install0="knetbsd -h -r cd0a /netbsd"
grub_install1="boot"
grub_run0="knetbsd -h -r ld0a (hd0,msdos1)/netbsd"
grub_run1="boot"
```

### OpenBSD

以下示例针对 OpenBSD 6.2 amd64，因为安装命令有版本号和架构。我成功运行了 5.2，其他版本只要更新安装命令中的版本号也应可用。i386 guest 同样可用，但需更新安装命令。

```ini
loader="grub"
cpu=1
memory=256M
network0_type="virtio-net"
network0_switch="public"
disk0_type="virtio-blk"
disk0_name="disk0.img"
grub_install0="kopenbsd -h com0 /6.2/amd64/bsd.rd"
grub_run0="kopenbsd -h com0 -r sd0a /bsd"
bhyve_options="-w"
```

### resflash

resflash 是用于 U 盘的 OpenBSD 镜像，也适合作为虚拟网络设备。以下示例为 resflash 20171202_1952 amd64。

```ini
loader="grub"
cpu=1
memory=256M
network0_type="virtio-net"
network0_switch="public"
disk0_type="virtio-blk"
disk0_name="disk0.img"
grub_run_partition="openbsd4"
grub_run0="kopenbsd -h com0 -r sd0d /bsd"
bhyve_options="-w"
```

启动命令如下：

```sh
vm create -t resflash resflash-vm
vm configure resflash-vm
cd /my/vm/resflash-vm
fetch https://stable.rcesoftware.com/pub/resflash/6.2/amd64/install/resflash-amd64-com0-115200-20171202_1952.img.gz
gunzip resflash-amd64-com0-115200-20171202_1952.img.gz
mv resflash-amd64-com0-115200-20171202_1952.img disk0.img
vm -f start resflash-vm
```

### OpenBSD（UEFI）

UEFI 模式下安装 OpenBSD 并使用 VNC 的模板：

```ini
loader="uefi"
uefi_vars="yes"
cpu=4
memory=8G
network0_type="virtio-net"
network0_switch="public"
disk0_type="virtio-blk"
disk0_name="disk0.img"
bhyve_options="-w"
graphics="yes"
xhci_mouse="yes"
graphics_res="1920x1080"
```

ZVOL 示例：

```ini
loader="uefi"
cpu=4
memory=4G
network0_type="virtio-net"
network0_switch="public"
disk0_name="disk0"
disk0_type="virtio-blk"
disk0_dev="sparse-zvol"
disk0_size="1T"
#bhyve_options="-w"
graphics="yes"
xhci_mouse="yes"
graphics_res="1920x1080"
```

OpenBSD UEFI 注意事项：

* 安装器默认会选择错误的根磁盘，请确保选择了正确的磁盘（sd1）。
* 安装完成后，请执行 `vm poweroff openbsd-vm`，否则 VM 会重新启动到安装盘。
* 对于 UEFI，安装盘需要的格式是 `.img`，而不能是 `.iso`。
* 使用命令：`# vm iso https://cdn.openbsd.org/pub/OpenBSD/7.6/amd64/install76.img`

### Alpine Linux

Alpine 的特殊之处在于安装 CD 上的文件针对不同的 Alpine 版本有不同的后缀。基本文件名为 `vmlinuz-[后缀]` 和 `initramfs-[后缀]`。请相应调整你的 vm `.conf` 文件：

| Alpine 版本 | 后缀      |
| :---------: | :-------: |
| standard  | vanilla |
| virtual   | virt    |

Alpine Virtual 模板：

```ini
loader="grub"
cpu=1
memory=512M
network0_type="virtio-net"
network0_switch="public"
disk0_type="virtio-blk"
disk0_name="disk0.img"
grub_install0="linux /boot/vmlinuz-virt initrd=/boot/initramfs-virt alpine_dev=cdrom:iso9660 modules=loop,squashfs,sd-mod,usb-storage,sr-mod"
grub_install1="initrd /boot/initramfs-virt"
grub_run0="linux /boot/vmlinuz-virt root=/dev/vda3 modules=ext4"
grub_run1="initrd /boot/initramfs-virt"
```

兼容模板：

```ini
guest="linux"
loader="grub"
grub_install0="linux /boot/grsec initrd=/boot/initramfs-grsec alpine_dev=cdrom:iso9660 modules=loop,squashfs,sd-mod,usb-storage,sr-mod"
grub_install1="initrd /boot/initramfs-grsec"
grub_install2="boot"
grub_run_partition="msdos1"
grub_run0="linux /boot/vmlinuz-grsec root=/dev/vda3 modules=ext4"
grub_run1="initrd /boot/initramfs-grsec"
grub_run2="boot"
```

### CentOS 6 (LVM)

请注意，CentOS 的 kernel 和 initramfs 文件带内核版本号，如果系统没有内核 `2.6.32-573.el6`，需要修改配置。

如果你熟悉 Linux，可以在 guest 中安装 grub2，并配置 `vm-bhyve` 使用 grub 配置文件。默认情况下，`vm-bhyve` 会查找 **/boot/grub/grub.cfg**，也可通过 `grub_run_dir` 和 `grub_run_file` 配置修改。（参见下文 CentOS 7 使用 guest 中 **/grub2/grub.cfg** 启动的示例）

```ini
guest="linux"
loader="grub"
grub_install0="linux /isolinux/vmlinuz"
grub_install1="initrd /isolinux/initrd.img"
grub_install2="boot"
grub_run_partition="msdos1"
grub_run0="linux /vmlinuz-2.6.32-573.el6.x86_64 root=/dev/mapper/VolGroup-lv_root"
grub_run1="initrd /initramfs-2.6.32-573.el6.x86_64.img"
grub_run2="boot"
```

### CentOS 7

```ini
guest="linux"
loader="grub"
grub_install0="linux /isolinux/vmlinuz"
grub_install1="initrd /isolinux/initrd.img"
grub_install2="boot"
grub_run_partition="msdos1"
grub_run_dir="/grub2"
```

### RHEL 6.5 (LVM)

基于内核 `2.6.32-431.el6.x86_64`。如果使用了其他版本内核，请相应修改配置。如果不确定内核版本，可在不指定 grub 命令的情况下启动 guest，然后使用 `vm console` 访问 bootloader，在 bootloader 中查找正确内核（`ls (hd0,1)/`）。

退出 bootloader，可手动输入正确启动命令或输入 `exit`。如常规操作，使用 `~ + Ctrl+D` 退出控制台。

```ini
guest="linux"
loader="grub"
grub_install0="linux /isolinux/vmlinuz"
grub_install1="initrd /isolinux/initrd.img"
grub_install2="boot"
grub_run_partition="msdos1"
grub_run0="linux /vmlinuz-2.6.32-431.el6.x86_64 root=/dev/mapper/VolGroup-lv_root"
grub_run1="initrd /initramfs-2.6.32-431.el6.x86_64.img"
grub_run2="boot"
```

### Debian & Ubuntu (Grub)

```ini
guest="linux"
loader="grub"
grub_run_partition="msdos1"
```

如果需要 Ubuntu 14.04，还需添加以下内容，否则会卡在 grub 提示符：

```ini
grub_run_dir="/grub"
grub_run_file="grub.cfg"
```

### Debian UEFI

```ini
loader="uefi"
cpu=2
memory=2G
network0_type="virtio-net"
network0_switch="public"
disk0_type="ahci-hd"
disk0_name="disk0.img"
disk0_size="10G"
graphics="yes"
xhci_mouse="yes"
graphics_res="1024x768"
graphics_wait="yes"
uefi_vars="yes"
```

### Ubuntu UEFI

类似 Debian，不使用 efivars，使用 zvol。

```ini
loader="uefi"
cpu=4
memory=8G
network0_type="virtio-net"
network0_switch="public"
graphics="yes"
xhci_mouse="yes"
graphics_res="1920x1080"
zfs_zvol_opts="volblocksize=128k"
disk0_name="disk0"
disk0_dev="sparse-zvol"
disk0_type="virtio-blk"
disk0_size="1T"
```

### VyOS

需要设置磁盘扇区大小，否则安装器会报错。

```ini
loader="grub"
disk0_opts="sectorsize=512"
grub_install0="linux /live/vmlinuz console=ttyS0 console=tty0 quiet initrd=/live/initrd.img boot=live nopersistent noautologin nonetworking nouser hostname=vyos"
grub_install1="initrd /live/initrd.img"
grub_install2="boot"
```

### RouterOS (CHR)

根据 [讨论](https://forum.mikrotik.com/viewtopic.php?t=135648)，可以在 bhyve 中运行 RouterOS 6.x 的 CHR。示例配置如下：

```ini
loader="grub"
cpu=2
memory="4G"
network0_type="virtio-net"
network0_switch="wan"
disk0_type="virtio-blk"
disk0_name="chr-6.44.5.img"
grub_run0="linux /boot/vmlinuz-64 crashkernel=16M"
grub_run1="initrd /boot/initrd.rgz"
```

修改 MikroTik 官网下载的镜像后，可以按照 [教程](https://it-notes.dragas.net/2023/03/21/creating-a-mikrotik-chr-routeros-7-bhyve-vm-in-freebsd-2/) 正确运行基于 RouterOS 7.x 的 CHR。

### Gentoo

使用 **/boot** 替代 **/isolinux**（参见 [issue 186](https://github.com/churchers/vm-bhyve/issues/186)）。

安装模板：

```ini
loader="grub"
cpu=1
memory=256M
grub_install0="linux /boot/gentoo root=/dev/ram0 init=/linuxrc  dokeymap looptype=squashfs loop=/image.squashfs  cdroot"
grub_install1="initrd /boot/gentoo.igz"
network0_type="virtio-net"
network0_switch="public"
disk0_type="virtio-blk"
disk0_name="disk0.img"
```

安装完成后的可用配置：

```ini
loader="grub"
cpu=1
memory=1024M
network0_type="virtio-net"
network0_switch="public"
disk0_type="virtio-blk"
disk0_name="disk0.img"
network0_mac="58:9c:fc:0e:4b:da"
grub_run_partition="2"
grub_run_dir="/grub"
```

### OpenWRT

可以使用固件 uefi-csm 运行 OpenWRT x86/64。

下面是个配置示例。只需下载最新的磁盘镜像并放入 vm 目录。

启动 VM 后，可以通过 `vm console VM` 访问控制台。

```ini
loader="uefi-csm"
cpu=2
memory=512M
network0_type="e1000"
network0_switch="public"
disk0_type="ahci-hd"
disk0_name="openwrt-19.07.1-x86-64-combined-ext4.img"
```

### OpenWRT UEFI

以下配置在 `uefi` 下使用 `virtio` 驱动，并为 LAN 创建了额外的 `switch`。

* 使用命令 `# vm switch create openwrt` 创建额外 switch。

配置文件中 `network1` 的 switch 设置为 `public`，因为 OpenWRT 会自动使用 `eth1` 作为 WAN 并通过 DHCP 获取 IP。

OpenWRT 也会自动将 LAN 子网设置为 `192.168.1.0/24`（eth0），如果 WAN 和 LAN 子网相同，会导致路由表冲突，因此需要修改 LAN 子网。

* 下载 x86/64 ext4 efi 镜像，用 gzip 解压，然后复制到 VM 目录，并重命名为 `disk0.img`，替换已创建的 `disk0.img`。

```sh
# vm iso https://downloads.openwrt.org/releases/24.10.0/targets/x86/64/openwrt-24.10.0-x86-64-generic-ext4-combined-efi.img.gz
```

配置示例：

```ini
loader="uefi"
graphics="yes"
xhci_mouse="yes"
graphics_res="1920x1080"
cpu=2
memory=512M
disk0_type="virtio-blk"
disk0_name="disk0.img"
debug="yes"
network0_type="virtio-net"
network0_switch="openwrt"
network1_type="virtio-net"
network1_switch="public"
```

## 虚拟磁盘

**Oct 19, 2021**

当从模板创建 guest 时，会为模板中指定的每个磁盘创建一块磁盘镜像。如果希望在 guest 创建后添加磁盘，可以使用命令 `vm add`，或者先创建磁盘镜像，再手动更新配置文件。

下面是一些支持的磁盘配置示例：

### 普通稀疏文件

```ini
disk0_type="virtio-blk"
disk0_name="disk0.img"
```

### 稀疏 ZVOL

如果不想使用稀疏 ZVOL，也可以通过指定 `disk0_dev="zvol"` 来支持非稀疏 ZVOL：

```ini
disk0_type="virtio-blk"
disk0_name="disk0"
disk0_dev="sparse-zvol"
```

### 自定义磁盘

可自定义磁盘镜像的路径。磁盘可以是稀疏文件、ZVOL，甚至是 **/dev/** 下的物理磁盘：

```ini
disk0_type="virtio-blk"
disk0_name="/dev/ada10"
disk0_dev="custom"
```

### 带选项的简单稀疏文件

```ini
disk0_type="virtio-blk"
disk0_name="disk0.img"
disk0_opts="nocache,direct"
```

### 自定义 ZVOL 磁盘

如果希望为已有 guest 添加额外的 ZVOL 磁盘，可按以下步骤操作：

首先为磁盘镜像创建新的 ZVOL。如果不希望 ZVOL 是稀疏的，请去掉选项 `-s`：

```sh
# zfs create -sV 50G -o volmode=dev path/to/dataset/zvol
```

然后将其作为额外磁盘添加到 guest 配置文件中：

```sh
# vm configure myguest

disk1_name="/dev/zvol/path/to/dataset/zvol"
disk1_type="virtio-blk"
disk1_dev="custom"
```

### VirtIO 9p 文件共享

以下示例展示了如何使用 VirtIO 9p 与 VM 共享文件或文件夹：

首先，将以下内容添加到 guest 配置文件中：

```sh
# vm configure myguest

disk1_type="virtio-9p"
disk1_name="sharename=/path/to/share"
disk1_dev="custom"
```

另外，可在 `disk1_name` 后追加 `,ro` 来设置只读。

在 FreeBSD guest 上，可使用以下命令挂载 9p 共享：

```sh
# mount -t 9p -o trans=virtio sharename /local/mount/point/of/share
```

持久化配置可通过 fstab(5) 完成：

```sh
sharename          /local/mount/point/of/share    9p      trans=virtio,rw 0 0
```

## 虚拟网卡

**Mar 2, 2020**

一些可能的网络配置示例。

所有 guest 都支持多个网络接口。

### 基本示例，连接到虚拟交换机“public”

```ini
network0_type="virtio-net"
network0_switch="public"
```

### 自定义 MAC 地址

请注意，首次运行时，如果网络接口没有 MAC 地址，`vm-bhyve` 将自动为其分配静态 MAC 地址，并写入配置文件。

```ini
network0_type="virtio-net"
network0_switch="public"
network0_mac="00:11:22:33:44:55"
```

`bhyve` 仅接受以 `58:9c:fc` 开头的自定义 MAC 地址，否则将无法启动（无论 guest OS 是什么），且不会给出有用的错误信息。

### 自定义网络设备

默认情况下，`vm-bhyve` 会为每个接口创建动态 tap 设备。如果需要进行复杂配置，需要手动设置网络，这可能会成为问题，因为设备在启动 guest 之前并不存在。

通过指定自定义设备，你可以在 `rc.conf` 自行配置该设备。请注意，我省略了配置选项 `switch`。如果指定了，tap 设备会自动附加到虚拟交换机。如果你按需求配置了自定义设备，可以省略 switch 设置。

```sh
network0_type="virtio-net"
network0_device="tap0"
```

## 数据存储

**Jul 16, 2016**

vm-bhyve 1.1 现在支持多个数据存储（datastore），并新增了一组用于管理数据存储的命令。

基本配置与以前相同。你需要为 `vm-bhyve` 的配置和 guest 创建一个目录或 ZFS 数据集，并在 **/etc/rc.conf** 中设置：

```ini
vm_dir="zfs:sys/data/vm"
```

在运行 `vm-bhyve` 时，该目录用于 ISO 文件和核心配置数据，同时也成为虚拟机的默认数据存储（datastore）。如果列出数据存储，将显示如下信息：

```sh
# vm datastore list
NAME            TYPE        PATH                      ZFS DATASET
default         zfs         /data/vm                  sys/data/vm
```

这里显示了数据存储的名称、类型和文件系统路径。对于 ZFS 存储，还会显示数据集名称。

可以使用以下命令添加新的数据存储：

```sh
# vm datastore add <名称> <存储规范>
```

* 名称必须为 16 个字符以内的字母、数字、`-` 或 `.`。
* 数据存储规范（spec）可以是目录路径或 `zfs:pool/dataset`。
* 注意：系统不会自动创建目录或数据集，需要提前创建。

示例：

```sh
# vm datastore add ssd zfs:sys/data/vm2
# vm datastore list
NAME            TYPE        PATH                      ZFS DATASET
default         zfs         /data/vm                  sys/data/vm
ssd             zfs         /data/vm2                 sys/data/vm2
```

要移除数据存储，可使用：

```sh
# vm datastore remove <名称>
```

该命令只会从配置中移除数据存储，数据本身不会被删除。

创建虚拟机时，默认会使用默认数据存储。但可以通过选项 `-d` 指定其他数据存储：

```sh
# vm create -d ssd -t centos7 -s 50G centos-ssd
```

命令 `list` 和 `info` 也更新完成，能显示数据存储信息：

```sh
# vm list
NAME            DATASTORE       LOADER      CPU    MEMORY    AUTOSTART    STATE
alpine          default         grub        1      512M      No           Stopped
centos          default         grub        1      512M      Yes [1]      Stopped
wintest         default         none        2      2G        No           Running (15087)
fb2             ssd             bhyveload   1      256M      No           Stopped

# vm info fb2
------------------------
Virtual Machine: fb2
------------------------
  state: stopped
  datastore: ssd
....
```

请注意，guest 名称仍需在所有数据存储中唯一。如果有多个相同名称的 guest（例如添加包含 guest 的数据存储），`vm-bhyve` 命令会按添加顺序搜索数据存储，仅操作找到的第一个 guest。

## 虚拟交换机

**Jun 29, 2020**

`vm-bhyve` 的设计理念是基于“虚拟交换机”。你可以创建任意数量的虚拟交换机，每个交换机都有简单的名称。

在配置 guest 时，你可以为每个网络接口指定要连接的交换机名称。当 guest 启动时，会创建虚拟以太网（`tap`）设备，并自动将其添加到指定的虚拟交换机中。

在后台，每个虚拟交换机对应系统上的简单 `bridge` 接口。`vm-bhyve` 会跟踪每个交换机对应的 bridge（使用 `ifconfig` 的 groups 功能），并利用此信息将每个 tap 接口连接到正确的 bridge。如果交换机名称少于 12 个字符，则接口命名为 `vm-{name}`，这在配置其他工具（如 pf/dnsmasq）时非常有用。

### 查看虚拟交换机信息

使用 `list` 命令可以查看交换机列表及基本信息：

```sh
# vm switch list
NAME    TYPE      IFACE      ADDRESS         PRIVATE  MTU  VLAN  PORTS
public  standard  vm-public  192.168.8.1/24  no       -    -     -
```

使用 `info` 命令可以查看更详细信息及连接的 guest：

```sh
# vm switch info public
------------------------
Virtual Switch: public
------------------------
  type: auto
  ident: bridge0
  vlan: -
  nat: -
  physical-ports: re0
  bytes-in: 385868007 (367.992M)
  bytes-out: 401540470 (382.938M)

  virtual-port
    device: tap1
    vm: wintest
```

### 创建一个简单的交换机 `public`

在示例模板中，每个 guest 都有一个接口连接到虚拟交换机 `public`。使用此名称并无特殊原因，你可以为交换机取任意名字：

```sh
# vm switch create public
# vm switch add public em0
```

上述命令创建了虚拟交换机，并将物理接口 `em0` 连接到它。如果 `em0` 能访问互联网，那么连接到该交换机的 guest 也能访问互联网。

### 分配 VLAN 号

为虚拟交换机分配 VLAN 号，会为连接的每个物理接口创建 `vlan` 接口，并将其连接到交换机，而非真实接口。这样，交换机上的 guest 可以正常通信，经过物理接口的数据包会带上指定的 VLAN 标签：

```sh
# vm switch vlan public 10
```

要移除 VLAN，将其设置为 `0`：

```sh
# vm switch vlan public 0
```

### 使用自定义 bridge

有时你可能希望手动配置 `bridge` 接口，以使用 `vm-bhyve` 不直接支持的功能。在这种情况下，可以先在 **/etc/rc.conf** 手动创建 bridge，然后导入到 `vm-bhyve`：

```sh
# vm switch create -t manual -b bridge0 customswitch
```

该命令假定你手动创建了 `bridge0`。运行后，`vm-bhyve` 会为 bridge 接口分配描述。如果 guest 配置中指定 `networkX_switch="customswitch"`，该接口将连接到你的自定义 bridge。

### NAT

有关 NAT 配置详情，请参见 Wiki 页面：

[https://github.com/churchers/vm-bhyve/wiki/NAT-Configuration](https://github.com/churchers/vm-bhyve/wiki/NAT-Configuration)

### 故障排除

如果无法删除虚拟交换机，请检查该交换机是否有对应的 bridge。通常格式应为 `vm-<switch_name>`，或者如果名称包含 `-`，则为 `bridge<#>`。如果不符合，需要先删除相关虚拟交换机配置，再重新创建交换机。

参考解决方案：[https://github.com/churchers/vm-bhyve/issues/373](https://github.com/churchers/vm-bhyve/issues/373)

## NAT 配置

**Oct 18, 2018**

不幸的是，从 v1.2 起，`vm-bhyve` 移除了内部 NAT 配置。由于是 shell 脚本，我们之前依赖配置外部系统（如 pf 和 dnsmasq）来提供 NAT 功能。有些用户想使用其他工具或防火墙，并且许多用户因已有的 pf 或 dnsmasq 配置导致 NAT 无法正常工作。现在的情况是，手动配置 NAT 比通过 `vm-bhyve` 启用 NAT 再手动安装或调整生成的配置更简单，出错率也更低。

下面是使用 pf 为 `vm-bhyve` guests 设置 NAT 主机的一些基础指南。

### 主机配置

在此示例中，我选择为 guest 使用 `192.168.8.x` 网络，网关为 `192.168.8.1`。

我们需要启用网关功能，使主机能在公共网络和 NAT 网络之间转发数据包。同时需要启动 pf 服务，在 **/etc/rc.conf** 中配置如下：

```ini
gateway_enable="yes"
pf_enable="yes"
```

接下来在 **/etc/pf.conf** 中创建 NAT 规则，将私有网络的数据包进行 NAT。此示例中，`em0` 是主机面向公网的接口：

```sh
nat on em0 from {192.168.8.0/24} to any -> (em0)
```

此时可以重启主机，或者如果愿意，也可以手动使用 `sysctl` 启动 pf 并启用转发：

```sh
# sysctl net.inet.ip.forwarding=1
# service pf start
```

### vm-bhyve 配置

现在需要创建交换机，用于连接 guest。应为该交换机分配 NAT 网络范围内的 IP，用作 guest 的网关。我将其命名为 `public`，这是所有 `vm-bhyve` 示例中使用的默认交换机名称，你也可以根据环境自定义名称：

```sh
# vm switch create -a 192.168.8.1/24 public     <-- 如果是新建交换机
# vm switch address public 192.168.8.1/24       <-- 如果已有交换机可直接使用
# vm switch list
NAME    TYPE      IDENT  ADDRESS         PRIVATE  MTU  VLAN  PORTS
public  standard  -      192.168.8.1/24  no       -    -     -
```

此时，你可以配置 guest 使用该虚拟交换机并启动：

Guest 配置示例：

```ini
network0_type="virtio-net"
network0_switch="public"
```

在 guest 中测试本地和外部网络连接：

```sh
root@test:~ # cat /etc/rc.conf
...
ifconfig_vtnet0="192.168.8.2/24"
defaultrouter="192.168.8.1"
root@test:~ # ping 192.168.8.1
64 bytes from 192.168.8.1: icmp_seq=0 ttl=64 time=0.135 ms
root@test:~ # ping 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=0 ttl=54 time=12.199 ms
```

### DHCP / dnsmasq

为 guest 提供 DHCP 服务可以使用 `dnsmasq`（需通过 Ports 安装）。基于上述虚拟交换机 `public` 和 `192.168.8.x` 网络范围，简单配置示例如下：

```ini
port=0
domain-needed
no-resolv
except-interface=lo0
bind-interfaces
local-service
dhcp-authoritative

interface=vm-public
dhcp-range=192.168.8.10,192.168.8.254
```

这样，guest 就可以通过虚拟交换机获得 IP 并通过主机 NAT 上网。

## 为 Guest 配置 Grub

**Jun 22, 2018**

`vm-bhyve` 内置支持软件包 `grub2-bhyve`，它能够启动大多数可由 GRUB2 启动的 guest。本节介绍可在 guest 配置文件中设置的各种配置选项，以控制 `grub2-bhyve` 及其效果。

要让 guest 使用 `grub2-bhyve` 加载器，需要在 guest 配置文件中设置 `loader="grub"`。

Grub 加载器运行在 guest 控制台上，因此当 guest 处于 `Bootloader` 状态时，可以使用标准命令 `vm console guest` 访问。如果以前台或交互模式（仅 tmux）启动 guest，一启动就会直接进入 grub 加载器。

### 无配置

如果完全未指定 grub 配置，`grub2-bhyve` 会在 guest 的 `hd0` 分区 `1` 上查找 **/boot/grub/grub.cfg** 文件。如果文件存在，会加载该文件，大多数情况下会显示带有选项的启动菜单。

如果未找到 GRUB2 配置文件，grub2-bhyve 会直接进入 `grub>` 命令行界面。

### guest 中 grub.cfg 文件位于非标准位置

如果 guest 有 GRUB2 配置文件，但不在 `grub2-bhyve` 查找的标准位置，可在 guest 配置文件中使用 `grub_run_dir` 和 `grub_run_file` 指定文件位置。

例如，CentOS 7 使用 **/grub2/grub.cfg**，文件名未变，仅需指定新目录：

```sh
grub_run_dir="/grub2"
```

若 guest 系统文件名为 **/grub2/grub2.conf**，则可指定：

```sh
grub_run_dir="/grub2"
grub_run_file="grub2.conf"
```

### grub.cfg 位于不同分区

在默认情况下，vm-bhyve 将 guest 磁盘分区的根指定为 `hd0,1`。如果启动分区不是 `1`，可在配置文件中指定其他分区：

```sh
grub_run_partition="2"
```

此配置下，`grub2-bhyve` 会在 `(hd0,2)/boot/grub/grub.cfg` 查找 grub 配置文件。分区设置也会影响自定义命令，例如：

```sh
grub_run_partition="2"
grub_run0="linux /vmlinuz"
```

等同于在 `grub>` 命令行运行 `linux (hd0,2)/vmlinuz`。

### guest 无 GRUB2，使用自定义启动命令

如果 guest 没有 GRUB2 配置文件，启动时 `grub2-bhyve` 会进入 `grub>` 命令行。此时可连接 guest 并输入启动命令，例如启动 NetBSD：

```sh
grub> knetbsd -h -r ld0a /netbsd
grub> boot
```

为了避免每次启动都手动输入，可在 guest 配置文件中指定启动命令（无需 `boot`）：

```ini
grub_run0="knetbsd -h -r ld0a /netbsd"
```

如果需要多条命令，可增加行并依次编号为 `grub_run1`、`grub_run2` 等。在指定命令后，vm-bhyve 会生成自定义的 grub.cfg 文件。当启动 guest 时，将显示包含单个 `guestname (bhyve run)` 选项的自定义 grub 菜单，该选项默认 3 秒后选择（可用 `loader_timeout="X"` 配置更改超时）。

对于大多数 Linux guest，最好在 guest 内安装 GRUB2，以便 guest 自行管理配置文件。内核升级可能需要修改启动命令，如果手动写入 guest 配置文件，未更新命令将导致无法启动。

### 安装器命令

在正常情况下，`grub2-bhyve` 会在 CD 的 **/boot/grub/grub.cfg** 查找 grub 配置文件，如找不到，则进入 `grub>` CLI。我们未提供配置选项指定其他位置，但提供了自定义启动命令的功能。例如 CentOS 6 样例模板的 ISO 安装命令如下：

```ini
grub_install0="linux /isolinux/vmlinuz"
grub_install1="initrd /isolinux/initrd.img"
```

## 运行 Windows

**Sep 3, 2020**

如果想运行 Windows，强烈建议使用 FreeBSD 11 及以上版本，并启用 UEFI 图形支持。

首先需要确保系统已安装 UEFI 固件（如果尚未安装）：

```sh
# pkg install bhyve-firmware
```

### Guest 配置

使用 Windows 模板创建 guest。默认配置为 2 个 CPU 和 2GB 内存，同时使用 Windows 原生支持的 Intel 网络适配器 `e1000`。如果需要修改配置选项，可使用命令 `vm configure guest`。

> **注意**
>
> 如果运行的是 Windows 10 以前的版本，需要使用 `disk0_opts="sectorsize=512"` 将磁盘扇区大小设置为 512。安装 Microsoft SQL Server 时也必须将扇区大小设置为 512。

```sh
# vm create -t windows winguest
```

### 安装

通过指定 Windows ISO 文件正常启动安装。在安装模式下运行时，`vm-bhyve` 将等待 VNC 客户端，在 VNC 连接后再启动 guest，以便捕捉 Windows 可能显示的 “Boot from CD/DVD”（从 CD/DVD 启动）选项。在此期间，`vm list` 中显示 guest 为锁定状态。

```sh
# vm install winguest Windows-Installer.iso
```

连接 VNC 客户端后，按常规方式完成 Windows 安装。

### 添加 VirtIO 网络驱动

虽然网络适配器 `e1000` 能开箱即用，提供网络访问，但建议尽量使用设备 `virtio-net`。安装驱动的方法有几种：

* 如果 guest 可以通过 e1000 上网，可直接在 guest 内下载并安装 VirtIO 驱动，安装完成后关闭 guest，更改 guest 配置为 VirtIO 设备并重启。
* 在安装模式下启动 guest，并指定 VirtIO ISO 文件：

```sh
# vm install winguest virtio-installer.iso
```

* 也可以给 guest 添加 CD 设备，并指向 ISO 文件：

```ini
disk1_type="ahci-cd"
disk1_dev="custom"
disk1_name="/full/path/to/virtio-installer.iso"
```

### CPU 设置

部分 Windows 版本（多数桌面版）不支持一个以上的物理 CPU。在默认情况下，bhyve 会将每个虚拟 CPU 配置为独立 package。

可通过修改 sysctl `hw.vmm.topology.cores_per_package` 来创建每个 CPU 的多个核心而不是多个 package。例如，将其设置为 4，将使 8 vCPU 的 guest 配置为 2 x 4 核 package。

必须在 **/boot/loader.conf** 中设置该 sysctl，重启生效。

在 FreeBSD 12，使用 vm-bhyve 1.3，可通过配置文件为每个 guest 控制 CPU 拓扑：

```ini
cpu=8
cpu_sockets=2
cpu_cores=4
cpu_threads=1
```

### NVMe 支持

从 FreeBSD 12.1R 起，bhyve 支持 NVMe 仿真。在 vm-bhyve 配置中，可使用以下选项：

```ini
disk0_type="nvme"
disk0_name="disk0.img"
disk0_opts="maxq=16,qsz=8,ioslots=1,sectsz=512,ser=ABCDEFGH"
```

甚至可以将 guest 安装到 NVMe 物理磁盘，无需虚拟磁盘。例如：

```ini
loader="uefi"
graphics="yes"
xhci_mouse="yes"
cpu=2
memory=8G
network0_type="e1000"
network0_switch="public"
utctime="no"
passthru0="4/0/0"
```

其中 `4/0/0` 为直通 NVMe SSD。

目前 NVMe 可启动 Windows 8.1 及更新版本。如果要从 NVMe 磁盘启动 Windows 7，请按以下步骤操作：

1. 使用 ahci-hd 控制器安装 Windows 7 guest，按常规流程进行。
2. 安装后，添加额外的 disk1.img 并使用 NVMe 控制器。
3. 安装 Microsoft NVMe 补丁 `Windows6.1-KB2990941-v3-x64.msu` 和 `Windows6.1-KB3087873-v2-x64.msu`，确保 Windows 7 guest 设备管理器中出现 NVMe 控制器和 disk1。
4. 关闭 guest，在配置中交换 disk0.img 和 disk1.img，然后重启。
5. 关闭 guest，移除 AHCI 控制器和 disk1.img，仅保留 NVMe 控制器与 disk0.img，然后再次启动。此时 Windows 7 guest 的设备管理器中将只显示 NVMe 控制器，不再有 AHCI 控制器。


## 运行 OmniOS

**Jan 26, 2022**

OmniOS 的开发在 OmniOSce 中延续，由 OmniOS 社区和基金会提供支持。现在不再需要修改安装介质，这在旧版本的 OmniOS 中是必须的（可查看此页面的旧版本）。

### 下载最新版本

大约每六个月发布一个稳定版本，并提供一年支持。建议访问官方 [OmniOSce 下载页面](https://omniosce.org/download.html)。

```sh
vm iso https://downloads.omniosce.org/media/stable/omniosce-r151028b.iso
```

### 开始安装

```sh
vm create omniosce
```

OmniOSce 支持 `virtio-net` 驱动，相比 `e1000` 性能更佳：

```ini
loader="uefi"
graphics="yes"

network0_type="virtio-net"
```

启动安装，并按照安装程序或 [OmniOSce 安装教程](https://omniosce.org/setup/freshinstall.html) 进行操作：

```sh
vm install omniosce omnios-r151040g.iso
```

使用 VNC 查看器连接，跟随安装教程操作：

```sh
0.0.0.0:5900
```

### 解决 rpool 创建失败

当选择磁盘并进入 ZFS Pool 选项时，需要将扇区大小设置为 4K。

### 解决 “internal error: Argument out of domain”

在使用基于文件的磁盘镜像安装 OmniOS 时可能出现如下错误（2020 年 2 月）：

```sh
Creating root pool with: zpool create -f rpool   c1t0d0
internal error: Argument out of domain
```

解决方法是使用不同的 `ashift` 参数创建池：

* 选择 `3  Shell (for manual rpool creation)`
* 创建池：`zpool create -o ashift=12 rpool  c1t0d0`
* 返回安装菜单：`exit`
* 继续安装：`2  Install OmniOS straight on to a preconfigured rpool`

[illumos bug tracker](https://www.illumos.org/issues/12237)
[待定修复](https://code.illumos.org/c/illumos-gate/+/347/1/usr/src/lib/libzfs/common/libzfs_pool.c)

### 配置 OmniOSce 串行控制台

在最新版本，不再必须使用串行控制台。你可以通过 UEFI loader 使用 VNC 查看器。不过，如果你想用串行控制台，可以按如下方式启用：

安装程序的最后一步可通过 shell 提示符来自定义安装，使用菜单项：**Shell (for post-install ops on /mnt)**。

![OmniOSce 安装器 postmenu](https://omniosce.org/assets/images/install/r26/postmenu.png?raw=true)

第一行配置 loader 仅使用串行控制台，其余命令将默认 OS 控制台切换到 ttya：

```sh
echo -h > /mnt/boot/config
```

```sh
cat << EOM > /mnt/boot/conf.d/serial
console="ttya,text"
os_console="ttya"
ttya-mode="115200,8,n,1,-"
EOM
```


## 运行 Linux：以 Alpine Linux 为例

**Mar 27, 2021**

我花了一些时间才弄明白这个过程，所以为了帮助其他人（包括未来的自己），我写了这份指南。

首先，按照 [https://github.com/churchers/vm-bhyve](https://github.com/churchers/vm-bhyve) 上的说明操作。

所有命令必须以 root 身份运行，因此要么直接以 root 登录，要么先运行 `su`。

如果你有配置 `lagg`（网络冗余），请使用例如 `lagg0` 作为接口：`vm switch add public lagg0`

注意，当你使用有线/无线连接的 `lagg` 冗余时，虚拟机只能通过有线连接访问互联网。

为了保证互联网访问正常，需要确保设置如下：

```sh
sysctl net.link.tap.up_on_open=1
sysctl net.link.bridge.ipfw=0
sysctl net.link.bridge.pfil_bridge=0
sysctl net.link.bridge.pfil_member=0
```

我这里前两个值是自动设置的（可能是由 vm-bhyve 完成的），所以我只需要设置后两个。

>**注意**
>
>如果你的防火墙是 `ipfw`/`firewall`，那么这些设置可适用；如果你使用 `pf`，可能需要其他设置。

出处：[FreeBSD 论坛](https://forums.freebsd.org/threads/bhyve-and-firewall-on-host.75089/)

要在开机时生效，请将所需值追加到 **/etc/sysctl.conf** 中，但省略 `sysctl` 字样。我添加了：

```ini
# 禁用在网络桥上传输的数据包过滤：
net.link.bridge.pfil_bridge=0
net.link.bridge.pfil_member=0
```

接下来，为了能够安装和运行 Alpine Linux，先用 `alpine` 模板创建虚拟机：`vm create -t alpine -s 5G alpine`

这一步会运行失败，但能让你进入 grub 命令行。在这里你可以找到所选 Alpine ISO 中实际的 Linux 内核名称；不同发行版名称不同。vm-bhyve 提供的模板使用的是 `vmlinuz-vanilla` 和 `initramfs-vanilla`，这是与发行版相关的部分。

接着启动安装，例如：`vm install alpine alpine-virt-3.13.2-x86_64.iso`

连接虚拟机：`vm console alpine`

启动时，它会显示：

```sh
error: file `/boot/vmlinuz-vanilla' not found.
error: you need to load the kernel first.

Press any key to continue...
```

按任意键后，在 GRUB 菜单按 `c` 进入 grub 命令行。输入 `ls` 查看可用设备，对于 `alpine-virt-3.13.2-x86_64.iso`，结果为：`(cd0) (cd0,msdos2) (hd0) (host)`

逐层查看文件树，找到位于 **/boot** 的内核：

```sh
ls (cd0)/
apks/ boot/ efi/
```

继续查看 **/boot**：

```sh
grub> ls (cd0)/boot/
config-virt dtbs-virt/ grub/ initramfs-virt modloop-virt syslinux/ System.map-virt vmlinuz-virt
```

可以看到，对于这个 ISO，内核文件名是 `initramfs-virt` 和 `vmlinuz-virt`。

你可以通过输入 `reboot` 退出控制台，等待几秒钟，然后按 `~+Ctrl+d` 返回主机终端。

接下来，编辑该虚拟机的配置文件：`vm config alpine`（将 `alpine` 替换为你的虚拟机名称），或者复制模板创建自己的模板。对于本例中的 ISO，需要将所有 `vanilla` 替换为 `virt`，即将 **/boot/vmlinuz-vanilla** 替换为 **/boot/vmlinuz-virt**，**/boot/initramfs-vanilla** 替换为 **/boot/initramfs-virt**。

如果使用 `vm config alpine`，只需重新运行：`vm install alpine alpine-virt-3.13.2-x86_64.iso`，然后 `vm console alpine`

或者如果你创建了新模板，则创建新虚拟机，再运行安装。

登录用户名为 `root`（无密码），此时系统消息会提示运行 `setup-alpine`。这个命令才是真正安装 Alpine 的步骤，在此之前网络不会工作。

## UEFI 图形界面（VNC）

UEFI 图形（VNC）支持说明

从 5 月 27 日起，bhyve UEFI 现在支持帧缓冲设备（frame buffer），可通过 VNC 访问。

### 更新 vm-bhyve

确保使用至少 vm-bhyve 1.1 版本。

### 使用支持的 FreeBSD 版本

* 如果从源码构建，图形支持目前仅在 12-CURRENT 和 11-STABLE 可用。
* 如果使用二进制版本，可使用 11.0 RC。
* 不支持低于 FreeBSD 11 的版本。

### 获取最新 UEFI 固件

可以通过 Port `sysutils/uefi-edk2-bhyve` 安装 UEFI 固件。依赖较多，建议使用 pkg 安装：

```sh
# pkg install uefi-edk2-bhyve
```

> **更新**
>
> 现在 `pkg install sysutils/bhyve-firmware` 会自动安装 `edk2-bhyve` 和 `uefi-edk2-bhyve-csm`。原 Port `sysutils/uefi-edk2-bhyve` 删除完成。

对于 vm-bhyve 1.1-p3 及以后版本，安装包即可自动查找固件路径。旧版本需手动复制固件至虚拟机目录下的 `.config`。

### 更新虚拟机配置

在已使用 UEFI 的 Windows 虚拟机上，添加以下配置：

```ini
graphics="yes"
```

启动虚拟机时，会添加 800x600 的帧缓冲设备。VNC 端口会动态分配，可在 `vm list` 或 `vm info guest` 下的 console-ports 查看。

### 其他配置选项

* 默认创建 PS2 鼠标，适用于旧版本 Windows/FreeBSD。
* 使用 XHCI 鼠标（体验更佳）：

```ini
xhci_mouse="yes"
```

* FreeBSD 13.0 及以后，启用 hms(4) 驱动：

```sh
# /boot/loader.conf
hw.usb.usbhid.enable=1
usbhid_load="YES"
```

* FreeBSD 13.0 前，使用 [utouch](https://github.com/wulf7/utouch)（Port `misc/utouch-kmod`）。

* 指定 VNC 监听的主机 IP：

```ini
graphics_listen="1.2.3.4"
```

* 指定 VNC 端口（默认为 `5900`，可为每台虚拟机指定不同端口）：

```ini
graphics_port="5901"
```

* 设置分辨率（默认 800x600，可选以下分辨率）：

```ini
graphics_res="1600x900"
```

支持的分辨率有：

```ini
1920x1200
1920x1080
1600x1200
1600x900
1280x1024
1280x720
1024x768
800x600
640x480
```

* 等待 VNC 客户端连接（在安装过程中需要在启动早期按键时非常有用。默认为 `auto`，即 vm-bhyve 在安装模式下首次启动时会等待；设置为 `no` 时，guest 永远不会等待，即使在安装模式下）：

```ini
graphics_wait="yes"
```

## 输出信息说明

**Aug 5, 2016**

使用 `vm info` 和 `vm switch info` 命令时显示的信息概览。请注意，这些命令可以接受指定的 guest 名称或 switch 名称，但如果未提供，则会输出所有 guest 或 switch 的详细信息。

### `vm info guest`

```sh
------------------------
Virtual Machine: fbsd
------------------------
  state: running (33508)
  datastore: default
  loader: bhyveload
  uuid: e5881af2-53ed-11e6-b442-50e549369bc6
  uefi: no
  cpu: 2
  memory: 256M
  memory-resident: 21311488 (20.324M)

  console-ports
    com1: tmux/fbsd

  network-interface
    number: 0
    emulation: virtio-net
    virtual-switch: public
    fixed-mac-address: 58:9c:fc:01:a1:ce
    fixed-device: -
    active-device: tap2
    desc: vmnet-fbsd-0-public
    mtu: 1500
    bridge: bridge0
    bytes-in: 206873 (202.024K)
    bytes-out: 850 (850.000B)

  virtual-disk
    number: 0
    device-type: file
    emulation: virtio-blk
    options: -
    system-path: /data/vm/fbsd/disk0.img
    bytes-size: 21474836480 (20.000G)
    bytes-used: 379974656 (362.372M)
```

### 主要信息

#### State（状态）

guest 是否正在运行，如果在本地运行，还会显示 bhyve 进程的 PID。

##### Datastore（数据存储）

guest 存储所在的 vm-bhyve 数据存储名称。数据存储能使用多个 ZFS 数据集或其他文件系统来存储虚拟机。

##### Loader（引导加载器）

非 UEFI guest 使用的内核加载器类型，为 `grub` 或 `bhyveload`。

##### UUID（通用唯一识别码）

guest 自动分配的 UUID。创建 guest 时会生成静态 UUID，可在配置文件中修改。

##### UEFI（统一可扩展固件接口）

guest 是否配置为使用 UEFI。若使用 BIOS 兼容固件，应设置为 `csm`。

##### CPU（处理器）

分配给 guest 的 CPU 数量。

##### Memory（内存）

分配给 guest 的系统内存大小。

#### Memory Resident（常驻内存）

guest 当前占用的主机内存大小。

#### 控制台端口

列出 guest 配置的各种控制台或图形端口。com 端口会显示连接的 **/dev/nmdm** 设备，若启用 tmux（仅 1.2 版本），会显示 tmux 会话。图形（VNC）控制台会显示 VNC 服务器监听的 IP 和端口。

#### 网络接口

##### Number（数字）

接口索引，对应 `networkX` 配置选项。

##### Emulation（仿真）

网络硬件仿真类型，通常为 `virtio-net`，FreeBSD 12 还支持 `e1000`。

##### Virtual Switch（虚拟交换机）

接口连接的虚拟交换机名称。

##### Fixed MAC Address（固定的 MAC 地址）

接口的 MAC 地址。所有虚拟接口在运行时分配静态 MAC，可在配置文件中修改。

##### Fixed Device（固定的设备）

若接口配置为使用特定 `tap` 设备，会显示在此。可通过手动创建接口并在 guest 配置中使用 `networkX_device` 指定。

##### Active Device（活动的设备）

当前 guest 使用的 `tap` 设备。如果配置了静态设备，应与 Fixed Device 相同。默认情况下，`tap` 设备在运行时动态创建。

##### Desc（设备描述）

接口描述，用作内部标识，由 `vmnet` + guest 名称 + 接口编号 + switch 名称组成。

##### MTU（最大传输单元）

接口的 MTU，vm-bhyve 会尝试与所连接的虚拟交换机 MTU 匹配。

##### Bridge（桥）

该虚拟交换机对应的主机 bridge 接口名称。

##### Bytes In / Bytes Out（接收 / 发送的字节数）

接口接收 / 发送的字节数。

#### 虚拟磁盘

##### Number（数字）

接口内部索引，对应 guest 配置文件中的 `diskX` 选项。

##### Device Type（设备类型）

磁盘在主机上的存储类型，可能为稀疏文件（`file`）、ZVOL 或自定义。

##### Emulation（仿真）

仿真磁盘设备类型，通常为 `virtio-blk` 或 `ahci-hd`。

##### Options（参数）

磁盘的附加配置，例如扇区大小、只读设置等。

##### System Path（系统路径）

磁盘镜像在主机上的路径。

##### Bytes Size / Bytes Used（总字节数 / 实际使用的字节数）

磁盘镜像总大小 / guest 实际使用的字节数。

### vm switch info

```sh
------------------------
Virtual Switch: public
------------------------
  type: auto
  ident: bridge0
  vlan: -
  nat: -
  physical-ports: re0
  bytes-in: 628193006 (599.091M)
  bytes-out: 1421559466 (1.323G)

  virtual-port
    device: tap1
    vm: win2
```

### 主要信息

#### Type（类型）

虚拟交换机类型。`auto` 表示由 vm-bhyve 管理，bridge 设备自动创建；`manual` 表示使用自己在 `rc.conf` 中配置的 bridge 接口。

#### Ident（标识）

该交换机对应的实际 bridge 接口。

#### VLAN（虚拟局域网）

虚拟交换机设置的 VLAN 编号（如果有）。

#### NAT（网络地址转换）

虚拟交换机是否启用 NAT。

#### Physical Ports（物理端口）

分配给交换机的主机物理接口列表，vm-bhyve 会将这些接口添加到 bridge。

#### Bytes In / Bytes Out（接收 / 发送的字节数）

交换机接收 / 发送的字节数。

#### Virtual Port（虚拟端口）

每个连接到交换机的虚拟机都会显示虚拟端口条目。

##### Device（设备）

连接到交换机的 tap 设备。

##### VM（虚拟机）

使用该接口的虚拟机名称。


## UEFI 引导下的串行控制台输出

**Jun 29, 2025**

如果你使用 `loader="grub"`，所需的命令会自动注入，以确保串行输出正常。不幸的是，像 Ubuntu 20.04 LTS 这样的现代 Linux 发行版要求使用 UEFI 才能在不修改上游镜像的情况下正确启动。为了启用正确的输出，我们必须修改 guest 内的 grub 配置。

### Ubuntu 20.04 LTS

创建文件 **/etc/default/grub.d/99-bhyve.cfg**，内容如下：

```ini
GRUB_CMDLINE_LINUX_DEFAULT=""
GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"

GRUB_TERMINAL="serial console"
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
```

然后运行 `update-grub` 激活新设置。下次重启后，使用 `vm console <guest>` 应该可以输出控制台内容。

### Ubuntu 22.04 LTS

创建文件 **/etc/default/grub.d/99-bhyve.cfg**，内容如下：

```ini
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0,115200"
GRUB_TERMINAL="console serial"
GRUB_SERIAL_COMMAND="serial --speed=115200"
```

然后运行 `# update-grub` 激活新设置。下次重启后，使用 `vm console <guest>` 应该可以输出控制台内容。

### CentOS 7

按照 [示例模板](https://github.com/churchers/vm-bhyve/blob/master/sample-templates/centos7.conf) 和 [Red Hat 官方文档](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-working_with_the_grub_2_boot_loader) 操作，打开 **/etc/default/grub** 并更新为：

```ini
GRUB_CMDLINE_LINUX_DEFAULT=""
# 保留之前的值 (...) 然后添加控制台配置
GRUB_CMDLINE_LINUX="... console=tty0 console=ttyS0,115200n8"

GRUB_TERMINAL_OUTPUT="serial console"
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"
```

然后运行 `grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg` 激活新设置。下次重启后，使用 `vm console <guest>` 应该可以输出控制台内容。

### AlmaLinux 8

按照 [示例模板](https://github.com/churchers/vm-bhyve/pull/387) 和 [Red Hat 官方文档](https://access.redhat.com/articles/7212#config8) 操作，使用 root 执行以下命令：

```ini
kernelopts=$(grub2-editenv - list | grep kernelopts)
grub2-editenv - set "$kernelopts console=tty0 console=ttyS0,115200"
```

注意：如果你迫切需要通过 VNC 输入管道符号 (`|`) 但无法输入，可以按 `Alt+124`。参考[这里](https://www.david-scherfgen.de/loesung-pipe-symbol-in-vnc-tippen/)。

下次重启后，使用 `vm console <guest>` 应该可以输出控制台内容。


## 迁移虚拟机

**Apr 9, 2024**

### 从本地主机迁移虚拟机到远程主机

可以使用单条命令将虚拟机完整地从源主机迁移到目标主机。

```sh
vm migrate -s guest-name new-host
```

理想情况下应使用免密码的密钥认证，尽管并非必须。

更多详情请参见手册页：

```sh
vm migrate [-s12t] [-r name] [-i snapshot] [-d datastore] guest host
```

## 云镜像

**Oct 29, 2024**

### 使用云镜像

你可以使用云镜像来创建虚拟机。`vm img` 命令会将镜像下载到数据存储，并在需要时解压（支持 `.xz`、`.tar.gz` 和 `.gz` 文件）。镜像应为 RAW 或 QCOW2 格式。使用此功能需要安装软件包 `qemu` 或 `qemu-devel`：

```sh
pkg install qemu
```

使用官方云镜像启动 FreeBSD：

```sh
vm img https://download.freebsd.org/ftp/releases/VM-IMAGES/14.0-RELEASE/amd64/Latest/FreeBSD-14.0-RELEASE-amd64.raw.xz
vm create -t freebsd-zvol -i FreeBSD-14.0-RELEASE-amd64.raw freebsd-cloud
vm start freebsd-cloud
```

### 使用 cloud-init

vm-bhyve 支持为虚拟机提供基础的 cloud-init 配置。可通过 `vm create` 命令的选项 `-C` 启用。你还可以使用选项 `-k <文件>` 将公钥注入虚拟机。使用 cloud-init 需要安装软件包 `cdrkit-genisoimage`：

```sh
pkg install cdrkit-genisoimage
```

示例：

```sh
vm img https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
vm create -t ubuntu-cloud -i noble-server-cloudimg-amd64.img -C -k ~/.ssh/id_rsa.pub ubuntu-server
vm start ubuntu-server
Starting ubuntu-server
* found guest in /zroot/vm/ubuntu-server
* booting...
ssh ubuntu@192.168.0.91
The authenticity of host '192.168.0.91 (192.168.0.91)' can't be established.
ECDSA key fingerprint is SHA256:6s9uReyhsIXRv0dVRcBCKMHtY0kDYRV7zbM7ot6u604.
No matching host key fingerprint found in DNS.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.0.91' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 24.04.1 LTS (GNU/Linux 6.8.0-45-generic x86_64)
```

### Ubuntu 云镜像示例配置文件

```sh
# ➜  .templates cat ubuntu-cloud.conf
loader="uefi"
uefi_vars="yes"
cpu=4
memory=4096M
network0_type="virtio-net"
network0_switch="public"
graphics="yes"
xhci_mouse="yes"
graphics_res="1600x900"
zfs_zvol_opts="volblocksize=128k"
disk0_name="disk0"
disk0_dev="sparse-zvol"
disk0_type="virtio-blk"
```

## vm-bhyve 命令行接口

**Oct 20, 2025**

### 🔧 **全局与系统命令**

* **`vm version`** – 显示 vm-bhyve 的版本和修订号。
* **`vm init`** – 初始化 vm-bhyve 目录、默认数据存储和交换机。
* **`vm set [setting=value]`** – 设置全局 vm-bhyve 配置选项。
* **`vm get [all|setting]`** – 显示全局配置或指定选项。

### 🖧 **交换机管理**

* **`vm switch list`** – 列出所有定义的虚拟交换机。
* **`vm switch info <名称>`** – 显示指定交换机的信息。
* **`vm switch create <名称>`** – 创建新交换机（可选参数：`-t type`，`-i interface` 等）。
* **`vm switch vlan <名称> <vlan|0>`** – 为交换机分配 VLAN ID。
* **`vm switch nat <名称> <on|off>`** – 启用或禁用 NAT。
* **`vm switch private <名称> <on|off>`** – 切换私有模式（无外部接口）。
* **`vm switch add <名称> <接口>`** – 为交换机添加物理接口。
* **`vm switch remove <名称> <接口>`** – 从交换机移除接口。
* **`vm switch destroy <名称>`** – 删除交换机。

### 💾 **数据存储管理**

* **`vm datastore list`** – 列出所有数据存储。
* **`vm datastore add <名称> <路径>`** – 在指定路径添加数据存储。
* **`vm datastore remove <名称>`** – 删除数据存储。

### 🖥️ **虚拟机管理**

* **`vm list`** – 列出所有虚拟机。
* **`vm info <名称>`** – 显示虚拟机详细信息。
* **`vm create <名称>`** – 创建新虚拟机（可选模板/数据集）。
* **`vm install <名称> <iso>`** – 使用 ISO 启动虚拟机进行安装。
* **`vm start <名称>`** – 启动虚拟机。
* **`vm stop <名称>`** – 优雅停止虚拟机。
* **`vm restart <名称>`** – 重启虚拟机。
* **`vm console <名称>`** – 连接虚拟机控制台（串口或图形界面）。
* **`vm configure <名称>`** – 编辑虚拟机配置文件。
* **`vm rename <旧> <新>`** – 重命名虚拟机。
* **`vm add <名称>`** – 向虚拟机添加磁盘或网卡。

### 🚀 **批量与电源控制**

* **`vm startall`** – 启动 **/etc/rc.conf** 中 `vm_list=" "` 列出的所有虚拟机。
* **`vm stopall`** – 停止所有运行中的虚拟机。
* **`vm reset <名称>`** – 强制重置虚拟机（类似按下重置键）。
* **`vm poweroff <名称>`** – 强制关闭虚拟机。
* **`vm destroy <名称>`** – 删除虚拟机及其资源。

### 🧩 **高级 / 其他**

* **`vm passthru`** – 列出可用于 PCI 直通的设备。
* **`vm clone <名称> <新名称>`** – 克隆虚拟机（可选择快照）。
* **`vm snapshot <名称>`** – 创建虚拟机快照。
* **`vm rollback <名称@快照>`** – 回滚虚拟机至指定快照。

### 💿 **镜像 / ISO**

* **`vm iso [url]`** – 列出或下载 ISO 镜像。
* **`vm img [url]`** – 列出或下载 raw 镜像文件。
* **`vm image list`** – 列出注册的虚拟机镜像。
* **`vm image create <名称>`** – 从虚拟机创建可复用镜像。
* **`vm image destroy <uuid>`** – 删除镜像。
* **`vm image provision <uuid> <新名称>`** – 从镜像创建新虚拟机。

## 代码布局

**Jun 27, 2016**

`vm-bhyve` 文件概览及其用途。

### ./vm

这是用户运行的主要命令。它几乎不包含逻辑，仅执行一些检查，确保 `vm-bhyve` 已启用、库文件和 **$vm_dir** 存在，并且操作系统和当前用户符合要求。如果运行在 ZFS 上，我们会将 **$vm_dir** 替换为指定数据集的挂载点。然后调用 `__parse_cmd` 来确定用户想执行的功能。

所有功能都存储在库文件中，通常位于 **/usr/local/lib/vm-bhyve/**。我们也会在 **./lib/** 中查找库文件，这能让你直接从当前目录运行 `vm`，便于开发。

### ./lib/vm-cmd

包含一些 case 语句，用于解析用户请求的命令并运行相应的函数。该文件还包含解析全局 `vm` 选项的函数。

### ./lib/vm-config

解析配置文件并返回配置变量的函数。我们使用简单的配置加载器，它读取配置文件并存储在全局变量中。`__config_get` 函数在全局变量中查找所请求的设置并返回。由于此文件代码量较小，将来可能会合并到 `vm-common`。

### ./lib/vm-core

处理基本命令（如 `start|stop|install|iso|console` 等）的所有函数。大多数命令（除 switch 和 ZFS 特定命令外）都由此文件中的函数处理。该文件以前还包含运行 bhyve 的代码，但因文件过大，迁移到了 `vm-run`。

### ./lib/vm-guest

包含运行需要启动加载器（bhyveload 或 grub-bhyve）的 guest 所需的代码。以前每个操作系统都有对应函数，现在所有 guest 都由通用的 `__guest_load` 函数处理。由于该文件现在仅包含两个函数，将来可能会合并回 `vm-run`。

### ./lib/vm-info

`vm-bhyve` 有两个“信息”命令（`vm info [guest]` & `vm switch info [switch]`），用于输出关于 guest 和 switch 的大量信息。代码量较大，因此在此文件中单独实现。

### ./lib/vm-rctl

为 bhyve 进程分配 rctl 限制的代码。此函数在 bhyve 启动前调用，然后等待 bhyve 启动，因为只要 bhyve 启动，我们的代码会阻塞直至其退出。

### ./lib/vm-run

启动 bhyve 的主要逻辑。生成 bhyve 设备字符串、创建控制台端口、分配网络接口的代码都在此文件中。我们将 bhyve 命令拆分到不同变量（`_devices`, `_comstring`, `_opts`, `_iso_dev`），所有 guest 通过一次 bhyve 调用处理。

### ./lib/vm-switch

处理 `vm switch ...` 命令的所有代码。还包含 switch 库函数，如 `__vm_switch_id`，用于根据 switch 名称返回对应的桥接接口名。

### ./lib/vm-sysrc

处理使用 sysrc 更新配置文件的函数。有时需要在现有配置中追加或删除值，这些操作由此文件中的函数处理。与 `vm-config` 类似，代码量较小，日后可能合并到 `vm-common`。

### ./lib/vm-util

包含各种通用工具函数。常用函数如 `util::err` 和 `util::log` 存放在此处。

### ./lib/vm-zfs

处理所有 ZFS 功能的函数。快照、克隆、回滚以及所有 `vm image` 命令的代码都在此文件中。

还有函数 `__zfs_init`，用于在 ZFS 上正确设置 vm-bhyve。在所有情况下 **$vm_dir** 应为 vm-bhyve 数据的文件系统路径。但在 ZFS 上，用户需提供 `"zfs:pool/vm"` 而非文件系统路径。`__zfs_init` 会确保 ZFS 正在运行，然后获取指定数据集的挂载点。如果成功，挂载点存储在 **$vm_dir**，数据集存储在全局 **$VM_ZFS_DATASET** 变量中。这样，大部分 vm-bhyve 代码可以像平常一样使用 **$vm_dir**，需要直接访问 ZFS 数据集的函数可使用 **$VM_ZFS_DATASET**。


## 代码风格

**Jun 30, 2016**

### 变量

一般情况下，我们对变量使用以下格式：

```sh
全局变量 - ${ALL_CAPS}
局部变量 - ${_variable}
```

除非变量设计为全局变量，否则应始终在当前函数顶部使用 `local` 声明。函数有时会修改在父函数中声明的变量，在这种情况下，应在函数说明中的 `@modifies` 列表中列出该变量，以明确该函数会修改其作用域之外的变量。

所有变量都使用花括号访问：`${_variable}`

### 函数

大部分函数前缀使用其所属库的名称，以便明确函数的定义来源：

```sh
lib::function_name
```

大多数函数前会有简短说明，解释函数的功能，并列出参数和/或返回值，除非代码本身非常明显。

私有函数前缀使用 `__`。

### 代码风格

#### 注释

代码应充分注释，特别是当逻辑不易立即理解时。

#### 缩进与行长

所有代码应正确缩进，使用 4 个空格。行长度应保持合理，如有必要，可使用 `\` 将代码分行。

#### If 语句

If 语句的写法如下。如果有多个语句，应在语句间留空行：

```sh
# 检查某条件
if [ test ]; then
    code
fi

# 检查其他条件
if [ test2 ]; then
    code2
fi
```

在合适的情况下，可将测试简化为一行，提高简洁性和可读性：

```sh
[ "check err" ] && util::err "Some error"
[ "check ok" ] || util::err "Another error"
```

为了简化代码并缩短行长，函数应尽量先进行检查，然后在必要时直接返回或退出，而不是在 if 语句中包含大量代码块，例如：

```sh
lib::good_function(){

   [ "check err condition" ] && return 1

   main_code_block
}

lib::bad_function(){

    if [ "check ok condition" ]; then

        main_code_block
    else
        return 1
    fi
}
```

#### 返回值

在合理情况下，函数成功返回 `0`，出错返回其他整数。对于返回值，我们更倾向于使用 `setvar`，而不是 `$(func)` 和 `echo`，因为后者需要子 Shell，而不使用子 Shell 已证明可提升性能。

```sh
lib::good_function(){
    local _var="$1"

    setvar "${_var}" "return value"
}

lib::bad_function(){

    echo "return value"
}

_variable=$(lib::bad_function)
lib::good_function "_variable"
```

#### Case 语句

在一般情况下，`;;` 应单独换行以提高可读性。唯一例外是当每个 case 仅包含一行时，换行会不必要地增加代码长度。函数调用应对齐：

```sh
case "${_variable}" in
    one)   func_one ;;
    two)   func_two ;;
    three) func_three ;;
esac
```



