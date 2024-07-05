# 旧版 FreeBSD 发布工程

<details open="" data-immersive-translate-walked="69735f23-684d-4cff-8b6a-5407c65b4955"><summary data-immersive-translate-walked="69735f23-684d-4cff-8b6a-5407c65b4955" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

英特尔、赛扬、迅驰、酷睿、EtherExpress、i386、i486、安腾、奔腾和至强是英特尔公司或其子公司在美国和其他国家的商标或注册商标。

制造商和销售商用来区分其产品的许多称号被视为商标。如果这些称号出现在本文件中，并且 FreeBSD 项目意识到了商标声明，这些称号将会被“™”或“®”符号跟随。

</details>

 摘要

|  | 本文档已过时，不准确描述了 FreeBSD 发布工程团队当前的发布程序。仅供历史目的保留。FreeBSD 发布工程团队当前使用的程序可以在《FreeBSD 发布工程》文章中找到。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |

本文描述了 FreeBSD 发布工程团队用于制作 FreeBSD 操作系统的生产级发布的方法。它详细介绍了官方 FreeBSD 发布所使用的方法论，并描述了为那些有兴趣为公司推出或商业产品化定制 FreeBSD 发布的人士提供的工具。

---

## 1. 简介

FreeBSD 的开发是一个非常开放的过程。FreeBSD 由世界各地数千人的贡献组成。FreeBSD 项目向普通公众提供 Subversion 访问，以便他人可以访问日志消息、开发分支之间的差异（补丁）以及其他形式源代码管理提供的其他生产力增强功能。这对吸引更多有才华的开发人员来参与 FreeBSD 起到了巨大帮助。然而，我认为每个人都会同意，如果向互联网上的每个人开放对主要存储库的写访问权限，则混乱很快就会显现。因此，只有近 300 人组成的“精选”团队被授予对 Subversion 存储库的写访问权限。这些 FreeBSD committers 通常是进行大部分 FreeBSD 开发的人员。一些选举产生的核心团队开发者为该项目提供一定程度的指导。

由于 FreeBSD 开发的快速步伐，使得主要开发分支不适合普通公众的日常使用。特别是，需要进行稳定化工作，将开发系统打磨成生产质量的版本。为了解决这一冲突，开发工作在多个平行轨道上进行。主要开发分支是我们的 Subversion 树的 HEAD 或 trunk，称为"FreeBSD-CURRENT"或简称为"-CURRENT"。

一组较稳定的分支被维护，称为"FreeBSD-STABLE"或简称为"-STABLE"。所有分支都在由 FreeBSD 项目维护的主 Subversion 存储库中存在。FreeBSD-CURRENT 是 FreeBSD 开发的“前沿”，所有新更改首先进入该系统。FreeBSD-STABLE 是从中进行主要发布的开发分支。更改以不同的速度进入此分支，并且通常假设它们首先进入 FreeBSD-CURRENT 并经过我们的用户社区的彻底测试。

分支名称中的稳定一词是指项目所承诺的预期的应用程序二进制接口稳定性。这意味着在同一分支的旧版本系统上编译的用户应用程序可以在同一分支的新版系统上运行。与以前的版本相比，ABI 稳定性有了很大的提高。在大多数情况下，旧 STABLE 系统的二进制文件可以在新的系统上未经修改地运行，包括 HEAD，前提是没有使用系统管理接口。

在发布之间的过渡期间，FreeBSD 项目的构建机器会自动生成每周快照，并可从 https:/download.FreeBSD.org/snapshots/ 下载。二进制发行快照的广泛可用性，以及我们用户社区通过 Subversion 和 “make buildworld” ^[ 4]^ 继续跟进 -STABLE 开发的趋势，有助于保持 FreeBSD-STABLE 在重大发布前的质量保证活动启动之前处于非常可靠的状态。

除了安装 ISO 快照，每周还提供虚拟机镜像，可用于 VirtualBox、qemu 或其他流行的仿真软件。这些虚拟机镜像可以从 https://download.FreeBSD.org/snapshots/VM-IMAGES/ 下载。

虚拟机镜像大约为 150MB xz(1) 压缩文件，连接到虚拟机时包含一个 10GB 稀疏文件系统。

用户在整个发布周期内不断提交错误报告和功能请求。问题报告通过我们在 https://www.freebsd.org/support/bugreports/ 提供的 web 界面输入到我们的 Bugzilla 数据库中。

为了服务我们最保守的用户，从 FreeBSD 4.3 开始引入了独立的发布分支。这些发布分支是在最终发布之前不久创建的。发布后，只有最关键的安全修复和新增内容会合并到发布分支中。除了通过 Subversion 进行源代码更新外，还有二进制补丁包可用于保持 releng/X.Y 分支的系统更新。

### 1.1. 本文描述的内容

本文的以下部分描述：

发布过程发布工程过程中不同阶段，直至实际系统构建。

发布构建实际的构建过程。

基础发行版如何由第三方扩展。

从 FreeBSD 4.4 中吸取的教训

未来发展方向

## 2. 发行过程

FreeBSD 的新版本大约每四个月从 -STABLE 分支发布一次。FreeBSD 的发行过程在预定发布日期前 70-80 天开始，当发行工程师向开发邮件列表发送邮件提醒开发人员他们只有 15 天的时间来集成新更改，然后代码冻结。在此期间，许多开发人员会进行所谓的 "MFC 扫描"。

MFC 代表 "Merge From CURRENT"，描述了将经过测试的更改从我们的 -CURRENT 开发分支合并到我们的 -STABLE 分支的过程。项目政策要求任何更改必须首先应用于主干，并在经过 -CURRENT 用户（开发人员应在提交到 -CURRENT 之前对更改进行广泛测试，但一个人不可能对通用操作系统的所有用法进行测试）进行充分的外部测试后合并到 -STABLE 分支。最短的 MFC 期是 3 天，通常仅用于琐碎或关键的错误修复。

### 2.1. 代码审查

预计发布日期前 60 天，源代码库进入“冻结代码”阶段。在此期间，所有提交到-稳定-分支的提交必须经过 Release Engineering Team <<a href="mailto:re@FreeBSD.org">re@FreeBSD.org</a>> 的批准。批准过程由一个预提交挂钩技术强制执行。在这段时间内允许的更改类型包括：

* 错误修复。
* 文档更新。
* 任何与安全相关的修复。
* 对设备驱动程序的轻微更改，如添加新的设备 ID。
* 来自供应商的驱动程序更新。
* 发布工程团队认为有必要的任何额外变更，考虑到潜在的风险。

在代码冻结开始不久后，会构建并发布一个用于广泛测试的 BETA1 镜像。在代码冻结期间，每两周至少发布一个 beta 镜像或发行候选版，直到最终版本准备就绪。在最终版本发布前的几天，发布工程团队将与安全官员团队、文档维护人员和port维护人员保持密切沟通，以确保实现成功发布所需的所有不同组件都可用。

当 BETA 镜像的质量令人满意，并且没有计划进行大规模且潜在风险的更改时，创建发布分支，并从发布分支构建发布候选 (RC) 镜像，而不是从 STABLE 分支构建 BETA 镜像。同时，STABLE 分支的冻结状态解除，发布分支进入“硬代码冻结”，除非涉及严重的 bug 修复或安全问题，否则很难证明对系统进行新更改的合理性。

### 2.2. 最终发布检查表

当若干 BETA 镜像可供广泛测试并解决所有主要问题后，可以开始最终发布的“润色”工作。

#### 2.2.1. 创建发布分支

|  | 在下面的所有示例中， $FSVN 指的是 FreeBSD Subversion 仓库的位置， svn+ssh://svn.FreeBSD.org/base/ 。 |
| -- | ------------------------------------------------------------------------------------------------------ |

FreeBSD 分支在 Subversion 中的布局在提交者指南中有描述。创建分支的第一步是确定要从中分支的 stable/<em>X</em> 源的修订版本。

```
# svn log -v $FSVN/stable/9
```

下一步是创建发布分支

```
# svn cp $FSVN/stable/9@REVISION $FSVN/releng/9.2
```

可以检出此分支：

```
# svn co $FSVN/releng/9.2 src
```

|  | 创建 releng 分支和 release 标签由发布工程团队完成。 |
| -- | ----------------------------------------------------- |

![FreeBSD Development Branch](https://docs.freebsd.org/images/articles/releng/branches-head.png)

![FreeBSD 3.x STABLE Branch](https://docs.freebsd.org/images/articles/releng/branches-releng3.png)

![FreeBSD 4.x STABLE Branch](https://docs.freebsd.org/images/articles/releng/branches-releng4.png)

![FreeBSD 5.x STABLE Branch](https://docs.freebsd.org/images/articles/releng/branches-releng5.png)

![FreeBSD 6.x STABLE Branch](https://docs.freebsd.org/images/articles/releng/branches-releng6.png)

![FreeBSD 7.x STABLE Branch](https://docs.freebsd.org/images/articles/releng/branches-releng7.png)

![FreeBSD 8.x STABLE Branch](https://docs.freebsd.org/images/articles/releng/branches-releng8.png)

![FreeBSD 9.x STABLE Branch](https://docs.freebsd.org/images/articles/releng/branches-releng9.png)

#### 2.2.2. 增加版本号

在最终版本可以被标记、构建和发布之前，需要修改以下文件以反映 FreeBSD 的正确版本：

* doc/en_US.ISO8859-1/books/handbook/mirrors/chapter.xml
* doc/en_US.ISO8859-1/books/porters-handbook/book.xml
* doc/en_US.ISO8859-1/htdocs/cgi/ports.cgi
* ports/Tools/scripts/release/config
* doc/shared/xml/freebsd.ent
* src/Makefile.inc1
* src/UPDATING
* src/gnu/usr.bin/groff/tmac/mdoc.local
* src/release/Makefile
* src/release/doc/en_US.ISO8859-1/shared/xml/release.dsl
* src/release/doc/shared/examples/Makefile.relnotesng
* src/release/doc/shared/xml/release.ent
* src/sys/conf/newvers.sh
* src/sys/sys/param.h
* src/usr.sbin/pkg_install/add/main.c
* doc/en_US.ISO8859-1/htdocs/search/opensearch/man.xml

发布说明和勘误文件也需要为新版本（在发布分支上）进行调整，并在稳定/当前分支上适当截断：

* src/release/doc/en_US.ISO8859-1/relnotes/common/new.xml
* src/release/doc/en_US.ISO8859-1/errata/article.xml

Sysinstall 应该更新以注明可用的 ports 数量和 Ports 集合所需的磁盘空间。^[ 5]^ 此信息目前保存在 src/usr.sbin/bsdinstall/dist.c 中。

发布版本构建完成后，应更新一些文件以向世界宣布发布。这些文件相对于 head/ 在 doc/ 子版本树中。

* share/images/articles/releng/branches-relengX.pic
* head/shared/xml/release.ent
* en_US.ISO8859-1/htdocs/releases/*
* en_US.ISO8859-1/htdocs/releng/index.xml
* share/xml/news.xml

此外，更新“BSD 家族树”文件：

* src/shared/misc/bsd-family-tree

#### 2.2.3. 创建发布标签

当最终版本准备好时，以下命令将创建 release/9.2.0 标签。

```
# svn cp $FSVN/releng/9.2 $FSVN/release/9.2.0
```

文档和Ports管理人员负责用 tags/RELEASE_9_2_0 标签标记各自的树。

当使用子版本 svn cp 命令创建发布标记时，这将标识特定时间点的源代码。通过创建标记，我们确保未来的发布构建者总是能够使用我们用来创建官方 FreeBSD 项目发布的相同源代码。

## 3. 发布构建

FreeBSD “发布版”可以由任何拥有快速计算机和访问源代码库的人构建（这应该是每个人，因为我们提供子版本访问！请参阅手册中的子版本部分获取详细信息）。唯一的特殊要求是 md(4) 设备必须可用。如果设备未加载到您的内核中，则在创建引导媒体过程中执行 mdconfig(8) 时应自动加载内核模块。构建发布所需的所有工具都可以从 src/release 中的子版本存储库中获得。这些工具旨在提供构建 FreeBSD 发行版的一致方法。实际上只需一个命令就可以构建完整的发布版，包括创建适合刻录到 CDROM 或 DVD 的 ISO 映像文件以及 FTP 安装目录。release(7) 充分记录了用于构建发布版的脚本。 generate-release.sh 是围绕 Makefile 目标的包装器： make release 。

### 3.1. 构建一个发布

release(7)文件记录了构建 FreeBSD 发布所需的确切命令。以下命令序列可以构建一个 9.2.0 发布：

```
# cd /usr/src/release
# sh generate-release.sh release/9.2.0 /local3/release
```

运行这些命令后，所有准备好的发布文件都会在/local3/release/R 目录中可用。

发布 Makefile 可以分解为几个不同的步骤。

* 在一个单独的目录层次结构中创建一个经过消毒的系统环境，使用“make installworld”。
* 从 Subversion 检出系统源代码、文档和 ports 的干净版本到发布构建层次结构中。
* 在 chrooted 环境中填充 /etc 和 /dev。
* chroot 进入发布构建层次结构，以使外部环境更难污染此构建。
* make world 在 chrooted 环境中。
* 构建与凯比尔相关的二进制文件。
* 构建通用内核。
* 创建一个分段目录树，在其中构建和打包二进制分发。
* 构建和安装文档工具链，用于将文档源文件（SGML）转换为 HTML 和文本文档，并随发布一起提供。
* 构建和安装实际文档（用户手册、教程、发行说明、硬件兼容性列表等）。
* 打包二进制和源码分发 tar 包。
* 创建 FTP 安装层次结构。
* （可选）为 CDROM/DVD 介质创建 ISO 镜像。

有关发布构建基础设施的更多信息，请参阅 release(7)。

|  | 从 /etc/make.conf 中删除任何特定于站点的设置是很重要的。例如，在一个设置了 CPUTYPE 为特定处理器的系统上构建二进制文件并分发是不明智的。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------- |

### 3.2. 贡献软件 ("ports")

FreeBSD Ports 集合是一个包含超过 36000 个第三方软件包的集合，可用于 FreeBSD。 Ports Management Team <<a href="mailto:portmgr@FreeBSD.org">portmgr@FreeBSD.org</a>> 负责维护一个一致的 ports 树，用于创建官方 FreeBSD 版本附带的二进制软件包。

### 发布 ISOs

从 FreeBSD 4.4 开始，FreeBSD 项目决定发布以前在 BSDi/Wind River Systems/FreeBSD Mall “官方” CDROM 发行版上销售的四个 ISO 映像。每张光盘必须包含一个 README.TXT 文件，用于解释光盘的内容，一个 CDROM.INF 文件，为 bsdinstall(8) 提供光盘的元数据以便验证和使用其中的内容，并一个 filename.txt 文件，提供光盘的清单。该清单可以通过简单的命令创建：

```
/stage/cdrom# find . -type f | sed -e 's/^\.\///' | sort > filename.txt
```

每张 CD 的具体要求如下。

#### 3.3.1. 磁盘 1

第一张磁盘几乎完全由 make release 创建。对 disc1 目录应进行的唯一更改是添加一个 tools 目录，并尽可能多地添加第三方流行软件包以适应磁盘。tools 目录包含允许用户从其他操作系统创建安装软盘的软件。应使此磁盘可引导，以便现代 PC 用户不需要创建安装软盘。

如果要包含 FreeBSD 的定制内核，则必须更新 bsdinstall(8) 和 release(7) 以包括安装说明。相关代码包含在 src/release 和 src/usr.sbin/bsdinstall 中。具体来说，需要在 src/usr.sbin/bsdinstall 下更新 src/release/Makefile、dist.c、dist.h、menus.c、install.c 和 Makefile 文件。可选地，您可以选择更新 bsdinstall.8。

#### 3.3.2. 磁盘 2

第二张磁盘也主要由 make release 创建。此磁盘包含一个“live filesystem”，可用于从 bsdinstall(8) 解决 FreeBSD 安装问题。此磁盘应是可启动的，并且还应包含 CVSROOT 目录中的 CVS 仓库压缩副本和 commerce 目录中的商业软件演示。

#### 3.3.3. 多卷支持

Sysinstall 支持多卷软件包安装。 这要求每个光盘都有一个包含一组所有卷上所有软件包的 INDEX 文件，以及一个额外字段，指示该特定软件包位于哪个卷上。 集合中的每个卷还必须在 cdrom.inf 文件中设置 CD_VOLUME 变量，以便 bsdinstall 可以告诉哪个卷是哪个。 当用户尝试安装不在当前光盘上的软件包时，bsdinstall 将提示用户插入适当的光盘。

## 4. 发行版

### 4.1. FTP 站点

当发布经过彻底测试并打包进行分发后，必须更新主 FTP 站点。官方的 FreeBSD 公共 FTP 站点都是其他 FTP 站点专用的主服务器的镜像。这个站点被称为 ftp-master 。当发布准备就绪时，必须修改以下文件 ftp-master ：

/pub/FreeBSD/releases/arch/X.Y-RELEASE/来自 make release 的可安装 FTP 目录。

/pub/FreeBSD/ports/arch/packages-X.Y-release/这个版本的完整软件包构建。

/pub/FreeBSD/releases/arch/X.Y-RELEASE/tools 是指向../../../tools 的符号链接。

/pub/FreeBSD/releases/arch/X.Y-RELEASE/packages 是指向../../../ports/arch/packages-X.Y-release 的符号链接。

/pub/FreeBSD/releases/arch/ISO-IMAGES/X.Y/X.Y-RELEASE-arch- .iso 是 ISO 镜像。 " " 是 disc1, disc2 等。如果只有 disc1 并且有一个备用的第一安装 CD（例如一个没有窗口系统的精简安装），也可能有一个 mini。

关于 FreeBSD FTP 站点的分发镜像架构的更多信息，请参阅 Mirroring FreeBSD 文章。

在更新 ftp-master 后，可能需要几个小时到两天的时间，才能让大多数一级 FTP 站点拥有新的软件，这取决于是否同时加载了一个软件包集。发布工程师必须在宣布 FTP 站点上新软件的普遍可用性之前，与 FreeBSD 镜像站点管理员协调。理想情况下，发布的软件包集应至少在发布日前四天加载。发布内容应在计划发布时间前 24 到 48 小时内加载，并关闭“其他”文件权限。这样镜像站点可以下载，但公众不能从镜像站点下载。在发布内容发布时，应向 FreeBSD 镜像站点管理员发送邮件，告知发布已分阶段进行，并注明镜像站点应开始允许访问的时间。务必包括时区，例如相对于 GMT 的时间。

### 4.2. CD-ROM 复制

即将推出：将 FreeBSD 镜像发送到复制器的技巧以及需要采取的质量保证措施。

## 5. 可扩展性

尽管 FreeBSD 形成了一个完整的操作系统，但没有任何东西强迫你完全按照我们打包分发的方式使用该系统。我们试图将系统设计得尽可能可扩展，以便它可以作为其他商业产品的构建平台。我们关于此的唯一“规则”是，如果你要分发经过非琐碎更改的 FreeBSD，我们鼓励你记录你的增强功能！FreeBSD 社区只能帮助支持我们提供的软件用户。我们当然鼓励创新，例如高级安装和管理工具，但我们不能回答有关它的问题。

### 5.1. bsdinstall 的脚本

FreeBSD 系统安装和配置工具 bsdinstall(8)可被脚本化以实现大规模站点的自动化安装。此功能可以与 Intel® PXE ^[ 6]^结合使用，从网络引导系统。

## 从 FreeBSD 4.4 中学到的经验

4.4 的发布工程过程正式开始于 2001 年 8 月 1 日。在那之后，所有对 FreeBSD RELENG_4 分支的提交都必须得到 Release Engineering Team <<a href="mailto:re@FreeBSD.org">re@FreeBSD.org</a>> 的明确批准。第一个 x86 架构的候选版本于 8 月 16 日发布，随后又发布了 4 个候选版本，最终版本于 9 月 18 日发布。由于在早期候选版本中发现了几个安全问题，安全官在过程的最后一周非常参与。总共有超过 500 封电子邮件在一个多月内发送给 Release Engineering Team <<a href="mailto:re@FreeBSD.org">re@FreeBSD.org</a>> 。

我们的用户社区明确表示，FreeBSD 发布的安全性和稳定性不应因任何自行设定的截止日期或目标发布日期而受到影响。FreeBSD 项目在其生命周期内有了巨大的增长，标准化的发布工程过程的需求从未如此明显。随着 FreeBSD 被移植到新平台，这将变得更加重要。

## 7. 未来方向

我们的发布工程活动必须随着我们不断增长的用户基础进行扩展。在此过程中，我们正在努力记录生成 FreeBSD 发行版所涉及的过程。

* 并行性 - 发布构建的某些部分实际上是“非常适合并行的”。大多数任务都是非常 I/O 密集型的，因此拥有多个高速磁盘驱动器实际上比使用多个处理器更能加速 make release 过程。如果在 chroot(2) 环境中为不同的层次结构使用多个磁盘，那么 CVS 检出 ports 和文档树可以与另一个磁盘上的 make world 同时进行。使用 RAID 解决方案（硬件或软件）可以显著减少整体构建时间。
* 交叉构建发行版 - 在 x86 硬件上构建 IA-64 或 Alpha 发行版？ make TARGET=ia64 release 。
* 回归测试 - 我们需要更好的自动化正确性测试来适用于 FreeBSD。
* 安装工具 - 我们的安装程序早已超出了其预期的寿命。目前正在开发几个项目以提供更先进的安装机制。libh 项目就是这样一个旨在提供智能的新软件包框架和 GUI 安装程序的项目。

## 8. 致谢

我想感谢 Jordan Hubbard 给我机会承担 FreeBSD 4.4 的一些发布工程责任，也感谢他多年来为 FreeBSD 所做的所有工作。当然，如果没有 Satoshi Asami <<a href="mailto:asami@FreeBSD.org">asami@FreeBSD.org</a>> 、 Steve Price <<a href="mailto:steve@FreeBSD.org">steve@FreeBSD.org</a>> 、 Bruce A. Mah <<a href="mailto:bmah@FreeBSD.org">bmah@FreeBSD.org</a>> 、 Nik Clayton <<a href="mailto:nik@FreeBSD.org">nik@FreeBSD.org</a>> 、 David O’Brien <<a href="mailto:obrien@FreeBSD.org">obrien@FreeBSD.org</a>> 、 Kris Kennaway <<a href="mailto:kris@FreeBSD.org">kris@FreeBSD.org</a>> 、 John Baldwin <<a href="mailto:jhb@FreeBSD.org">jhb@FreeBSD.org</a>> 和其他 FreeBSD 开发社区成员所做的所有发布相关工作，这次发布是不可能实现的。我还想感谢 Rodney W. Grimes <<a href="mailto:rgrimes@FreeBSD.org">rgrimes@FreeBSD.org</a>> 、 Poul-Henning Kamp <<a href="mailto:phk@FreeBSD.org">phk@FreeBSD.org</a>> 和其他在 FreeBSD 早期日子里开发发布工程工具的人们。本文受到了 CSRG 发布工程文档^[7]^、NetBSD 项目^[8]^和 John Baldwin 提出的发布工程过程笔记^[9]^的影响。

---

[1](https://docs.freebsd.org/en/articles/releng/#_footnoteref_1). Subversion, [http://subversion.apache.org](http://subversion.apache.org/)

2. FreeBSD 提交者

3. FreeBSD 核心团队

4. 重建世界

5. FreeBSD Ports 集合 https://www.FreeBSD.org/ports
6. 无盘操作与 PXE
7. Marshall Kirk McKusick，Michael J. Karels 和 Keith Bostic：4.3BSD 的发布工程
8. NetBSD 开发者文档：发布工程 http://www.NetBSD.org/developers/releng/index.html
9. John Baldwin 的 FreeBSD Release Engineering Proposal https://people.FreeBSD.org/~jhb/docs/releng.txt
