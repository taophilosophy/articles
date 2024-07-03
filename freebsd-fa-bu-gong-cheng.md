# FreeBSD 发布工程

<details open="" data-immersive-translate-walked="98272659-72e6-4d59-b2dd-08cbeaddb9dc"><summary data-immersive-translate-walked="98272659-72e6-4d59-b2dd-08cbeaddb9dc" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

Git 和 Git 标志是 Software Freedom Conservancy, Inc. 的注册商标或商标，Git 项目的公司总部在美国和/或其他国家。

英特尔， 赛扬，赛睿，酷睿，EtherExpress，i386，i486，Itanium，奔腾 和 至强 是英特尔公司或其子公司在美国和其他国家的商标或注册商标。

制造商和销售商用于区分其产品的许多名称被视为商标。 在本文档中出现这些名称时，如果 FreeBSD 项目知悉了商标声明，则这些名称后面跟有“™”或“®”符号。

</details>

 摘要

本文描述了 FreeBSD 项目的发布工程过程。

---

## 1. FreeBSD 发布工程过程简介

FreeBSD 的开发有着非常具体的工作流程。一般来说，所有对 FreeBSD 基础系统的更改都提交到主分支，这反映了源代码树的顶部。

经过合理的测试期后，变更才能合并到 stable/ 分支。合并到 stable/ 分支的默认最短时间段为三（3）天。

尽管一般规则要求在从主分支合并之前等待至少三天，但也有一些特殊情况，如关键安全修复或直接阻碍发布构建流程的错误修复，可能需要立即合并。

几个月后，stable/ 分支中的变更数量显著增加，是时候发布下一个版本的 FreeBSD 了。这些发布历史上被称为“点”版本。

在稳定分支发布之间，大约每两年会直接从 main 分支创建一个版本。这些版本历史上被称为“点零”版本。

这篇文章将重点介绍 FreeBSD 发布工程团队在“点零”版本和“点”版本中的工作流程和职责。

文章的以下部分描述了：

一般信息和准备在开始发布周期之前的一般信息和准备。

发布周期内的网站更改发布周期内的网站更改。

发布工程术语术语和一般信息，如“代码冻结期”和“代码冻结期”，在本文档中使用。

发行主要版本的发布工程过程。

发行稳定版本的发布工程过程。

构建 FreeBSD 安装媒体的相关信息，包括特定的构建过程。

将 FreeBSD 安装媒体发布到项目镜像发布安装介质的过程。

包装发布周期结束包装发布周期。

## 2. 一般信息和准备

在发行周期开始约两个月前，FreeBSD 发布工程团队会制定一个发布计划。该计划包括发布周期的各个重要节点，如冻结日期、分支日期和构建日期。例如：

| 里程碑                     | 预计日期           |
| ---------------------------- | -------------------- |
| 主要的冰雪:                | 2016 年 5 月 27 日 |
| 主要冷冻:                  | 2016 年 6 月 10 日 |
| 主要 KBI 冻结：            | 2016 年 6 月 24 日 |
| doc/ 树汁 [1]:             | 2016 年 6 月 24 日 |
| Ports 季度分支 [2]:        | 2016 年 7 月 1 日  |
| 稳定版本 13 分支:          | 2016 年 7 月 8 日  |
| doc/ 树标签[3]:            | 2016 年 7 月 8 日  |
| BETA1 构建开始:            | 2016 年 7 月 8 日  |
| 主要解冻：                 | 2016 年 7 月 9 日  |
| BETA2 构建开始：           | 2016 年 7 月 15 日 |
| BETA3 构建开始[*]：        | 2016 年 7 月 22 日 |
| releng/13.0 分支:          | 2016 年 7 月 29 日 |
| RC1 构建开始：             | 2016 年 7 月 29 日 |
| 稳定/13 解冻:              | 2016 年 7 月 30 日 |
| RC2 构建开始：             | 2016 年 8 月 5 日  |
| 最终 Ports 软件包构建 [4]: | 2016 年 8 月 6 日  |
| Ports 发行标签:            | 2016 年 8 月 12 日 |
| RC3 构建开始 [*]:          | 2016 年 8 月 12 日 |
| 发布构建开始:              | 2016 年 8 月 19 日 |
| 发布公告:                  | 2016 年 9 月 2 日  |

|  | 标有“[*]”的项目为“根据需要”。 |
| -- | ----------------------------------- |

1. doc/ 树冰泥由 FreeBSD 文档工程团队协调。
2. 所使用的Ports季度分支由最终 RC 构建计划的时间决定。新的季度分支在季度的第一天创建，因此在考虑发布周期里程碑时应使用此指标。季度分支由 FreeBSD Ports管理团队创建。
3. doc/ 树由 FreeBSD 文档工程团队标记。
4. 最终的Ports软件包构建在最终（或预期为最终） RC 构建之后由 FreeBSD Ports管理团队完成。

|  | 如果发布是从现有的稳定分支创建的，可以排除 KBI 冻结日期，因为在已建立的稳定分支上，KBI 已经被认为是冻结的。 |
| -- | ------------------------------------------------------------------------------------------------------------- |

在编写发布周期时间表时，需要考虑许多事项，特别是目标日期依赖于预定义的里程碑，这些里程碑具有依赖性。例如，Ports集合发布标记源自最后一次活跃季度分支时的活跃季度分支。这在某种程度上定义了使用哪个季度分支，发布标记可以何时发生，以及最终ports树的哪个修订版本用于最终 RELEASE 构建。

在时间表获得一般同意后，FreeBSD 发布工程团队会将时间表发送给 FreeBSD 开发者。

许多开发者通常会告知 FreeBSD 发布工程团队各种正在进行中的工作。在某些情况下，可能会请求延长正在进行中工作的时间，而在其他情况下，可能会请求对树的特定子集进行“全面批准”。

当出现此类请求时，重要的是确保讨论时间表（即使是估计的）。对于全面批准，应明确指定全面批准的时间长度。例如，FreeBSD 开发者可能会请求从代码准备期开始到 RC 构建开始的全面批准。

|  | 为了跟踪全面批准情况，FreeBSD 发布工程团队使用内部知识库记录这类请求，其中定义了授予全面批准的区域、作者、全面批准到期时间以及授予批准的原因。一个例子是授予所有 FreeBSD 发布工程团队成员对发布/doc/的全面批准，以更新发布说明和其他与发布相关的文档直至最终 RC 。 |
| -- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

|  | FreeBSD 发布工程团队还使用此存储库来跟踪在发布周期开始之前收到的待批准请求，发布工程师会通过电子邮件向 FreeBSD 开发人员指定截止期限。 |
| -- | --------------------------------------------------------------------------------------------------------------------------------------- |

根据涉及的代码集和代码集对整个 FreeBSD 的总体影响，FreeBSD 发布工程团队可能会批准或拒绝这些请求。

对正在进行的扩展也适用相同的规则。例如，与树的其余部分隔离开的新设备驱动程序的正在开发工作可能会获得延期。然而，对于新调度程序，可能不可行，尤其是如果这样的重大更改在其他分支中不存在。

该时间表也已添加到项目网站，在 doc/ 存储库中的~/website/content/en/releases/13.0R/schedule.adoc。随着发布周期的进展，该文件将不断更新。

|  | 在大多数情况下，schedule.adoc 可以从先前的版本中复制并相应更新。 |
| -- | ------------------------------------------------------------------ |

除了在网站上添加 schedule.adoc 之外，还会更新~/shared/releases.adoc，以在各个子页面上添加到时间表的链接，并在项目网站索引页面上启用与时间表的链接。

该时间表也链接到 ~/website/content/en/releng/_index.adoc。

大约在预定的“代码冻结”前一个月，FreeBSD 发布工程团队会向 FreeBSD 开发者发送提醒邮件。

## 3. 发布工程术语

本节描述了本文档其余部分使用的一些术语。

### 3.1. 代码冰期

尽管代码冰期并不是对树的硬冻结，但 FreeBSD 发布工程团队要求现有代码库中的错误优先于新功能。

代码冻结前不强制要求提交批准到分支。

### 3.2. 代码冻结

代码冻结标志着所有提交到分支的代码都需要 FreeBSD 发布工程团队的明确批准。

FreeBSD Git 仓库包含多个钩子，在提交到树之前执行完整性检查。其中一个钩子将评估是否需要特定批准才能提交到特定分支。

为了强制执行由 FreeBSD 发布工程团队批准的提交，发布工程团队必须批准对分支的任何更改。在这种情况下，提交日志必须包含一个 Approved by: re (login) 行，其中 "login" 是批准者的登录 ID。

|  | 在代码冻结期间，FreeBSD 提交者被敦促遵循变更请求指南。 |
| -- | -------------------------------------------------------- |

### 3.3. 内核二进制接口（KBI）/内核编程接口（KPI）冻结

KBI/KPI 的稳定性意味着跨两个实现函数的软件版本之间调用者的结束状态相同。 无论调用者是进程、线程还是函数，都期望函数以某种方式运行，否则分支上的 KBI/KPI 稳定性将被破坏。

## 4. 发布周期中的网站更改

这个部分描述了随着发布周期的进展应该发生的网站变化。

|  | 本部分指定的文件都是相对于 main 代码库的 doc 分支的。 |
| -- | ------------------------------------------------------- |

### 4.1. 在发布周期开始之前的网站变化

当发布周期计划可用时，需要更新这些文件，以在 FreeBSD 项目网站上启用不同的功能：

| 编辑的文件 | 修改内容                                |
| ------------ | ----------------------------------------- |
| `~/shared/releases.adoc`           | 修改 beta-upcoming 从 IGNORE 到 INCLUDE |
| `~/shared/releases.adoc`           | 修改 beta-testing 从 IGNORE 到 INCLUDE  |

### 4.2. BETA 或 RC 期间的网站更改

当从 PRERELEASE 过渡到 BETA 时，需要更新这些文件以在下载页面上启用“帮助测试”块。所有文件都相对于 doc 存储库中的 head/ ：

| 编辑文件 | 要更改内容                                      |
| ---------- | ------------------------------------------------- |
| `~/shared/releases.adoc`         | 将 betarel-vers 更新为 BETA<em>1</em>           |
| `~/website/data/en/news/news.toml`         | 添加一条公告 BETA                               |
| `~/website/static/security/advisory-template.txt`         | 将新的 BETA 、 RC 或最终的 RELEASE 添加到模板中 |
| `~/website/static/security/errata-template.txt`         | 将新的 BETA ， RC 或最终的 RELEASE 添加到模板中 |

一旦创建 releng/13.0 分支，各种与发布相关的文件需要添加到 doc/ 存储库中

|  | 有关 FreeBSD 12.x 及以后版本，相关的与发布相关的文件存储在 doc 存储库中 |
| -- | ------------------------------------------------------------------------- |

### 4.3. Ports 更改期间 BETA ， RC 和最终 RELEASE

对于发布周期中的每个构建，包含各种发行版集的 MANIFEST 文件，例如 base.txz ， kernel.txz 等，都会添加到 misc/freebsd-release-manifests port。这允许除 ports-mgmt/poudriere 之外的其他实用程序通过提供可验证校验和的机制来安全地使用这些发行版集。

## 5. 从主要版本发布

本节描述了 FreeBSD 发布周期的一般程序，从主分支开始。

### 5.1. FreeBSD“ALPHA”版本

从 FreeBSD 10.0-RELEASE 循环开始，引入了“ALPHA”版本的概念。与 BETA 和 RC 版本不同， ALPHA 版本不包含在 FreeBSD 发布计划中。

ALPHA 构建背后的想法是在创建 stable/ 分支之前提供定期的 FreeBSD 提供的构建。

每周应构建一次 FreeBSD ALPHA 快照。

对于第一个 ALPHA 构建，需要将 sys/conf/newvers.sh 中的 CURRENT 值从 CURRENT 更改为 ALPHA1 。对于后续的 ALPHA 构建，逐个递增每个 ALPHA<em>N</em> 值。

查看构建 FreeBSD 安装媒体的信息，了解如何构建 ALPHA 镜像。

### 5.2. 创建稳定/13 分支

创建稳定/分支时，在新的稳定/13 分支和主分支中需要进行若干更改。列出的文件是相对于存储库根目录的。要在 Git 中创建新的稳定/13 分支：

|  | 确保您在主分支上 |
| -- | ------------------ |

```
% git checkout -b stable/13
```

一旦创建了 stable/13 分支，请进行以下编辑：

| 要编辑的文件 | 如何修改                                                                      |
| -------------- | ------------------------------------------------------------------------------- |
| UPDATING     | 更新 FreeBSD 版本，并移除关于 WITNESS 的通知                                  |
| `contrib/jemalloc/include/jemalloc/jemalloc_FreeBSD.h`             | `#ifndef MALLOC_PRODUCTION`<br />`#define MALLOC_PRODUCTION`<br />`#endif`                                                                          |
| `lib/clang/llvm.build.mk`             | 取消注释 -DNDEBUG                                                             |
| `sys/*/conf/GENERIC*`             | 删除调试支持                                                                  |
| `sys/*/conf/MINIMAL`             | 删除调试支持                                                                  |
| `release/release.conf.sample`             | 更新 SRCBRANCH                                                                |
| `sys/*/conf/GENERIC-NODEBUG`             | 删除这些内核配置                                                              |
| `sys/arm/conf/std.arm*`             | 删除调试选项                                                                  |
| `sys/conf/newvers.sh`             | 更新 BRANCH 值以反映 BETA1                                                    |
| `share/mk/src.opts.mk`             | 将 REPRODUCIBLE_BUILD 从 __DEFAULT_NO_OPTIONS 移动到 __DEFAULT_YES_OPTIONS    |
| `share/mk/src.opts.mk`             | 将 LLVM_ASSERTIONS 从 __DEFAULT_YES_OPTIONS 移动到 __DEFAULT_NO_OPTIONS       |
| `libexec/rc/rc.conf`             | 将 dumpdev 从 AUTO 设置为 NO （它是可配置的，可以为那些希望默认启用的人启用） |
| `release/Makefile`             | 删除 debug.witness.trace 条目                                                 |

然后在主分支中，这将成为一个新的主要版本：

| 要编辑的文件 | 要更改什么                                             |
| -------------- | -------------------------------------------------------- |
| `UPDATING`             | 更新 FreeBSD 版本                                      |
| `sys/conf/newvers.sh`             | 将 BRANCH 值更新为反映 CURRENT 的值，然后递增 REVISION |
| `Makefile.inc1`             | 更新 TARGET_TRIPLE 和 MACHINE_TRIPLE                   |
| `sys/sys/param.h`             | 更新 __FreeBSD_version                                 |
| `gnu/usr.bin/cc/cc_tools/freebsd-native.h`             | 更新 FBSD_MAJOR 和 FBSD_CC_VER                         |
| `contrib/gcc/config.gcc`             | 追加 freebsdversion.h 部分                             |
| `lib/clang/llvm.build.mk`             | 更新 OS_VERSION 的值                                   |
| `lib/clang/freebsd_cc_version.h`             | 更新 FREEBSD_CC_VERSION                                |
| `lib/clang/include/lld/Common/Version.inc`             | 更新 LLD_REVISION_STRING                               |
| `Makefile.libcompat`             | 更新 LIB32CPUFLAGS                                     |

## 6. 从稳定版发布

本节描述了 FreeBSD 发布周期的一般流程，从已建立的 stable/分支开始。

### 6.1. FreeBSD stable 分支代码冰封

为准备在 stable 分支上冻结代码做准备，需要更新多个文件以反映发布周期已正式开始。这些文件都相对于稳定分支的最顶层目录。

| 要编辑的文件 | 需要更改的内容                    |
| -------------- | ----------------------------------- |
| `sys/conf/newvers.sh`             | 将 BRANCH 值更新为反映 PRERELEASE |
| `Makefile.inc1`             | 更新 TARGET_TRIPLE                |
| `lib/clang/llvm.build.mk`             | 更新 OS_VERSION                   |
| `Makefile.libcompat`             | 更新 LIB32CPUFLAGS                |

### 6.2. FreeBSD `BETA` Builds

在代码冰冻后，发布周期的下一阶段是代码冻结。这是所有提交到稳定分支都需要自 FreeBSD 发布工程团队明确批准的时点。这由 gitadm@FreeBSD.org 负责处理仓库。

|  | 在发布周期中，有两个一般例外情况不需要提交批准。第一个是任何需要由发布工程师提交以继续发布周期日常工作流程的更改，另一个是在发布周期内可能发生的安全修复。 |
| -- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |

一旦代碼凍結生效，從分支生成的下一個構建將被標記為 BETA1 。這是通過將 sys/conf/newvers.sh 中的 BRANCH 值從 PRERELEASE 更新為 BETA1 來完成的。

一旦這個完成，將啟動第一組 BETA 構建。後續的 BETA 構建不需要更新任何文件，除了 sys/conf/newvers.sh，將 BETA 構建版本號進行增加。

### 6.3. 創建 releng/13.0 分支

当第一个 RC （候选版本）构建准备开始时，将创建 releng/分支。这是一个多步骤过程，必须按特定顺序执行，以避免出现与 __FreeBSD_version 值重叠等异常情况。下面列出的路径是相对于存储库根目录的。提交的顺序和要更改的内容为：

|  | 确保您位于 stable/13 分支 |
| -- | --------------------------- |

```
% git checkout -b releng/13.0
```

| 要编辑的文件 | 变更什么                                                  |
| -------------- | ----------------------------------------------------------- |
| `sys/conf/newvers.sh`             | 将 BETA<em>X</em> 更改为 RC1                              |
| `sys/sys/param.h`             | 更新 __FreeBSD_version                                    |
| `sys/conf/kern.opts.mk`             | Move `REPRODUCIBLE_BUILD` from `<em>DEFAULT_NO_OPTIONS</em>` *to* `DEFAULT_YES_OPTIONS`                                             |
| `etc/pkg/FreeBSD.conf`             | Replace `latest` with `quarterly` as the default package repository location |
| `release/pkg_repos/release-dvd.conf`             | Replace `latest` with `quarterly` as the default package repository location |
| `sys/conf/newvers.sh`             | 用 PRERELEASE 更新 BETA<em>X</em>                         |
| `sys/sys/param.h`             | 更新 __FreeBSD_version                                    |

然后，gitadm@FreeBSD.org 为 releng 分支添加新的批准者，如同为 stable 分支所做的那样。

```
% git add .
% git commit
```

现在有两个新的 __FreeBSD_version 值存在，还需要更新 Documentation Project 存储库中的~/documentation/content/en/books/porters-handbook/versions/chapter.adoc。

第一个 RC 构建完成并通过测试后，稳定/分支可以由 gitadm@FreeBSD.org 进行“解冻”。

在第一个 RC 可用后，应向 FreeBSD Bugmeister 团队发送电子邮件，以将新的 FreeBSD -RELEASE 添加到 bug 跟踪器中显示的下拉菜单中的 versions 中。

## 7. 构建 FreeBSD 安装介质

本节介绍生成 FreeBSD 开发快照和发行版的一般过程。

### 7.1. 发布构建脚本

在 FreeBSD 9.0-RELEASE 之前，src/release/Makefile 已更新以支持，并引入了 src/release/generate-release.sh 脚本作为一个包装器，以自动调用目标。

在 FreeBSD 9.2-RELEASE 之前，引入了 src/release/release.sh，它主要基于 src/release/generate-release.sh，支持指定配置文件来覆盖各种选项和环境变量。对配置文件的支持允许为每个架构构建一个发布，通过为每个调用指定单独的配置文件来实现。

使用 src/release/release.sh 构建单个发布的简要示例在/scratch 中：

```
# /bin/sh /usr/src/release/release.sh
```

使用 src/release/release.sh 构建单个交叉编译的发行版，使用不同的目标目录作为简短示例，创建一个定制的 release.conf 包含：

```
# release.sh configuration for powerpc/powerpc64
CHROOTDIR="/scratch-powerpc64"
TARGET="powerpc"
TARGET_ARCH="powerpc64"
KERNEL="GENERIC64"
```

然后，作为如下方式调用 src/release/release.sh：

```
# /bin/sh /usr/src/release/release.sh -c $HOME/release.conf
```

更多详细信息和示例用法，请参阅 src/release/release.conf.sample。

### 7.2. 构建 FreeBSD 发行版

在发布周期内，每个架构的 CHECKSUM.SHA512 和 CHECKSUM.SHA256 的副本存储在 FreeBSD Release Engineering Team 内部存储库中，除了包含在各种公告电子邮件中。每个包含诸如 base.txz、kernel.txz 等哈希值的 MANIFEST 也会被添加到Ports Collection 的 misc/freebsd-release-manifests 中。

为了准备发布构建，需要更新多个文件：

| 要编辑的文件 | 要更改的内容                                    |
| -------------- | ------------------------------------------------- |
| `sys/conf/newvers.sh`             | 将 BRANCH 的值更新为 RELEASE                    |
| `UPDATING`             | 添加预期的公告日期                              |
| `lib/csu/common/crtbrand.S`             | 用 sys/sys/param.h 中的值替换 __FreeBSD_version |

构建最终的 RELEASE 后，将 releng/13.0 分支标记为使用构建 RELEASE 的修订版 release/13.0.0，类似于创建 stable/13 和 releng/13.0 分支，这是使用 git tag 完成的。从仓库根目录：

|  | 确保你处于 releng/13.0 分支 |
| -- | ----------------------------- |

```
% git tag release/13.0.0
```

## 8. 将 FreeBSD 安装媒体发布到项目镜像

本节描述了将 FreeBSD 开发快照和发布版发布到项目镜像的过程。

### 8.1. 部署 FreeBSD 安装介质镜像

部署 FreeBSD 快照和发布版是一个两部分的过程：

* 创建目录结构以匹配 ftp-master 上的层次结构。如果在构建配置文件中（例如上述引用的构建脚本的 main.conf）定义了 EVERYTHINGISFINE ，则在构建完成后，${DESTDIR}/R/ftp-stage 中将自动创建这些目录结构，路径结构与 ftp-master 上预期的相匹配。这相当于直接在目录中运行以下命令：

  ```
  # make -C /usr/src/release -f Makefile.mirrors EVERYTHINGISFINE=1 ftp-stage
  ```

  架构建立后，thermite.sh 将从构建主机的 ${DESTDIR}/R/ftp-stage 同步到 /snap/ftp/snapshots 或者 /snap/ftp/releases。
* | 在将文件复制到 ftp-master 上的暂存目录之后，将文件移动到 pub/ 开始传播到项目镜像。一旦所有构建完成，/snap/ftp/snapshots 或者发布的 /snap/ftp/releases 将由 ftp-master 使用 rsync 轮询到 /archive/tmp/snapshots 或者 /archive/tmp/releases。 |  |
  |  | 在 FreeBSD 项目基础设施的 ftp-master 上，此步骤需要 root 级别的访问权限，因为必须以 archive 用户身份执行此步骤。 |
  | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -- |

### 8.2. 发布 FreeBSD 安装媒体

一旦镜像被暂存在 /archive/tmp/ 中，它们就可以通过将它们放入 /archive/pub/FreeBSD 中进行公开，为了减少传播时间，使用硬链接从 /archive/tmp 到 /archive/pub/FreeBSD 中。

|  | 为了使其有效，/archive/tmp 和 /archive/pub 必须位于同一逻辑文件系统上。 |
| -- | ------------------------------------------------------------------------- |

但是，有一个警告，必须在之后使用 rsync 来纠正 pub/FreeBSD/snapshots/ISO-IMAGES 中的符号链接，这将替换为硬链接，从而增加传播时间。

|  | 与暂存步骤一样，这需要 root 级访问，因为此步骤必须以 archive 用户身份执行。 |
| -- | ----------------------------------------------------------------------------- |

 作为 archive 用户：

```
% cd /archive/tmp/snapshots
% pax -r -w -l . /archive/pub/FreeBSD/snapshots
% /usr/local/bin/rsync -avH /archive/tmp/snapshots/* /archive/pub/FreeBSD/snapshots/
```

根据需要将快照替换为发布版。

## 9. 结束发布周期

本节描述了一般的发布后任务。

### 9.1. Post-Release Errata Notices

As the release cycle approaches conclusion, it is common to have several EN (Errata Notice) candidates to address issues that were discovered late in the cycle. Following the release, the FreeBSD Release Engineering Team and the FreeBSD Security Team revisit changes that were not approved prior to the final release, and depending on the scope of the change in question, may issue an EN.

|  | The actual process of issuing ENs is handled by the FreeBSD Security Team. |
| -- | ---------------------------------------------------------------------------- |

完成发布周期后，开发人员应填写勘误通知模板，特别是 Background 、 Problem Description 、 Impact 部分，如适用， Workaround 部分。

完成勘误通知模板后，应将其与针对 releng/ 分支的修补程序或稳定/ 分支的修订列表一起通过电子邮件发送。

发布后立即提出勘误通知请求时，请求应发送至 FreeBSD 发布工程团队和 FreeBSD 安全团队。如在《移交给 FreeBSD 安全团队》中所述，当 releng/ 分支移交给 FreeBSD 安全团队后，应将勘误通知请求发送至 FreeBSD 安全团队。

### 9.2. 移交给 FreeBSD 安全团队

版本发布后大约两周，发布工程师更新 Git 库，将 releng/13.0 分支的审批人从发布工程团队改为安全官。

## 10. 版本终止

此部分描述了在发布达到生命周期结束（End-of-Life）时需要更新的与网站相关的文件。

### 10.1. 终止生命周期的网站更新

当一个发布达到终止生命周期时，应该在网站上删除和/或更新对该发布的引用。

| 文件 | 如何更改                                                 |
| ------ | ---------------------------------------------------------- |
| `~/website/themes/beastie/layouts/index.html`     | 删除 u-relXXX-announce 和 u-relXXX-announce 的引用。     |
| `~/website/content/en/releases/_index.adoc`     | 将 u-relXXX-* 变量从支持的发布列表移至“遗留版本”列表。 |
| `~/website/content/en/releng/_index.adoc`     | 更新适当的 releng 分支以反映该分支不再受支持。           |
| `~/website/content/en/security/_index.adoc`     | 从受支持的分支列表中删除该分支。                         |
| `~/website/content/en/where.adoc`     | 删除发布的 URL。                                         |
| `~/website/themes/beastie/layouts/partials/sidenav.html`     | 删除 u-relXXX-announce 和 u-relXXX-announce 引用。       |
| `~/website/static/security/advisory-template.txt`     | 删除对发布和 releng 分支的引用。                         |
| `~/website/static/security/errata-template.txt`     | 删除对发布和 releng 分支的引用。                         |
