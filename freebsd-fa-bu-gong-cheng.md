# FreeBSD 发布工程

- 原文：[FreeBSD Release Engineering](https://docs.freebsd.org/en/articles/freebsd-releng/)

## 摘要

本文介绍了 FreeBSD 项目的发布工程过程。



## 1. FreeBSD 发布工程过程介绍

FreeBSD 的开发有一个非常具体的工作流程。一般来说，所有对 FreeBSD 基础系统的更改都会提交到主分支，该分支反映了源代码树的顶部。

经过合理的测试期后，更改可以合并到 `stable/` 分支。合并到 `stable/` 分支的默认最短时间框架是三（3）天。

虽然合并到 `stable/` 分支前通常需要等待至少三天，但在一些特殊情况下，可能需要立即合并，例如关键的安全修复或直接阻碍发布构建过程的 bug 修复。

经过几个月的开发，`stable/` 分支中的更改数量显著增加时，就到了发布下一个版本的 FreeBSD 的时候。这些发布历来被称为 `点` 版本发布。

在 `stable/` 分支发布之间，大约每两（2）年一次，发布将直接从主分支切出。这些发布历来被称为 `点零` 版本发布。

本文将重点介绍 FreeBSD 发布工程团队在 `点零` 和 `点` 发布中的工作流程和责任。

本文的以下章节介绍了：

[一般信息和准备](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-prep)
发布周期开始前的一般信息和准备。

[发布周期中的网站更改](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-website)
发布周期中的网站更改。

[发布工程术语](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-terms)
本文中使用的术语和一般信息，如"代码冻结前的准备阶段"和"代码冻结"。

[从主分支发布](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-head)
`点零` 版本发布的发布工程过程。

[从 `stable/` 分支发布](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-stable)
`点` 版本发布的发布工程过程。

[构建 FreeBSD 安装介质](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-building)
构建安装介质的具体程序。

[将 FreeBSD 安装介质发布到项目镜像](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-mirrors)
发布安装介质的程序。

[完成发布周期](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-wrapup)
完成发布周期。

## 2. 一般信息和准备

在发布周期开始前大约两个月，FreeBSD 发布工程团队会决定发布的时间表。该时间表包括发布周期中的各个里程碑节点，例如冻结日期、分支日期和构建日期。例如：

| 里程碑                                  | 预计日期           |
| --------------------------------------- | ------------------ |
| 主分支代码冻结前的准备阶段 (main slush) | 2016 年 5 月 27 日 |
| 主分支冻结 (main freeze)                | 2016 年 6 月 10 日 |
| 主分支 KBI 冻结 (main KBI freeze)       | 2016 年 6 月 24 日 |
| `doc/` 树冻结前的准备阶段 \[1]             | 2016 年 6 月 24 日 |
| Ports 季度分支 \[2]                     | 2016 年 7 月 1 日  |
| stable/13 分支                          | 2016 年 7 月 8 日  |
| `doc/` 树标签 \[3]                      | 2016 年 7 月 8 日  |
| BETA1 构建开始                          | 2016 年 7 月 8 日  |
| 主分支解冻 (main thaw)                  | 2016 年 7 月 9 日  |
| BETA2 构建开始                          | 2016 年 7 月 15 日 |
| BETA3 构建开始 \[\*]                    | 2016 年 7 月 22 日 |
| releng/13.0 分支                        | 2016 年 7 月 29 日 |
| RC1 构建开始                            | 2016 年 7 月 29 日 |
| stable/13 解冻 (stable/13 thaw)         | 2016 年 7 月 30 日 |
| RC2 构建开始                            | 2016 年 8 月 5 日  |
| 最终 Ports 包构建 \[4]                  | 2016 年 8 月 6 日  |
| Ports 发布标签                          | 2016 年 8 月 12 日 |
| RC3 构建开始 \[\*]                      | 2016 年 8 月 12 日 |
| RELEASE 构建开始                        | 2016 年 8 月 19 日 |
| RELEASE 公告发布                        | 2016 年 9 月 2 日  |


>**注意**
>
> 标记为 `[*]` 的项目是“按需”的。

1. `doc/` 树冻结由 FreeBSD 文档工程团队协调。
2. 使用的 Ports 季度分支由最终的 `RC` 构建计划决定。新的季度分支会在每季度的第一天创建，因此在考虑发布周期的里程碑时应使用该指标。季度分支由 FreeBSD Ports 管理团队创建。
3. `doc/` 树的标签由 FreeBSD 文档工程团队创建。
4. 最终的 Ports 包构建由 FreeBSD Ports 管理团队在最终（或预期为最终的）`RC` 构建之后进行。

>**注意**
>
>如果发布是从现有的 `stable/` 分支创建的，则可以省略 KBI 冻结日期，因为在已经建立的 `stable/` 分支上，KBI 已经被视为冻结。

在编写发布周期的时间表时，需要考虑一些因素，特别是那些目标日期依赖于已定义的里程碑节点的情况。例如，Ports 的发布标签来自于在最后一个 `RC` 时的活跃季度分支。这部分决定了使用哪个季度分支，何时可以进行发布标签，以及用于最终 `RELEASE` 构建的 ports 树的修订版本。

在时间表达成共识后，FreeBSD 发布工程团队会将该时间表通过电子邮件发送给 FreeBSD 开发人员。

许多开发人员通常会通知 FreeBSD 发布工程团队他们正在进行的各种工作。在某些情况下，可能会请求延长正在进行的工作的时间，而在其他情况下，可能会请求对特定子树的 `blanket approval`（通行许可）。

在这些请求中，重要的是确保讨论时间表（即使是估算的）。对于 blanket approval，应该明确许可的时间范围。例如，FreeBSD 开发人员可能会请求从代码冻结前的准备阶段开始到 `RC` 构建开始期间对 **release/doc/** 的 blanket approval。

>**注意**
>
> 为了跟踪 blanket approval，FreeBSD 发布工程团队使用内部仓库来记录这些请求，记录内容包括：授予 blanket approval 的区域、作者、批准过期时间以及批准原因。一个例子是，直到最终的 `RC` 构建，授予 FreeBSD 发布工程团队所有成员对 **release/doc/** 的 blanket approval，以便更新发布说明和其他与发布相关的文档。

>**注意**
>
>FreeBSD 发布工程团队还使用这个仓库来跟踪在发布周期开始各个构建之前收到的待批准请求，发布工程师会通过电子邮件通知 FreeBSD 开发人员批准的截止时间。

根据所涉及的底层代码集及其对 FreeBSD 整体的影响，FreeBSD 发布工程团队可能会批准或拒绝这些请求。

同样适用于正在进行的扩展。例如，对于新设备驱动进行中的工作，如果该工作与树中其他部分没有直接关联，可能会获得延期。然而，新调度程序可能不可行，尤其是当这种剧烈的变化在其他分支中不存在时。

该时间表还会添加到项目网站的 `doc/` 仓库中的 **\~/website/content/en/releases/13.0R/schedule.adoc** 文件中。随着发布周期的进展，此文件会不断更新。

>**注意**
>
>在大多数情况下，**schedule.adoc** 可以从先前的发布中复制并相应更新。

除了将 **schedule.adoc** 添加到网站中，**\~/shared/releases.adoc** 还会更新，以便在各个子页面中添加指向时间表的链接，并使该链接能够在项目网站的主页上显示。

该时间表还会链接到 **\~/website/content/en/releng/\_index.adoc**。

大约在计划的“代码冻结前的准备阶段”前一个月，FreeBSD 发布工程团队会向 FreeBSD 开发人员发送提醒邮件。

## 3. 发布工程术语

本节介绍了本文件中使用的一些术语。

### 3.1. 代码冻结前的准备阶段（Code Slush）

尽管代码冻结前的准备阶段并不是对树的硬性冻结，FreeBSD 发布工程团队要求将现有代码库中的错误修复优先于新功能的开发。

代码冻结前的准备阶段并不强制要求对分支的提交进行审批。

### 3.2. 代码冻结（Code Freeze）

代码冻结标志着一个时间点，从此刻起，所有提交到分支的代码都需要得到 FreeBSD 发布工程团队的明确批准。

FreeBSD Git 仓库包含多个钩子，以在提交到树之前执行有效性检查。其中一个钩子将检查提交到特定分支是否需要特定的批准。

为了执行 FreeBSD 发布工程团队的提交批准，发布工程团队必须批准对该分支的任何更改，此时提交日志中必须包括一行 `Approved by: re (login)`，其中 "login" 是批准人的登录 ID。

>**注意**
>
>在代码冻结期间，FreeBSD 提交者应遵循 [变更请求指南](https://wiki.freebsd.org/Releng/ChangeRequestGuidelines)。

### 3.3. KBI/KPI 冻结（KBI/KPI Freeze）

KBI/KPI 稳定性意味着，在实现同一函数的两个不同版本的软件中调用该函数时，会产生相同的最终状态。调用方（无论是进程、线程还是函数）都期望该函数以特定方式运行，否则分支上的 KBI/KPI 稳定性就会被破坏。

## 4. 发布周期中的网站更改

本节介绍了在发布周期进行过程中，网站应进行的更改。

>**注意**
>
>本节中指定的文件是相对于 `doc` 仓库的 `main` 分支的。

### 4.1. 发布周期开始前的网站更改

当发布周期时间表可用时，需要更新以下文件以启用 FreeBSD 项目网站上的不同功能：

| 文件待编辑                       | 需要更改的内容                                    |
| --------------------------- | ------------------------------------------ |
| **\~/shared/releases.adoc** | 将 `beta-upcoming` 从 `IGNORE` 更改为 `INCLUDE` |
| **\~/shared/releases.adoc** | 将 `beta-testing` 从 `IGNORE` 更改为 `INCLUDE`  |

### 4.2. 在 `BETA` 或 `RC` 阶段的网站更改

从 `PRERELEASE` 过渡到 `BETA` 时，需要更新以下文件以启用下载页面上的“帮助测试”模块。所有文件相对于 **head/** 在 `doc` 仓库中：

| 文件待编辑                                                | 需要更改的内容                              |
| ---------------------------------------------------- | ------------------------------------ |
| **\~/shared/releases.adoc**                          | 将 `betarel-vers` 更新为 `BETA1`         |
| **\~/website/data/en/news/news.toml**                | 添加一条宣布 `BETA` 的条目                    |
| **\~/website/static/security/advisory-template.txt** | 将新的 `BETA`、`RC` 或最终 `RELEASE` 添加到模板中 |
| **\~/website/static/security/errata-template.txt**   | 将新的 `BETA`、`RC` 或最终 `RELEASE` 添加到模板中 |

创建了 `releng/13.0` 分支之后，相关的发布文档需要添加到 `doc/` 仓库中。

>**注意**
>
>相关的发布文档存在于 FreeBSD 12.x 及以后的 **doc** 仓库中。

### 4.3. 在 `BETA`、`RC` 和最终 `RELEASE` 期间的 Ports 更改

在发布周期中的每个构建过程中，包含各种分发集的 `SHA256` 值的 `MANIFEST` 文件（如 `base.txz`、`kernel.txz` 等）将被添加到 Port [misc/freebsd-release-manifests](https://cgit.freebsd.org/ports/tree/misc/freebsd-release-manifests/) 中。这使得除了 `bsdinstall[8]` 之外的工具，如 [ports-mgmt/poudriere](https://cgit.freebsd.org/ports/tree/ports-mgmt/poudriere/)，能够安全地使用这些分发集，通过提供一个机制来验证校验和。

## 5. 从 main 发布

本节介绍了从 main 分支进行 FreeBSD 发布周期的一般程序。

### 5.1. FreeBSD “ALPHA” 构建

从 FreeBSD 10.0-RELEASE 开始，引入了“ALPHA”构建的概念。与 `BETA` 和 `RC` 构建不同，`ALPHA` 构建不包含在 FreeBSD 发布计划中。

`ALPHA` 构建的目的是在创建 `stable/` 分支之前提供定期的 FreeBSD 提供的构建。

FreeBSD `ALPHA` 快照应该大约每周构建一次。

对于第一次 `ALPHA` 构建，需要将 **sys/conf/newvers.sh** 中的 `BRANCH` 值从 `CURRENT` 更改为 `ALPHA1`。对于随后的每个 `ALPHA` 构建，将每个 `ALPHAN` 值增加 1。

有关构建 `ALPHA` 镜像的信息，请参阅 [构建 FreeBSD 安装介质](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-building)。

### 5.2. 创建 `stable/13` 分支

在创建 `stable/` 分支时，既需要对新的 `stable/` 分支进行更改，也需要对 `main` 分支进行更改。以下列出的文件是相对于仓库根目录的。要在 Git 中创建新的 `stable/13` 分支：

>**注意**
>
>确保你在 `main` 分支中

```sh
% git checkout -b stable/13
```

创建 stable/13 分支后，进行以下编辑：

| 文件待编辑                                                     | 需要更改的内容                                                                     |
| --------------------------------------------------------- | --------------------------------------------------------------------------- |
| **UPDATING**                                              | 更新 FreeBSD 版本，移除关于 `WITNESS` 的说明                                            |
| **contrib/jemalloc/include/jemalloc/jemalloc\_FreeBSD.h** | <pre>#ifndef MALLOC_PRODUCTION<br>#define MALLOC_PRODUCTION<br>#endif</pre>|
| **lib/clang/llvm.build.mk**                               | 取消注释 `-DNDEBUG`                                                             |
| **sys/\*/conf/GENERIC\***                                 | 移除调试支持                                                                      |
| **sys/\*/conf/MINIMAL**                                   | 移除调试支持                                                                      |
| **release/release.conf.sample**                           | 更新 `SRCBRANCH`                                                              |
| **sys/\*/conf/GENERIC-NODEBUG**                           | 移除这些内核配置                                                                    |
| **sys/arm/conf/std.arm\***                                | 移除调试选项                                                                      |
| **sys/conf/newvers.sh**                                   | 更新 `BRANCH` 值以反映 `BETA1`                                                    |
| **share/mk/src.opts.mk**                                  | 将 `REPRODUCIBLE_BUILD` 从 `__DEFAULT_NO_OPTIONS` 移动到 `__DEFAULT_YES_OPTIONS` |
| **share/mk/src.opts.mk**                                  | 将 `LLVM_ASSERTIONS` 从 `__DEFAULT_YES_OPTIONS` 移动到 `__DEFAULT_NO_OPTIONS`    |
| **libexec/rc/rc.conf**                                    | 将 `dumpdev` 设置为 `NO`，而不是 `AUTO`（对于那些希望默认启用的用户，可以通过配置来启用）                    |
| **release/Makefile**                                      | 移除 `debug.witness.trace` 条目                                                 |

然后，在现在将成为新主版本的 `main` 分支中：

| 文件待编辑                                         | 需要更改的内容                                   |
| --------------------------------------------- | ----------------------------------------- |
| **UPDATING**                                  | 更新 FreeBSD 版本                             |
| **sys/conf/newvers.sh**                       | 更新 `BRANCH` 值以反映 `CURRENT`，并增加 `REVISION` |
| **Makefile.inc1**                             | 更新 `TARGET_TRIPLE` 和 `MACHINE_TRIPLE`     |
| **sys/sys/param.h**                           | 更新 `__FreeBSD_version`                    |
| **gnu/usr.bin/cc/cc\_tools/freebsd-native.h** | 更新 `FBSD_MAJOR` 和 `FBSD_CC_VER`           |
| **contrib/gcc/config.gcc**                    | 追加 `freebsdversion.h` 部分                  |
| **lib/clang/llvm.build.mk**                   | 更新 `OS_VERSION` 的值                        |
| **lib/clang/freebsd\_cc\_version.h**          | 更新 `FREEBSD_CC_VERSION`                   |
| **lib/clang/include/lld/Common/Version.inc**  | 更新 `LLD_REVISION_STRING`                  |
| **Makefile.libcompat**                        | 更新 `LIB32CPUFLAGS`                        |

## 6. 从 `stable/` 分支发布

本节介绍了 FreeBSD 发布周期的基本流程，从一个已建立的 `stable/` 分支开始。

### 6.1. FreeBSD `stable` 分支代码冻结前的准备阶段

在 `stable` 分支的代码冻结之前，需要更新几个文件，以反映发布周期的正式开始。这些文件都位于 stable 分支的最顶层：

| 要编辑的文件                      | 需要更改的内容                     |
| --------------------------- | --------------------------- |
| **sys/conf/newvers.sh**     | 更新 `BRANCH` 值为 `PRERELEASE` |
| **Makefile.inc1**           | 更新 `TARGET_TRIPLE`          |
| **lib/clang/llvm.build.mk** | 更新 `OS_VERSION`             |
| **Makefile.libcompat**      | 更新 `LIB32CPUFLAGS`          |

### 6.2. FreeBSD `BETA` 构建

在代码冻结前的准备阶段之后，发布周期的下一阶段是代码冻结。这时，所有对 stable 分支的提交都需要 FreeBSD 发布工程团队的明确批准。此过程由 [gitadm@FreeBSD.org](mailto:gitadm@FreeBSD.org) 执行，负责管理仓库。

>**注意**
>
>在发布周期中，有两个通常的例外情况不需要提交批准。第一个是任何发布工程师需要提交的更改，以便继续发布周期的日常工作流，另一个是发布周期中可能出现的安全修复。

代码冻结生效后，下一次从该分支构建的版本被标记为 `BETA1`。这通过在 **sys/conf/newvers.sh** 中将 `BRANCH` 值从 `PRERELEASE` 更新为 `BETA1` 来完成。

完成此操作后，第一组 `BETA` 构建开始。随后的 `BETA` 构建只需更新 **sys/conf/newvers.sh** 中的 `BETA` 构建编号，不需要更新其他文件。

### 6.3. 创建 `releng/13.0` 分支

当第一个 `RC`（发布候选版）构建准备好开始时，releng/ 分支将被创建。这是一个多步骤的过程，必须按特定顺序进行，以避免例如与 `__FreeBSD_version` 值重叠等异常情况。以下路径相对于仓库根目录。提交的顺序和需要更改的内容如下：

> **注意**
>
> 请确保你处于 `stable/13` 分支

```sh
% git checkout -b releng/13.0
```

| 要编辑的文件                                  | 需要更改的内容                                                                |
| --------------------------------------- | ---------------------------------------------------------------------- |
| **sys/conf/newvers.sh**                 | 将 `BETAX` 更改为 `RC1`                                                    |
| **sys/sys/param.h**                     | 更新 `__FreeBSD_version`                                                 |
| **sys/conf/kern.opts.mk**               | 将 `REPRODUCIBLE_BUILD` 从 `__DEFAULT_NO_OPTIONS` 移到 `__DEFAULT_YES_OPTIONS` |
| **etc/pkg/FreeBSD.conf**                | 将 `latest` 替换为 `quarterly` 作为默认的包仓库位置                                  |
| **release/pkg\_repos/release-dvd.conf** | 将 `latest` 替换为 `quarterly` 作为默认的包仓库位置                                  |
| **sys/conf/newvers.sh**                 | 更新 `BETAX` 为 `PRERELEASE`                                              |
| **sys/sys/param.h**                     | 更新 `__FreeBSD_version`                                                 |

然后，[gitadm@FreeBSD.org](mailto:gitadm@FreeBSD.org) 会为 releng 分支添加新的批准者，就像之前为 stable 分支所做的那样。

```sh
% git add .
% git commit
```

现在已经存在两个新的 `__FreeBSD_version` 值，接着更新文档项目仓库中的 **\~/documentation/content/en/books/porters-handbook/versions/chapter.adoc** 文件。

在第一个 `RC` 构建完成并测试后，`stable/` 分支可以由 [gitadm@FreeBSD.org](mailto:gitadm@FreeBSD.org) 进行“解冻”。

在第一个 `RC` 可用后，应向 FreeBSD Bugmeister Team 发送电子邮件，要求将新的 FreeBSD `-RELEASE` 添加到 bug 跟踪器中下拉菜单的 `versions` 列表中。

## 7. 构建 FreeBSD 安装介质

本节介绍了生成 FreeBSD 开发快照和发布版本的基本流程。

### 7.1. 发布构建脚本

在 FreeBSD 9.0-RELEASE 之前，**src/release/Makefile** 已更新以支持 `bsdinstall[8]`，并且引入了 **src/release/generate-release.sh** 脚本，作为自动调用 `release[7]` 目标的包装器。

在 FreeBSD 9.2-RELEASE 之前，**src/release/release.sh** 被引入，该脚本在很大程度上基于 **src/release/generate-release.sh**，并包括支持指定配置文件以覆盖各种选项和环境变量的功能。配置文件的支持使得通过为每次调用指定一个单独的配置文件来支持交叉构建每个架构的发布版本。

以下是使用 **src/release/release.sh** 构建单个发布版本并放置于 **/scratch** 的简短示例：

```sh
# /bin/sh /usr/src/release/release.sh
```

以下是使用 **src/release/release.sh** 构建单个交叉构建的发布版本并使用不同的目标目录的简短示例，首先创建一个自定义的 **release.conf** 配置文件，其中包含：

```sh
# release.sh 配置用于 powerpc/powerpc64
CHROOTDIR="/scratch-powerpc64"
TARGET="powerpc"
TARGET_ARCH="powerpc64"
KERNEL="GENERIC64"
```

然后使用以下命令调用 **src/release/release.sh**：

```sh
# /bin/sh /usr/src/release/release.sh -c $HOME/release.conf
```

有关更多详细信息和示例用法，请参见 `release[7]` 和 **src/release/release.conf.sample**。

### 7.2. 构建 FreeBSD 发布版本

在发布周期中，每个架构的 **CHECKSUM.SHA512** 和 **CHECKSUM.SHA256** 副本将存储在 FreeBSD 发布工程团队的内部仓库中，并且也会包含在各种发布公告电子邮件中。每个 **MANIFEST**，其中包含 **base.txz**、**kernel.txz** 等文件的哈希值，也会添加到 Ports Collection 中的 [misc/freebsd-release-manifests](https://cgit.freebsd.org/ports/tree/misc/freebsd-release-manifests/)。

在准备发布构建时，需要更新以下几个文件：

| 要编辑的文件                        | 需要更改的内容                                           |
| ----------------------------- | ------------------------------------------------- |
| **sys/conf/newvers.sh**       | 更新 `BRANCH` 值为 `RELEASE`                          |
| **UPDATING**                  | 添加预期的公告日期                                         |
| **lib/csu/common/crtbrand.S** | 将 `__FreeBSD_version` 替换为 **sys/sys/param.h** 中的值 |

在构建完成最终的 `RELEASE` 后，`releng/13.0` 分支会被标记为 `release/13.0.0`，使用构建 `RELEASE` 时的修订版本。与创建 `stable/13` 和 `releng/13.0` 分支类似，使用 `git tag` 来完成此操作。从仓库根目录执行：

  >**注意**
  >
  >确保你处于 `releng/13.0` 分支

```sh
% git tag release/13.0.0
```

## 8. 将 FreeBSD 安装介质发布到项目镜像

本节介绍了将 FreeBSD 开发快照和发布版本发布到项目镜像的流程。

### 8.1. 临时存储 FreeBSD 安装介质镜像

临时存储 FreeBSD 快照和发布版本是一个两步过程：

- 创建目录结构以匹配 `ftp-master` 上的层次结构
  如果在构建配置文件中定义了 `EVERYTHINGISFINE`，例如在上面提到的构建脚本的 **main.conf** 中，这将在构建完成后在 `chroot[8]` 中自动发生，创建一个路径结构匹配 `ftp-master` 上预期的目录结构，存放在 **\${DESTDIR}/R/ftp-stage**。这相当于在 `chroot[8]` 中直接运行以下命令：

  ```sh
  # make -C /usr/src/release -f Makefile.mirrors EVERYTHINGISFINE=1 ftp-stage
  ```

  在每个架构构建完成后，**thermite.sh** 将通过 rsync 将构建中的 **\${DESTDIR}/R/ftp-stage** 从构建主机同步到 **/snap/ftp/snapshots** 或 **/snap/ftp/releases**。

- 在将文件移至 **pub/** 以开始传播到项目镜像之前，将文件复制到 `ftp-master` 上的临时存储目录
  所有构建完成以后，**/snap/ftp/snapshots** 或发布版本的 **/snap/ftp/releases** 将通过 rsync 被 `ftp-master` 拉取到 **/archive/tmp/snapshots** 或 **/archive/tmp/releases**。

  >**注意**
  >
  >在 FreeBSD 项目基础设施中的 `ftp-master` 上，这一步需要 `root` 级别的访问权限，因为此步骤必须以 `archive` 用户身份执行。

### 8.2. 发布 FreeBSD 安装介质

镜像在 **/archive/tmp/** 中暂存后，它们就准备好通过将其放入 **/archive/pub/FreeBSD** 来公开发布。为了减少传播时间，使用 `pax[1]` 创建从 **/archive/tmp** 到 **/archive/pub/FreeBSD** 的硬链接。

>**注意**
>
>为了有效执行此操作，**/archive/tmp** 和 **/archive/pub** 必须位于同一逻辑文件系统上。

然而有一个注意事项，必须在 `pax[1]` 之后使用 rsync 来修正 **pub/FreeBSD/snapshots/ISO-IMAGES** 中的符号链接，`pax[1]` 会将这些符号链接替换为硬链接，从而增加传播时间。

>**注意**
>
>与暂存步骤一样，这需要 `root` 级别的访问权限，因为此步骤必须以 `archive` 用户身份执行。

作为 `archive` 用户执行：

```sh
% cd /archive/tmp/snapshots
% pax -r -w -l . /archive/pub/FreeBSD/snapshots
% /usr/local/bin/rsync -avH /archive/tmp/snapshots/* /archive/pub/FreeBSD/snapshots/
```

根据需要，将 *snapshots* 替换为 *releases*。

## 9. 发布周期的收尾工作

本节介绍了发布后的常规任务。

### 9.1. 发布后勘误通知

随着发布周期的结束，通常会有几个勘误通知（Errata Notice）候选项，以解决在周期末期发现的问题。在发布之后，FreeBSD 发布工程团队和 FreeBSD 安全团队会重新审视在最终发布前未批准的更改，根据问题的范围，可能会发布勘误通知。

>**注意**
>
> 实际发布勘误通知的过程由 FreeBSD 安全团队处理。

要请求发布后勘误通知，开发人员应填写 [勘误通知模板](https://www.freebsd.org/security/errata-template.txt)，特别是 `Background`、`Problem Description`、`Impact`，以及（如适用）`Workaround` 部分。

完成的勘误通知模板应通过电子邮件与针对 `releng/` 分支的补丁或来自 `stable/` 分支的修订列表一起发送。

对于发布后立即请求勘误通知，应将请求同时发送给 FreeBSD 发布工程团队和 FreeBSD 安全团队。`releng/` 分支已根据 [交接到 FreeBSD 安全团队](https://docs.freebsd.org/en/articles/freebsd-releng/#releng-wrapup-handoff) 的说明交接给 FreeBSD 安全团队之后，应发送勘误通知请求给 FreeBSD 安全团队。

### 9.2. 交接到 FreeBSD 安全团队

在发布后的大约两周，发布工程师更新 Git 仓库，将 `releng/13.0` 分支的审批人从发布工程团队更改为安全官。

## 10. 发布生命周期结束

本节描述当发布版本达到生命周期结束（EoL）时，需要更新的与网站相关的文件。

### 10.1. 生命周期结束的网站更新

当一个发布版本达到生命周期结束时，应该在网站上删除和/或更新对该发布版本的引用：

| 文件                                                          | 需要更改的内容                                           |
| ----------------------------------------------------------- | ------------------------------------------------- |
| **\~/website/themes/beastie/layouts/index.html**            | 删除 `u-relXXX-announce` 和 `u-relXXX-announce` 的引用。 |
| **\~/website/content/en/releases/\_index.adoc**             | 将 `u-relXXX-*` 变量从支持的发布版本列表移动到遗留版本列表。             |
| **\~/website/content/en/releng/\_index.adoc**               | 更新相应的 releng 分支，表明该分支不再支持。                        |
| **\~/website/content/en/security/\_index.adoc**             | 从支持的分支列表中删除该分支。                                   |
| **\~/website/content/en/security/unsupported.adoc**         | 将该分支添加到不受支持的分支列表。                                 |
| **\~/website/content/en/where.adoc**                        | 删除该发布版本的 URL。                                     |
| **\~/website/themes/beastie/layouts/partials/sidenav.html** | 删除 `u-relXXX-announce` 和 `u-relXXX-announce` 的引用。 |
| **\~/website/static/security/advisory-template.txt**        | 删除与该发布版本和 releng 分支相关的引用。                         |
| **\~/website/static/security/errata-template.txt**          | 删除与该发布版本和 releng 分支相关的引用。                         |

