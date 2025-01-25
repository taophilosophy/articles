# BSD 简介

<details open="" data-immersive-translate-walked="1743ab44-cb8d-46ea-b879-6014ad6e4594"><summary data-immersive-translate-walked="1743ab44-cb8d-46ea-b879-6014ad6e4594" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

AMD、AMD Athlon、AMD Opteron、AMD Phenom、AMD Sempron、AMD Turion、Athlon、Élan、Opteron 和 PCnet 是 Advanced Micro Devices, Inc. 的商标。

苹果、AirPort、FireWire、iMac、iPhone、iPad、Mac、Macintosh、Mac OS、Quicktime 和 TrueType 是 Apple Inc. 在美国和其他国家注册的商标。

Git 和 Git 徽标是 Software Freedom Conservancy, Inc.（Git 项目的公司总部）在美国和/或其他国家的注册商标或商标。

Linux 是 Linus Torvalds 的注册商标。

Motif, OSF/1, and UNIX are registered trademarks and IT DialTone and The Open Group are trademarks of The Open Group in the United States and other countries.

Sun, Sun Microsystems, Java, Java Virtual Machine, JDK, JRE, JSP, JVM, Netra, OpenJDK, Solaris, StarOffice, SunOS and VirtualBox are trademarks or registered trademarks of Sun Microsystems, Inc. in the United States and other countries.

UNIX is a registered trademark of The Open Group in the United States and other countries.

制造商和销售商用来区分其产品的大部分名称都被视为商标。在本文件中出现这些名称时，如果 FreeBSD 项目意识到其商标声明，这些名称后面将跟随“™”或“®”符号。

</details>

 摘要

在开源世界中，“Linux”几乎等同于“操作系统”，但并非唯一的开源 UNIX®操作系统。

那么秘密是什么呢？为什么 BSD 不太出名？本文将解决这些及其他问题。

在本文中，将像这样指出 BSD 和 Linux 之间的差异。

---

## 1. 什么是 BSD？

BSD 代表“伯克利软件发行版”。这是来自加州大学伯克利分校的源代码发行版名称，最初是 AT&T 研究 UNIX®操作系统的扩展。几个开源操作系统项目基于称为 4.4BSD-Lite 的此源代码发布。此外，它们包括来自其他开源项目（尤其是 GNU 项目）的多个软件包。整个操作系统包括：

* BSD 内核处理进程调度、内存管理、对称多处理（SMP）、设备驱动程序等功能。
* C 库是系统的基本 API。BSD C 库基于伯克利的代码，而不是 GNU 项目的代码。
* 诸如shells、文件实用工具、编译器和链接器之类的实用工具。其中一些实用工具源自 GNU 项目，其他则不是。
* 处理图形显示的 X 窗口系统。大多数版本的 BSD 中使用的 X 窗口系统由 X.Org 项目维护。FreeBSD 允许用户选择各种桌面环境，例如 Gnome、KDE 或 Xfce；以及像 Openbox、Fluxbox 或 Awesome 这样的轻量级窗口管理器。
* 许多其他程序和实用工具。

## 2. 真的 UNIX®？

BSD 操作系统并非克隆品，而是 AT&T 研究 UNIX® 操作系统的开源衍生版本，也是现代 UNIX® System V 的祖先。这可能会让你感到惊讶。为什么会这样，当 AT&T 从未将其代码作为开源发布？

的确，AT&T UNIX® 并非开源，从版权角度来看，BSD 明确地不是 UNIX®，但另一方面，AT&T 导入了来自其他项目的源代码，尤其是加州大学伯克利分校的计算机科学研究小组（CSRG）。从 1976 年开始，CSRG 开始发布他们的软件磁带，称之为伯克利软件发布或 BSD。

最初的 BSD 发行版主要由用户程序组成，但这种情况在 CSRG 与国防高级研究计划局（DARPA）签订合同以升级其网络 ARPANET 上的通信协议后发生了显著变化。新协议被称为互联网协议，后来由于最重要的协议而被称为 TCP/IP。第一种广泛分发的实现是 1982 年的 4.2BSD 的一部分。

在 1980 年代，一些新的工作站公司涌现出来。许多人更愿意许可 UNIX®而不是为自己开发操作系统。特别是，Sun Microsystems 许可了 UNIX®并实现了一个 4.2BSD 版本，称为 SunOS™。当 AT&T 自己被允许商业销售 UNIX®时，他们从一个有些简陋的实现 System III 开始，随后迅速推出 System V。System V 代码库不包括网络功能，因此所有实现都包括来自 BSD 的额外软件，包括 TCP/IP 软件，还有诸如 csh 和 vi 编辑器等工具。这些增强统称为 Berkeley 扩展。

BSD 磁带包含 AT&T 的源代码，因此需要 UNIX®源代码许可证。到 1990 年，CSRG 的资金即将耗尽，面临关闭。该小组的一些成员决定发布 BSD 代码，这是开源的，不含 AT&T 的专有代码。这最终通过 Networking Tape 2（通常称为 Net/2）实现。Net/2 不是一个完整的操作系统：大约 20%的内核代码缺失。CSRG 的一名成员 William F. Jolitz 编写了剩余的代码，并于 1992 年初发布为 386BSD。与此同时，另一组前 CSRG 成员成立了一家名为 Berkeley Software Design Inc.的商业公司，并基于相同的源代码发布了一个操作系统的测试版，称为 BSD/386。该操作系统的名称后来更改为 BSD/OS。

386BSD 从未成为一个稳定的操作系统。相反，它在 1993 年分裂成了两个项目：NetBSD 和 FreeBSD。这两个项目最初的分歧是由于对 386BSD 改进的耐心不同：NetBSD 的人们在年初就开始了工作，而 FreeBSD 的第一个版本直到年底还没有准备好。与此同时，代码库已经有了足够的分歧，使合并变得困难。此外，这些项目有不同的目标，正如我们将在下面看到的。1996 年，OpenBSD 从 NetBSD 中分裂出来，2003 年，DragonFlyBSD 从 FreeBSD 中分裂出来。

## 3. 为什么 BSD 不太为人所知？

因为一些原因，BSD 相对较不为人所知：

1. BSD 开发人员通常更感兴趣于优化他们的代码，而不是市场营销。
2. Linux 的许多流行之处是由于 Linux 项目外部的因素，如新闻界，以及成立提供 Linux 服务的公司。直到最近，开源的 BSD 并没有这样的支持者。
3. 1992 年，AT&T 起诉了 BSDI，即 BSD/386 的供应商，指控该产品包含了 AT&T 拥有版权的代码。该案件于 1994 年庭外和解，但诉讼的阴影依然困扰着人们。2000 年 3 月，网络上发表的一篇文章声称这场诉讼案件已经“最近解决”。这场诉讼案件澄清的其中一个细节是命名问题：在 1980 年代，BSD 被称为“BSD UNIX®”。随着从 BSD 中清除了最后一丝 AT&T 代码，它也失去了使用 UNIX® 名称的权利。因此，你会在书名中看到对“4.3BSD UNIX® 操作系统”和“4.4BSD 操作系统”的提及。

## 4. 比较 BSD 和 Linux

那么，说 Debian Linux 和 FreeBSD 之间真正的区别是什么呢？对于普通用户，差异令人惊讶地很小：两者都是类 UNIX®操作系统。两者都是由非商业项目开发（当然，这并不适用于许多其他 Linux 发行版）。在接下来的章节中，我们将研究 BSD 并将其与 Linux 进行比较。该描述最符合 FreeBSD，据估计占有 BSD 安装量的 80％，但与 NetBSD、OpenBSD 和 DragonFlyBSD 的差异很小。

### 4.1. 谁拥有 BSD？

没有一个人或公司拥有 BSD。它是由一个遍布全球的高度技术和忠诚的社区的贡献者创造和分发的。BSD 的一些组件是自己权威的开源项目，并由不同的项目维护者管理。

### 4.2. BSD 是如何开发和更新的？

BSD 内核是按照开源开发模型开发和更新的。每个项目维护一个公开访问的源代码树，其中包含项目的所有源文件，包括文档和其他附属文件。用户可以获取任何版本的完整副本。

全球许多开发人员为 BSD 的改进作出贡献。他们分为三种类型：

* 贡献者编写代码或文档。他们不被允许直接向源代码树提交（添加代码）。要将他们的代码包含到系统中，必须由注册开发人员（称为提交者）审核和检入。
* 提交者是具有对源代码树写访问权限的开发人员。要成为提交者，个人必须展示在其活跃领域的能力。提交者可以自行决定在向源代码树提交更改之前是否应获得授权。通常，经验丰富的提交者可以在不获得一致同意的情况下进行显然正确的更改。例如，文档项目提交者可以在不经审核的情况下纠正排版或语法错误。另一方面，进行深远或复杂更改的开发人员通常需要在提交之前提交其更改供审核。在极端情况下，例如，具有首席架构师等职能的核心团队成员可能会要求从源代码树中删除更改，这个过程称为回滚。所有提交者都会收到描述每个个体提交的邮件，因此不可能进行秘密提交。
* 核心团队。FreeBSD 和 NetBSD 各有一个管理项目的核心团队。核心团队是在项目进展过程中发展起来的，他们的角色并不总是明确的。成为核心团队成员不一定是开发人员，但通常是这样。核心团队的规则因项目而异，但总体而言，他们比非核心团队成员对项目方向有更大的发言权。

这种安排在许多方面与 Linux 不同：

1. 没有一个人控制系统的内容。实际上，这种差异被高估了，因为首席架构师可以要求撤回代码，即使在 Linux 项目中，也有几个人被允许进行更改。
2. 另一方面，有一个中央存储库，这是一个可以找到整个操作系统源代码的地方，包括所有旧版本。
3. BSD 项目维护整个“操作系统”，而不仅仅是内核。这种区别只是在边缘上有用：没有应用程序，无论是 BSD 还是 Linux 都没有用。在 BSD 下使用的应用程序通常与在 Linux 下使用的应用程序相同。
4. 由于单一的 Git 源代码树的正式维护，BSD 的开发是清晰的，可以通过发布号或日期访问系统的任何版本。Git 还允许系统的增量更新：例如，FreeBSD 仓库每天更新约 100 次。其中大部分变更都很小。

### 4.3. BSD 发行版

FreeBSD、NetBSD 和 OpenBSD 提供了三种不同的“发行版”。与 Linux 一样，发行版被分配一个数字，例如 1.4.1 或 3.5。此外，版本号带有表示用途的后缀：

1. 系统的开发版本称为 CURRENT。FreeBSD 将一个数字分配给 CURRENT，例如 FreeBSD 5.0-CURRENT。NetBSD 使用略有不同的命名方案，并附加一个表示内部接口变化的单字母后缀，例如 NetBSD 1.4.3G。OpenBSD 没有指定一个数字（“OpenBSD-current”）。系统上的所有新开发都将投入到这个分支中。
2. 定期，每年两到四次，项目会发布系统的 RELEASE 版本，可以通过 CD-ROM 或 FTP 网站免费下载，例如 OpenBSD 2.6-RELEASE 或 NetBSD 1.4-RELEASE。RELEASE 版本面向终端用户，是系统的常规版本。NetBSD 还提供补丁版本，第三位数字表示，例如 NetBSD 1.4.2。
3. 在 RELEASE 版本中发现 bug 后，会被修复，并将修复添加到 Git 树中。在 FreeBSD 中，生成的版本称为 STABLE 版本，而在 NetBSD 和 OpenBSD 中则继续称为 RELEASE 版本。较小的新功能也可以在 CURRENT 分支测试一段时间后添加到此分支。安全和其他重要的 bug 修复也适用于所有受支持的 RELEASE 版本。

*相比之下，Linux 维护两个独立的代码树：稳定版本和开发版本。稳定版本有一个偶数次版本号，例如 2.0、2.2 或 2.4。开发版本有一个奇数次版本号，例如 2.1、2.3 或 2.5。在每种情况下，数字后面都有一个进一步的数字表示确切的发布版本。此外，每个供应商都添加了自己的用户程序和工具，因此发行版的名称也很重要。每个发行版供应商也为发行版分配版本号，因此完整的描述可能是“TurboLinux 6.0，内核版本 2.2.14”。*

### 4.4. 有哪些版本的 BSD 可用？

与众多 Linux 发行版不同，只有四个主要的开源 BSD。每个 BSD 项目都维护自己的源代码树和内核。实际上，尽管如此，项目之间的用户空间代码的分歧似乎比在 Linux 中少。

很难对每个项目的目标进行分类：差异非常主观。基本上，

* FreeBSD 旨在通过端用户的高性能和易用性，并且是网络内容提供者的首选。它可以运行在多种平台上，并且拥有比其他项目更多的用户。
* NetBSD 旨在实现最大的可移植性：“当然可以运行 NetBSD”。它可以运行在从掌上电脑到大型服务器的各种设备上，并且甚至被用于 NASA 的太空任务。它特别适合在老旧的非 Intel®硬件上运行。
* OpenBSD 旨在实现安全性和代码纯净性：它采用开源概念和严格的代码审查相结合的方式来创建一个可以证明是正确的系统，这使得它成为像银行、证券交易所和美国政府部门这样注重安全性的组织的选择。与 NetBSD 类似，它可以运行在多种平台上。
* DragonFlyBSD 旨在在从单节点 UP 系统到大规模集群系统的各种环境下实现高性能和可扩展性。 DragonFlyBSD 有几个长期技术目标，但重点在于提供一个易于理解，维护和开发的 SMP 能力基础设施。

还有另外两个不是开源的 BSD UNIX®操作系统，即 BSD/OS 和苹果的 Mac OS® X：

* BSD/OS 是 4.4BSD 派生系统中最古老的。它不是开源的，尽管源代码许可证的成本相对较低。它在许多方面类似于 FreeBSD。在 BSDi 被 Wind River Systems 收购两年后，BSD/OS 未能作为独立产品存活。Wind River 可能仍然提供支持和源代码，但所有新的开发都集中在 VxWorks 嵌入式操作系统上。
* Mac OS® X 是 Apple® 的 Mac® 系列最新的操作系统版本。这个操作系统的 BSD 核心 Darwin 可作为功能齐全的开源操作系统用于 x86 和 PPC 计算机。然而，Aqua/Quartz 图形系统和 Mac OS® X 的许多其他专有方面仍然是闭源的。几个 Darwin 开发人员也是 FreeBSD 提交者，反之亦然。

### 4.5. BSD 许可证与 GNU 公共许可证有何不同？

Linux 是在 GNU 通用公共许可证 (GPL) 下发布的，该许可证旨在消除闭源软件。特别是，任何在 GPL 下发布的产品的衍生作品，如果要求，必须提供源代码。相比之下，BSD 许可证的限制较少：允许仅二进制分发。这对于嵌入式应用特别有吸引力。

### 4.6. 还有什么我需要知道吗？

由于 BSD 上可用的应用程序比 Linux 少，BSD 开发者创建了一个 Linux 兼容包，允许 Linux 程序在 BSD 下运行。该包包括内核修改，以正确执行 Linux 系统调用，并提供 Linux 兼容文件，如 C 库。在相同速度的 Linux 机器和 BSD 机器上运行的 Linux 应用程序之间没有明显的执行速度差异。

BSD 的“一供应商提供所有”特性意味着升级比 Linux 更容易处理。BSD 通过为早期库版本提供兼容模块来处理库版本升级，因此可以无问题地运行几年前的二进制文件。

### 4.7. 我应该使用 BSD 还是 Linux？

在实践中，所有这些是什么意思？谁应该使用 BSD，谁应该使用 Linux？

这是一个非常难回答的问题。以下是一些指导原则：

* "如果它没坏，就不要修理它": 如果你已经在使用一个开源操作系统，并且对它感到满意，那么可能没有什么好的理由去更改。
* BSD 系统，特别是 FreeBSD，在性能上可能比 Linux 表现更好。但这并非普遍情况。在许多情况下，性能几乎没有差异。在一些情况下，Linux 可能比 FreeBSD 表现更好。
* 一般来说，BSD 系统因为代码基础更加成熟，在可靠性方面拥有更好的声誉。
* BSD 项目因其文档的质量和完整性享有良好声誉。各种文档项目旨在提供活跃更新的文档，支持多种语言，并覆盖系统的所有方面。
* BSD 许可证可能比 GPL 更具吸引力。
* BSD 可以执行大多数 Linux 二进制文件，而 Linux 不能执行 BSD 二进制文件。许多 BSD 实现还可以执行其他类 UNIX 系统的二进制文件。因此，BSD 可能比 Linux 提供了更容易的迁移路径。

### 4.8. 谁为 BSD 提供支持、服务和培训?

iXsystems, Inc.为 FreeBSD 提供支持合同。

此外，每个项目都有一份可供雇佣的顾问名单: FreeBSD、NetBSD 和 OpenBSD。
