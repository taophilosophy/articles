# 旧版 FreeBSD 发布工程

- 原文：[Legacy FreeBSD Release Engineering](https://docs.freebsd.org/en/articles/releng/)

## 摘要

>**注意**
>
>本文过时，无法准确概述 FreeBSD 发布工程团队当前的发布流程。本文仅为历史性文档保存。FreeBSD 发布工程团队当前使用的流程可以参阅文章 [FreeBSD 发布工程](https://docs.freebsd.org/en/articles/freebsd-releng/)。

本文介绍了 FreeBSD 发布工程团队用于制作 FreeBSD 操作系统生产质量发布版本的方法。它详细介绍了官方 FreeBSD 发布所用的流程，并为有兴趣为企业部署或商业化产品化制作定制 FreeBSD 发布版本的人介绍可用工具。


## 1. 介绍

FreeBSD 的开发过程非常开放。FreeBSD 由来自全球成千上万人的贡献组成。FreeBSD 项目提供 Subversion ^\[[1](https://docs.freebsd.org/en/articles/releng/#_footnotedef_1)]^ 访问权限，允许公众访问日志信息、开发分支之间的差异（补丁）以及其他正式源代码管理系统所提供的生产力提升。这对于吸引更多有才华的开发者加入 FreeBSD 项目起到了重要作用。然而，我想大家都会同意，如果将写访问权限开放给互联网上的每个人，混乱很快就会显现出来。因此，只有约 300 名经过挑选的人才被赋予 Subversion 仓库的写访问权限。这些 [FreeBSD 提交者](https://docs.freebsd.org/en/articles/contributors/#staff-committers)^\[[2](https://docs.freebsd.org/en/articles/releng/#_footnotedef_2)]^ 通常是进行 FreeBSD 开发工作的主要人员。一个由选举产生的 [核心团队](https://www.freebsd.org/administration/#t-core)^\[[3](https://docs.freebsd.org/en/articles/releng/#_footnotedef_3 "查看脚注")]^ 为项目提供一定程度的方向。

由于 `FreeBSD` 开发进程非常快速，主开发分支不适合大众日常使用。特别是，需要进行稳定化工作，将开发系统打磨成生产质量的发布版本。为了解决这个冲突，开发继续在多个并行的轨道上进行。主开发分支是 Subversion 树的 *HEAD* 或 *trunk*，称为 "FreeBSD-CURRENT" 或简写为 `-CURRENT`。

另外，FreeBSD 项目还维护着一组更加稳定的分支，称为 `FreeBSD-STABLE` 或简写为 `-STABLE`。所有分支都存在于 FreeBSD 项目维护的主 Subversion 仓库中。FreeBSD-CURRENT 是 FreeBSD 开发的“前沿”，所有的新更改首先进入系统。FreeBSD-STABLE 是主要版本发布所基于的开发分支。更改以不同的速度进入该分支，并且一般假设它们已经首先进入 FreeBSD-CURRENT，并经过了我们的用户社区的广泛测试。

分支名中的 *stable* 术语指的是项目承诺的应用二进制接口（ABI）稳定性。这意味着，在相同分支的较旧版本上编译的用户应用程序可以在较新的系统上运行，而无需修改。与之前的版本相比，ABI 稳定性有了很大改进。在大多数情况下，来自旧版 *STABLE* 系统的二进制文件在较新的系统上可以不做修改地运行，包括 *HEAD*，前提是没有使用系统管理接口。

在版本发布之间，FreeBSD 项目的构建机器每周会自动生成快照，并通过 `https:/download.FreeBSD.org/snapshots/` 提供下载。二进制发布快照的广泛可用性，以及我们的用户社区倾向于通过 Subversion 和 “make buildworld” ^\[[4](https://docs.freebsd.org/en/articles/releng/#_footnotedef_4 "查看脚注")]^ 跟进 -STABLE 开发，有助于保持 FreeBSD-STABLE 始终处于可靠的状态，即使在主要版本临近前的质量保证活动加速之前也是如此。

除了安装 ISO 快照外，每周还提供虚拟机镜像文件，供 VirtualBox、qemu 或其他流行的仿真软件使用。虚拟机镜像可以从 <https://download.FreeBSD.org/snapshots/VM-IMAGES/> 下载。

虚拟机镜像大约 150MB，通过 [xz(1)](https://man.freebsd.org/cgi/man.cgi?query=xz&sektion=1&format=html) 压缩，附加到虚拟机时包含一个 10GB 的稀疏文件系统。

在整个发布周期中，用户会不断提交错误报告和功能请求。这些问题报告通过在 [https://www.freebsd.org/support/bugreports/](https://www.freebsd.org/support/bugreports/) 提供的 Web 界面录入到 Bugzilla 数据库中。

为了服务于最保守的用户，FreeBSD 4.3 引入了独立的发布分支。这些发布分支在最终版本发布前不久创建。正式发布后，仅有最关键的安全修复和新增内容会合并到该发布分支。除了通过 Subversion 更新源代码外，还提供二进制补丁包，以帮助保持 *releng/X.Y* 分支的系统更新。

### 1.1. 本文介绍的内容

本文的以下章节介绍了：

[发布流程](https://docs.freebsd.org/en/articles/releng/#release-proc)
发布工程流程的不同阶段，直至实际系统构建。

[发布构建](https://docs.freebsd.org/en/articles/releng/#release-build)
实际的构建过程。

[扩展性](https://docs.freebsd.org/en/articles/releng/#extensibility)
如何由第三方扩展基本发布。

[FreeBSD 4.4 发布的经验教训](https://docs.freebsd.org/en/articles/releng/#lessons-learned)
通过发布 FreeBSD 4.4 学到的一些经验教训。

[未来方向](https://docs.freebsd.org/en/articles/releng/#future)
开发的未来方向。

## 2. 发布流程

FreeBSD 的新版本大约每四个月从 -STABLE 分支发布一次。FreeBSD 的发布流程在预计的发布日期前约 70-80 天开始加速，这时发布工程师会向开发邮件列表发送电子邮件，提醒开发人员他们只有 15 天时间将新的更改集成到代码库中，之后将会冻结代码。在此期间，许多开发人员会进行所谓的 "MFC 扫描"（MFC sweep）。

MFC 代表 `Merge From CURRENT`，即从我们的 -CURRENT 开发分支合并经过测试的更改到 -STABLE 分支。项目政策要求任何更改必须首先应用到主干分支，然后在经过 -CURRENT 用户的充分外部测试后再合并到 -STABLE 分支（开发人员被要求在提交到 -CURRENT 之前进行广泛测试，但由于一个人无法遍历通用操作系统的所有使用情况）。最短 MFC 期限为 3 天，通常仅用于微小或关键的 bug 修复。

### 2.1. 代码审查

预计发布日期前 60 天，源代码仓库进入“代码冻结”阶段。在此期间，所有提交到 -STABLE 分支的更改都必须得到发布工程团队 `re@FreeBSD.org` 的批准。该批准过程在技术上通过预提交钩子（pre-commit hook）强制执行。此期间允许进行的更改包括：

- 错误修复。
- 文档更新。
- 任何类型的安全相关修复。
- 对设备驱动程序的轻微更改，如添加新的设备 ID。
- 来自供应商的驱动程序更新。
- 发布工程团队认为合理的任何其他更改，考虑到潜在的风险。

在代码冻结开始后不久，构建并发布 *BETA1* 镜像以进行广泛测试。在代码冻结期间，每两周至少发布一个 beta 镜像或发布候选版本（RC），直到最终版本准备好。在最终发布前的几天里，发布工程团队会与安全官员团队、文档维护人员和 Ports 维护人员保持密切联系，以确保发布所需的所有不同组件都准备就绪。

当 BETA 镜像的质量足够令人满意且不再计划进行大规模且潜在风险较高的更改时，发布分支将被创建，并从发布分支构建 *Release Candidate*（RC）镜像，而不是从 STABLE 分支构建的 BETA 镜像。此外，STABLE 分支上的冻结期将解除，发布分支将进入 "硬代码冻结"（hard code freeze）状态，在此状态下，除非涉及严重的 bug 修复或安全问题，否则很难再为系统做出新的更改。

### 2.2. 最终发布清单

当多个 BETA 镜像发布并进行广泛测试，且所有重大问题解决完毕时，最终发布的 "打磨" 阶段将开始。

#### 2.2.1. 创建发布分支

>**注意**
>
> 以下所有示例中的 `$FSVN` 指的是 FreeBSD Subversion 仓库的位置，`svn+ssh://svn.FreeBSD.org/base/`。

FreeBSD 在 Subversion 中的分支布局可参阅 [提交者指南](https://docs.freebsd.org/en/articles/committers-guide/#subversion-primer-base-layout)。创建分支的第一步是确定要从哪个 `stable/X` 源代码修订版创建分支。

```
# svn log -v $FSVN/stable/9
```

接下来的步骤是创建 *发布分支*：

```
# svn cp $FSVN/stable/9@REVISION $FSVN/releng/9.2
```

可以通过以下命令检出此分支：

```
# svn co $FSVN/releng/9.2 src
```

>**注意**
>
>创建 `releng` 分支和 `release` 标签由 [发布工程团队](https://www.freebsd.org/administration/#t-re) 完成。

![FreeBSD 开发分支](https://docs.freebsd.org/images/articles/releng/branches-head.png)

![FreeBSD 3.x STABLE 分支](https://docs.freebsd.org/images/articles/releng/branches-releng3.png)

![FreeBSD 4.x STABLE 分支](https://docs.freebsd.org/images/articles/releng/branches-releng4.png)

![FreeBSD 5.x STABLE 分支](https://docs.freebsd.org/images/articles/releng/branches-releng5.png)

![FreeBSD 6.x STABLE 分支](https://docs.freebsd.org/images/articles/releng/branches-releng6.png)

![FreeBSD 7.x STABLE 分支](https://docs.freebsd.org/images/articles/releng/branches-releng7.png)

![FreeBSD 8.x STABLE 分支](https://docs.freebsd.org/images/articles/releng/branches-releng8.png)

![FreeBSD 9.x STABLE 分支](https://docs.freebsd.org/images/articles/releng/branches-releng9.png)

#### 2.2.2. 更新版本号

在最终版本可以标记、构建并发布之前，以下文件需要修改，以反映正确的 FreeBSD 版本：

- **doc/en\_US.ISO8859-1/books/handbook/mirrors/chapter.xml**
- **doc/en\_US.ISO8859-1/books/porters-handbook/book.xml**
- **doc/en\_US.ISO8859-1/htdocs/cgi/ports.cgi**
- **ports/Tools/scripts/release/config**
- **doc/shared/xml/freebsd.ent**
- **src/Makefile.inc1**
- **src/UPDATING**
- **src/gnu/usr.bin/groff/tmac/mdoc.local**
- **src/release/Makefile**
- **src/release/doc/en\_US.ISO8859-1/shared/xml/release.dsl**
- **src/release/doc/shared/examples/Makefile.relnotesng**
- **src/release/doc/shared/xml/release.ent**
- **src/sys/conf/newvers.sh**
- **src/sys/sys/param.h**
- **src/usr.sbin/pkg\_install/add/main.c**
- **doc/en\_US.ISO8859-1/htdocs/search/opensearch/man.xml**

还需要调整发布说明和 Errata 文件，以适应新版本（在发布分支上），并适当截断（在 stable/current 分支上）：

- **src/release/doc/en\_US.ISO8859-1/relnotes/common/new\.xml**
- **src/release/doc/en\_US.ISO8859-1/errata/article.xml**

应更新 Sysinstall，以标注可用 Ports 的数量和 Ports 所需的磁盘空间。^\[[5](https://docs.freebsd.org/en/articles/releng/#_footnotedef_5 "查看脚注")^] 这些信息目前保存在 **src/usr.sbin/bsdinstall/dist.c** 中。

构建发布后，应更新若干文件以向世界宣布发布。这些文件位于 `doc/` Subversion 仓库的 `head/` 目录下：

- **share/images/articles/releng/branches-relengX.pic**
- **head/shared/xml/release.ent**
- **en\_US.ISO8859-1/htdocs/releases/\***
- **en\_US.ISO8859-1/htdocs/releng/index.xml**
- **share/xml/news.xml**

另外，还需要更新“BSD 家族树”文件：

- **src/shared/misc/bsd-family-tree**

#### 2.2.3. 创建发布标签

当最终发布准备就绪时，以下命令将创建 `release/9.2.0` 标签。

```sh
# svn cp $FSVN/releng/9.2 $FSVN/release/9.2.0
```

文档和 Ports 管理者负责使用 `tags/RELEASE_9_2_0` 标签标记他们各自的树。

当使用 Subversion `svn cp` 命令创建 *发布标签* 时，这会标识源代码在特定时间点的状态。通过创建标签，我们确保未来的发布构建者能够始终使用我们用于创建官方 FreeBSD 项目发布的相同源代码。

## 3. 发布构建

任何具有快速计算机和访问源代码库权限的人都可以构建 FreeBSD "发布"。（这应该是所有人，因为我们提供 Subversion 访问！详情请参见《[手册中的 Subversion 部分](https://docs.freebsd.org/en/books/handbook/#svn)》）。*唯一*的特殊要求是必须提供 [md(4)](https://man.freebsd.org/cgi/man.cgi?query=md&sektion=4&format=html) 设备。如果设备没有加载到内核中，那么在创建启动媒体阶段执行 [mdconfig(8)](https://man.freebsd.org/cgi/man.cgi?query=mdconfig&sektion=8&format=html) 时，内核模块应自动加载。构建发布所需的所有工具都可以从 Subversion 仓库中的 **src/release** 获取。这些工具旨在提供一种一致的方法来构建 FreeBSD 发布。实际上，可以仅通过一条命令来构建完整的发布，包括创建适合刻录到 CDROM 或 DVD 的 ISO 镜像，以及一个 FTP 安装目录。[release(7)](https://man.freebsd.org/cgi/man.cgi?query=release&sektion=7&format=html) 完整记录了用于构建发布的 `src/release/generate-release.sh` 脚本。`generate-release.sh` 是 `make release` 的包装器。

### 3.1. 构建发布

[release(7)](https://man.freebsd.org/cgi/man.cgi?query=release&sektion=7&format=html) 详细说明了构建 FreeBSD 发布所需的确切命令。以下命令序列可以构建 9.2.0 版本的发布：

```sh
# cd /usr/src/release
# sh generate-release.sh release/9.2.0 /local3/release
```

运行这些命令后，所有准备好的发布文件将保存在 **/local3/release/R** 目录中。

发布 **Makefile** 可以分为几个不同的步骤。

- 在一个单独的目录层次结构中创建一个干净的系统环境，通过 `make installworld`。
- 从 Subversion 检出一个干净的系统源代码、文档和 Ports 到发布构建层次结构中。
- 在 chroot 环境中填充 **/etc** 和 **/dev**。
- 进入发布构建层次结构的 chroot 环境，以减少外部环境对构建的污染。
- 在 chroot 环境中执行 `make world`。
- 构建与 Kerberos 相关的二进制文件。
- 构建 **GENERIC** 内核。
- 创建一个用于构建和打包二进制分发版的暂存目录树。
- 构建和安装文档工具链，用于将文档源（SGML）转换为 HTML 和文本文档，这些将随发布一起提供。
- 构建和安装实际文档（用户手册、教程、发布说明、硬件兼容性列表等）。
- 打包二进制和源代码的分发 tarball。
- 创建 FTP 安装层次结构。
- *(可选)* 创建适合 CDROM/DVD 媒体的 ISO 镜像。

有关发布构建基础设施的更多信息，请参见 [release(7)](https://man.freebsd.org/cgi/man.cgi?query=release&sektion=7&format=html)。

>**注意**
>
>从 **/etc/make.conf** 中删除任何特定站点的设置非常重要。例如，分发在 `CPUTYPE` 设置为特定处理器的系统上构建的二进制文件是不明智的。

### 3.2. 第三方软件（Ports）

[FreeBSD Ports Collection](https://ports.freebsd.org/) 是一个包含 36000 余款第三方软件包的集合，适用于 FreeBSD。Ports Management Team `portmgr@FreeBSD.org` 负责维护一个一致的 Ports 树，用于创建伴随官方 FreeBSD 发布的二进制包。

### 3.3. 发布 ISO

从 FreeBSD 4.4 开始，FreeBSD 项目决定发布之前在 *BSDi/Wind River Systems/FreeBSD Mall* "官方" CDROM 发行版中提供的所有四个 ISO 镜像。每个光盘必须包含一个 **README.TXT** 文件，解释光盘的内容，一个 **CDROM.INF** 文件，提供光盘的元数据，以便 [bsdinstall(8)](https://man.freebsd.org/cgi/man.cgi?query=bsdinstall&sektion=8&format=html) 验证并使用光盘内容，以及一个 **filename.txt** 文件，提供光盘的清单。这个 *清单* 可以通过以下简单命令创建：

```sh
/stage/cdrom# find . -type f | sed -e 's/^\.\///' | sort > filename.txt
```

每张光盘的具体要求如下所述。

#### 3.3.1. 光盘 1

第一张光盘几乎完全由 `make release` 创建。唯一应该对 **disc1** 目录做的更改是添加一个 **tools** 目录，并添加尽可能多的流行第三方软件包以适应光盘空间。**tools** 目录包含允许用户从其他操作系统创建安装软盘的软件。此光盘应制作成可启动的，以便现代 PC 用户无需创建安装软盘。

如果要包含自定义的 FreeBSD 内核，则必须更新 [bsdinstall(8)](https://man.freebsd.org/cgi/man.cgi?query=bsdinstall&sektion=8&format=html) 和 [release(7)](https://man.freebsd.org/cgi/man.cgi?query=release&sektion=7&format=html)，以包括安装说明。相关代码位于 **src/release** 和 **src/usr.sbin/bsdinstall** 中。具体来说，需要更新 **src/release/Makefile**，以及 **src/usr.sbin/bsdinstall** 目录下的 **dist.c**、**dist.h**、**menus.c**、**install.c** 和 **Makefile** 等文件。此外，可以选择更新 **bsdinstall.8**。

#### 3.3.2. 光盘 2

第二张光盘也大部分由 `make release` 创建。此光盘包含一个"实时文件系统"，可由 [bsdinstall(8)](https://man.freebsd.org/cgi/man.cgi?query=bsdinstall&sektion=8&format=html) 用于 FreeBSD 安装的故障排除。此光盘应为可启动，并且还应包含 **CVSROOT** 目录中的压缩版 CVS 仓库以及 **commerce** 目录中的商业软件演示。

#### 3.3.3. 多卷支持

Sysinstall 支持多卷软件包安装。这要求每张光盘有一个 **INDEX** 文件，包含所有卷中的软件包，并附加一个额外字段，指示该软件包所在的卷。每个卷还必须在 **cdrom.inf** 文件中设置 `CD_VOLUME` 变量，以便 bsdinstall 可以区分哪个卷是哪个。当用户尝试安装当前光盘上没有的软件包时，bsdinstall 会提示用户插入相应的光盘。

## 4. 分发

### 4.1. FTP 站点

当发布版本经过彻底测试并完成打包后，必须更新主 FTP 站点。所有官方的 FreeBSD 公共 FTP 站点都是某个主服务器的镜像，该主服务器仅对其他 FTP 站点开放，称为 `ftp-master`。当发布版本准备就绪后，必须在 `ftp-master` 上修改以下文件：

**/pub/FreeBSD/releases/arch/X.Y-RELEASE/**
这是由 `make release` 生成的可安装 FTP 目录。

**/pub/FreeBSD/ports/arch/packages-X.Y-release/**
这是该版本的完整包构建内容。

**/pub/FreeBSD/releases/arch/X.Y-RELEASE/tools**
指向 **../../../tools** 的符号链接。

**/pub/FreeBSD/releases/arch/X.Y-RELEASE/packages**
指向 **../../../ports/arch/packages-X.Y-release** 的符号链接。

**/pub/FreeBSD/releases/arch/ISO-IMAGES/X.Y/X.Y-RELEASE-arch-\*.iso**
ISO 镜像文件。其中的“\*”表示 **disc1**、**disc2** 等。如有 **disc1**，且存在另一个精简版本的安装光盘（例如不带图形系统的安装盘），则还可能存在一个 **mini** 光盘。

有关 FreeBSD FTP 站点分发镜像架构的更多信息，请参阅 [《Mirroring FreeBSD》](https://docs.freebsd.org/en/articles/hubs/) 一文。

在更新 `ftp-master` 后，大多数 Tier-1 FTP 站点完成同步可能需要数小时至两天，具体取决于是否同时加载了软件包集。发布工程师必须在对外宣布 FTP 站点上新软件正式可用之前，与 [FreeBSD 镜像站点管理员](https://lists.freebsd.org/subscription/mirror-announce) 进行协调。理想情况下，发布所需的软件包集合应至少在正式发布日期前四天上传。发布文件应在计划发布时间前 24 至 48 小时上传，且权限设置为禁止公众访问，以便镜像站点可以下载但公众暂时无法获取。当文件上传完成时，应发送邮件给 [FreeBSD 镜像站点管理员](https://lists.freebsd.org/subscription/mirror-announce)，说明发布文件就绪，并告知何时允许镜像站点对外开放访问。请务必提供带有时区的时间，例如相对于 GMT 的时间。

### 4.2. 复制光盘

敬请期待：向复制商提供 FreeBSD ISO 文件的相关建议，以及需要采取的质量控制措施。

## 5. 可扩展性

虽然 FreeBSD 本身是一个完整的操作系统，但我们并不强求用户必须按照我们打包发布的方式使用它。我们努力将系统设计得尽可能具有可扩展性，使其能够作为其他商业产品的构建平台。我们唯一的“规则”是：如果你打算分发包含非小幅更改的 FreeBSD，我们鼓励你记录你的改动！FreeBSD 社区只能支持我们所提供软件的用户。我们当然鼓励各种创新，例如开发更先进的安装和管理工具，但我们无法负责回答有关这些改动产生的问题。


### 5.1. 脚本化 `bsdinstall`

FreeBSD 的系统安装与配置工具 [bsdinstall(8)](https://man.freebsd.org/cgi/man.cgi?query=bsdinstall&sektion=8&format=html) 可以通过编写脚本来实现自动化安装，适用于大规模部署场景。此功能可与 Intel® PXE [^6] 配合使用，从网络引导系统启动。

## 6. 来自 FreeBSD 4.4 的经验教训

FreeBSD 4.4 的发布工程正式始于 2001 年 8 月 1 日。从该日起，所有提交至 FreeBSD `RELENG_4` 分支的更改都必须经过发布工程团队 [re@FreeBSD.org](mailto:re@FreeBSD.org) 明确批准。x86 架构的首个候选版本于 8 月 16 日发布，之后又发布了 4 个候选版本，最终版本于 9 月 18 日发布。在最后一周，安全官员密切参与了工作，因为在先前的候选版本中发现了若干安全问题。在短短一个多月的时间内，共向发布工程团队 [re@FreeBSD.org](mailto:re@FreeBSD.org) 发送了 *500* 余封电子邮件。

我们的用户群体明确表示，FreeBSD 发布版本的安全性与稳定性不应为任何自定的截止日期或目标发布日期所牺牲。随着 FreeBSD 项目的不断壮大，对标准化发布工程流程的需求愈发突出。随着 FreeBSD 被移植到更多新平台，这一点将变得更加重要。

## 7. 后续方向

我们的发布工程活动必须随着不断增长的用户基础而扩展。为此，我们正努力记录 FreeBSD 发布过程中所涉及的各项流程。

- **并行化** —— 发布构建过程中的某些部分实际上是“高度可并行”的。大多数任务对 I/O 要求较高，因此使用多块高速硬盘比使用多个处理器更能有效加快 `make release` 过程。如果在 [chroot(2)](https://man.freebsd.org/cgi/man.cgi?query=chroot&sektion=2&format=html) 环境中为不同的目录层级使用不同的硬盘，那么在一块硬盘上运行 `make world` 的同时，可以在其他硬盘上并发进行 Ports 和 `doc` 树的 CVS 检出。使用 RAID 方案（无论是硬件还是软件）都能显著缩短整体构建时间。
- **交叉构建发布版本** —— 想在 x86 硬件上构建 IA-64 或 Alpha 的版本？使用 `make TARGET=ia64 release` 即可。
- **回归测试** —— FreeBSD 需要更完善的自动化正确性测试机制。
- **安装工具** —— 我们的安装程序早就超出其原设计寿命。目前有多个项目正在开发更先进的安装机制。libh 项目便是其中之一，旨在提供一个智能化的新软件包框架与图形化安装程序。

## 8. 致谢

我要感谢 Jordan Hubbard 给我机会承担 FreeBSD 4.4 发布工程中的部分职责，并感谢他多年来为 FreeBSD 发展所做的一切。当然，没有以下几位在发布工作中所做的贡献，本次发布是不可能完成的：Satoshi Asami [asami@FreeBSD.org](mailto:asami@FreeBSD.org)、Steve Price [steve@FreeBSD.org](mailto:steve@FreeBSD.org)、Bruce A. Mah [bmah@FreeBSD.org](mailto:bmah@FreeBSD.org)、Nik Clayton [nik@FreeBSD.org](mailto:nik@FreeBSD.org)、David O’Brien [obrien@FreeBSD.org](mailto:obrien@FreeBSD.org)、Kris Kennaway [kris@FreeBSD.org](mailto:kris@FreeBSD.org)、John Baldwin [jhb@FreeBSD.org](mailto:jhb@FreeBSD.org)，以及整个 FreeBSD 开发社区。我还要感谢 Rodney W. Grimes [rgrimes@FreeBSD.org](mailto:rgrimes@FreeBSD.org)、Poul-Henning Kamp [phk@FreeBSD.org](mailto:phk@FreeBSD.org) 及其他在 FreeBSD 初期为发布工程工具做出贡献的人们。本文也受到了 CSRG[^7]、NetBSD 项目[^8]、以及 John Baldwin 提出的发布工程流程笔记[^9] 的影响。


[^6]: 参见 [Diskless Operation with PXE](https://docs.freebsd.org/en/books/handbook/#network-diskless)

[^7]: Marshall Kirk McKusick、Michael J. Karels 和 Keith Bostic：《The Release Engineering of 4.3BSD》

[^8]: NetBSD 开发文档：Release Engineering [http://www.NetBSD.org/developers/releng/index.html](http://www.netbsd.org/developers/releng/index.html)

[^9]: John Baldwin 的 FreeBSD 发布工程流程建议 [https://people.FreeBSD.org/\~jhb/docs/releng.txt](https://people.freebsd.org/~jhb/docs/releng.txt)


