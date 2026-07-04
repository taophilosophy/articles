# 使用 FreeBSD 开发产品

## 摘要

FreeBSD 项目是全球性、基于志愿者的合作项目，旨在开发可移植且高质量的操作系统。FreeBSD 项目以宽松的许可协议发布其产品的源代码，目的是鼓励使用其代码。与 FreeBSD 项目合作可以帮助组织减少产品上市时间、降低工程成本，并提高产品质量。

本文探讨了在专用设备和软件产品中使用 FreeBSD 代码时可能遇到的问题。本文突出了使 FreeBSD 成为产品开发优秀基础的特性。文章最后建议了一些与 FreeBSD 项目合作时的“最佳实践”。


## 1. 简介

FreeBSD 如今作为高性能服务器操作系统而广为人知。它部署在全球数百万台 Web 服务器和面向互联网的主机上。FreeBSD 的代码也构成了许多产品的核心部分，从网络路由器、防火墙和存储设备等专用设备，到个人计算机。FreeBSD 的部分内容也用于商业包装软件中（请参阅[FreeBSD 作为构建模块集](https://docs.freebsd.org/en/articles/building-products/#freebsd-intro)）。

在本文中，我们将从软件工程资源的角度探讨 [FreeBSD 项目](https://www.freebsd.org/)，即作为一组构建模块和过程，你可以利用它们来构建产品。

尽管 FreeBSD 的源代码免费向公众发布，但为了充分获得该项目成果的益处，组织需要与该项目进行*合作*。在本文接下来的章节中，我们将讨论与 FreeBSD 项目有效合作的方式，以及在此过程中需要避免的陷阱。

**读者警告。** 作者认为，本文中列出的 FreeBSD 项目的特点在文章构思和撰写时（2005 年）是大致真实的。然而，读者应当意识到，开源社区使用的实践和过程会随着时间的推移发生变化，因此本文中的信息应被视为指示性而非规范性。

### 1.1. 目标读者

本文适合以下广泛群体：

* 产品公司的决策者，他们希望提高产品质量，减少上市时间，并从长远来看降低工程成本。
* 寻找利用“开源”最佳实践的技术顾问。
* 对理解开源项目动态感兴趣的行业观察者。
* 希望使用 FreeBSD 并寻找贡献方式的软件开发者。

### 1.2. 文章目标

阅读完本文后，你应当能够：

* 理解 FreeBSD 项目的目标及其组织结构。
* 理解其开发模式和发布工程过程。
* 理解传统企业软件开发过程与 FreeBSD 项目中使用的过程有何不同。
* 了解项目使用的通信渠道，以及你可以期待的透明度程度。
* 了解与项目合作的最佳方式——如何有效地减少工程成本、缩短上市时间、管理安全漏洞，并在 FreeBSD 项目发展过程中保持与你产品的未来兼容性。

### 1.3. 文章结构

本文的其余部分结构如下：

* [FreeBSD 作为构建模块集](https://docs.freebsd.org/en/articles/building-products/#freebsd-intro) 介绍了 FreeBSD 项目，探讨了其组织结构、关键技术和发布工程过程。
* [与 FreeBSD 合作](https://docs.freebsd.org/en/articles/building-products/#freebsd-collaboration) 介绍了与 FreeBSD 项目合作的方式，并审视了企业在与 FreeBSD 等志愿项目合作时常遇到的陷阱。
* [结论](https://docs.freebsd.org/en/articles/building-products/#conclusion) 进行总结。

## 2. FreeBSD 作为构建模块集

FreeBSD 是构建产品的优秀基础：

* FreeBSD 源代码以宽松的 BSD 许可协议发布，方便其在商业产品中的采纳，[为何使用 BSD 样式许可证发布开源项目](https://docs.freebsd.org/en/articles/building-products/#Mon2005) 几乎没有麻烦。
* FreeBSD 项目有着卓越的工程实践，企业可以加以利用。
* 该项目在运作上极其透明，使得使用其代码的组织可以有效地规划未来。
* 从加利福尼亚大学伯克利分校的计算机科学研究组[二十年的伯克利 Unix：从 AT&T 拥有到自由再分发](https://docs.freebsd.org/en/articles/building-products/#McKu1999-1)所带来的文化，促进了高质量的工作。FreeBSD 中的一些特性定义了技术的前沿。

[创新发生在其他地方：开源作为商业战略](https://docs.freebsd.org/en/articles/building-products/#GoldGab2005) 更详细地探讨了使用开源的商业原因。对于企业来说，将 FreeBSD 组件用于产品中带来的好处包括缩短上市时间、降低开发成本和降低开发风险。

### 2.1. 使用 FreeBSD 构建

以下是一些组织使用 FreeBSD 的方式：

* 作为经过测试的库和工具代码的上游来源。
  作为该项目的“下游”，企业可以利用上游代码所获得的新特性、错误修复和测试。
* 作为嵌入式操作系统（例如，用于 OEM 路由器和防火墙设备）。在这种模型下，企业使用定制的 FreeBSD 内核和应用程序集，并为设备添加专有的管理层。OEM 可以从 FreeBSD 项目上游添加的新硬件支持中受益，并从基础系统所接受的测试中获益。
  FreeBSD 提供了一个自托管的开发环境，允许轻松创建这样的配置。
* 作为高端存储和网络设备管理功能的类 Unix 环境，运行在独立的处理器“刀片”上。
  FreeBSD 提供了创建专用操作系统和应用程序镜像的工具。它对 BSD Unix API 的实现成熟且经过测试。FreeBSD 还可以为高端设备的其他组件提供稳定的交叉开发环境。
* 作为通过来自全球开发者团队的广泛测试和支持，获取非关键“知识产权”的途径。
  在这种模型下，组织将有用的基础设施框架贡献给 FreeBSD 项目（例如，参见 [netgraph(3)](https://man.freebsd.org/cgi/man.cgi?query=netgraph&sektion=3&format=html)）。代码所获得的广泛曝光有助于快速识别性能问题和缺陷。顶尖开发者的参与也有助于对基础设施进行有益的扩展，贡献组织也能从中受益。
* 作为支持嵌入式操作系统如 [RTEMS](http://www.rtems.com/) 和 [eCOS](http://ecos.sourceware.org/) 的交叉开发环境。
  在 FreeBSD 所移植和打包的 36000 个应用程序中，有许多完备的开发环境。
* 作为在其他专有操作系统中支持类 Unix API 的方式，提高其对应用开发者的吸引力。
  在这种情况下，FreeBSD 的部分内核和应用程序被“移植”到与其他任务一起运行的专有操作系统中。稳定且经过良好测试的类 Unix API 实现的可用性可以减少将流行应用程序移植到专有操作系统所需的工作量。由于 FreeBSD 附带了高质量的内部文档，并且具有有效的漏洞管理和发布工程过程，保持系统更新的成本较低。

### 2.2. 技术

FreeBSD 项目支持大量的技术，以下是其中的一部分：

* 完整的系统，可以为[多种架构](https://www.freebsd.org/platforms/)进行交叉托管。
* 模块化的对称多处理能力内核，具有可加载的内核模块和灵活易用的配置系统。
* 支持以接近机器速度模拟 Linux™ 和 SVR4 二进制文件。支持二进制 Windows™ (NDIS) 网络驱动程序。
* 提供多种编程任务的库：归档工具、FTP 和 HTTP 支持、线程支持，以及完整的 POSIX™ 类编程环境。
* 安全特性：强制访问控制（[mac(9)](https://man.freebsd.org/cgi/man.cgi?query=mac&sektion=9&format=html)）、Jails（[jail(2)](https://man.freebsd.org/cgi/man.cgi?query=jail&sektion=2&format=html)）、ACLs 和内核加密设备支持。
* 网络特性：防火墙、QoS 管理、高性能 TCP/IP 网络，支持许多扩展。
  FreeBSD 内核中的 Netgraph（[netgraph(4)](https://man.freebsd.org/cgi/man.cgi?query=netgraph&sektion=4&format=html)）框架允许内核网络模块以灵活的方式连接。
* 存储技术支持：光纤通道、SCSI、软件和硬件 RAID、ATA 和 SATA。
  FreeBSD 支持多种文件系统，其原生的 UFS2 文件系统支持软更新、快照以及超大的文件系统容量（每个文件系统可达 16TB）[软更新：一种消除大多数同步写入的技术](https://docs.freebsd.org/en/articles/building-products/#McKu1999)。
  FreeBSD 内核中的 GEOM（[geom(4)](https://man.freebsd.org/cgi/man.cgi?query=geom&sektion=4&format=html)）框架允许内核存储模块以灵活的方式组合。
* 超过 36000 个移植的应用程序，包括商业和开源软件，通过 FreeBSD 的 Ports 进行管理。

### 2.3. 组织结构

FreeBSD 的组织结构是非等级化的。

FreeBSD 的贡献者主要分为两类：普通用户和具有写权限的开发者（行话中称为 *committers*）。

第一类贡献者有成千上万；绝大多数对 FreeBSD 的贡献来自于这个群体。对代码库的提交权限（写权限）授予那些对项目有持续贡献的个人。获得提交权限的开发者承担额外责任，并为新提交者分配导师，以帮助他们熟悉相关流程。

![freebsd organization](https://docs.freebsd.org/images/articles/building-products/freebsd-organization.png)

**图 1. FreeBSD 组织结构**

冲突解决由九名成员组成的“核心团队”来执行，该团队由提交者选举产生。

FreeBSD 没有“企业”提交者。个人提交者需要对自己引入的代码变更负责。[FreeBSD 提交者指南](https://docs.freebsd.org/en/articles/committers-guide/) [提交者指南](https://docs.freebsd.org/en/articles/building-products/#ComGuide) 详细记录了提交者的规则和责任。

FreeBSD 的项目模型在[FreeBSD 项目的项目模型](https://docs.freebsd.org/en/articles/building-products/#Nik2005)中有详细的讨论。

### 2.4. FreeBSD 发布工程流程

FreeBSD 的发布工程流程在确保发布版本质量高方面起着重要作用。在任何时间点，FreeBSD 的志愿者都会支持多个代码分支（[FreeBSD 发布分支](https://docs.freebsd.org/en/articles/building-products/#fig-freebsd-branches)）：

* 新特性和破坏性代码进入开发分支，也称为 *-CURRENT* 分支。
* *-STABLE* 分支是从 HEAD 定期分支出来的代码线。只有经过测试的代码才能进入 *-STABLE* 分支。新特性在 *-CURRENT* 分支中经过测试和稳定后，才允许进入 *-STABLE* 分支。
* *-RELEASE* 分支由 FreeBSD 安全团队维护。只有对关键问题的 bug 修复才能进入 *-RELEASE* 分支。

![FreeBSD 分支](https://docs.freebsd.org/images/articles/building-products/freebsd-branches.png)

**图 2. FreeBSD 发布分支**

只要用户和开发者对其仍有兴趣，代码分支就会保持活跃。

机器架构被分为“层级”；*Tier 1* 架构由项目的发布工程和安全团队全面支持，*Tier 2* 架构以尽力而为的方式支持，而实验性架构则属于 *Tier 3*。支持的[架构列表](https://docs.freebsd.org/en/articles/committers-guide/#archs)是 FreeBSD 文档的一部分。

发布工程团队在项目网站上发布了[未来发布的路线图](https://www.freebsd.org/releng/)。路线图中列出的日期并非截止日期；FreeBSD 会在代码和文档准备好时发布。

FreeBSD 的发布工程流程可参见[FreeBSD 发布工程](https://docs.freebsd.org/en/articles/building-products/#RelEngDoc)。

## 3. 与 FreeBSD 协作

像 FreeBSD 这样的开源项目提供了非常高质量的成品代码。

尽管访问优质源代码可以降低初始开发成本，但随着时间的推移，管理变更的成本将开始占主导地位。随着计算环境的变化和新安全漏洞的发现，你的产品也需要改变和适应。使用开源代码最好视为一个 *持续过程* 而非一次性活动。最适合合作的项目是那些 *活跃的* 项目；即，拥有活跃社区、明确目标和透明工作风格的项目。

* FreeBSD 拥有活跃的开发者社区。截至撰写本文时，来自全球每个有人居住的大陆的数千名贡献者参与其中，且有 300 余名具有提交权限的个人。
* FreeBSD 项目的目标是[贡献给 FreeBSD 项目](https://docs.freebsd.org/en/articles/building-products/#Hub1994)：
  * 为流行的计算机硬件开发高质量的操作系统；
  * 并且以宽松的许可证将我们的工作提供给所有人。
* FreeBSD 享有开放且透明的工作文化。项目中的几乎所有讨论都通过电子邮件在[公开邮件列表](https://lists.freebsd.org/)上进行，这些邮件列表也会被存档以供后人查阅。项目的政策有[文档记录](https://www.freebsd.org/internal/policies/)，并受版本控制管理。项目的参与对所有人开放。

### 3.1. 理解 FreeBSD 文化

为了有效地与 FreeBSD 项目合作，你需要理解该项目的文化。

志愿者驱动的项目与营利性公司运作的规则不同。公司进入开源世界时常犯的错误是低估了这些差异。

**动机。** 大多数对 FreeBSD 的贡献都是自愿进行的，不涉及金钱奖励。驱动个人参与的动机因素是复杂的，从利他主义到解决 FreeBSD 所尝试解决的问题的兴趣不等。在这种环境中，“优雅从来不是可有可无的”[Good Lisp 编程风格教程](https://docs.freebsd.org/en/articles/building-products/#Nor1993)。

**长期视角。** FreeBSD 的根源可以追溯到近二十年前，加利福尼亚大学伯克利分校计算机科学研究组（CSRG）的工作。[^freebsd_history] 一些最初的 CSRG 开发者仍与该项目保持联系。

[^freebsd_history]: FreeBSD 的源代码库包含了自项目创立以来的所有历史记录，并且还有可用的 CD-ROM，里面包含了 CSRG（计算机科学研究组）早期的代码。

该项目重视长期视角 [十年学会编程](https://docs.freebsd.org/en/articles/building-products/#Nor2001)。在项目中，常见的缩写是 DTRT，代表“做正确的事”（Do The Right Thing）。

**开发流程。** 计算机程序是沟通的工具：在一个层面上，程序员通过精确的符号与工具（编译器）沟通，工具将他们的指令翻译成可执行代码。在另一个层面上，相同的符号用于程序员之间意图的沟通。

在项目中，很少使用正式的规格和设计文档。相反，使用清晰且编写良好的代码和变更日志（[变更日志样本](https://docs.freebsd.org/en/articles/building-products/#fig-change-log)）。FreeBSD 的开发通过“粗略共识和运行中的代码”进行[互联网的架构原则](https://docs.freebsd.org/en/articles/building-products/#Carp1996)。

```sh
r151864 | bde | 2005-10-29 09:34:50 -0700 (Sat, 29 Oct 2005) | 13 lines
Changed paths:
   M /head/lib/msun/src/e_rem_pio2f.c

Use double precision to simplify and optimize arg reduction for small
and medium size args too: instead of conditionally subtracting a float
17+24, 17+17+24 or 17+17+17+24 bit approximation to pi/2, always
subtract a double 33+53 bit one.  The float version is now closer to
the double version than to old versions of itself -- it uses the same
33+53 bit approximation as the simplest cases in the double version,
and where the float version had to switch to the slow general case at
|x| == 2^7*pi/2, it now switches at |x| == 2^19*pi/2 the same as the
double version.

This speeds up arg reduction by a factor of 2 for |x| between 3*pi/4 and
2^7*pi/4, and by a factor of 7 for |x| between 2^7*pi/4 and 2^19*pi/4.
```

变更日志样本

程序员之间的沟通通过使用通用编码标准 [style(9)](https://man.freebsd.org/cgi/man.cgi?query=style&sektion=9&format=html) 得到增强。

**沟通渠道。** FreeBSD 的贡献者遍布全球。电子邮件（以及较少使用的 IRC）是项目中首选的沟通方式。

### 3.2. 与 FreeBSD 项目合作的最佳实践

接下来，我们来看一些在产品开发中最有效利用 FreeBSD 的最佳实践。

**规划长期发展。**
设置有助于跟踪 FreeBSD 开发进展的流程。例如：

**跟踪 FreeBSD 源代码。** 该项目通过使用 [svnsync](https://docs.freebsd.org/en/articles/committers-guide/#svn-advanced-use-setting-up-svnsync) 使镜像其 SVN 仓库变得简单。拥有完整的源代码历史对于调试复杂问题非常有用，并且可以深入了解原开发者的意图。使用强大的源代码管理系统，可以轻松地合并上游 FreeBSD 代码库和你自己内部代码之间的变化。

[通过 `svn blame` 生成的注释源代码清单](https://docs.freebsd.org/en/articles/building-products/#fig-svn-blame) 显示了变更日志样本中引用的文件的注释清单部分。每一行代码的来源历史清晰可见。显示 FreeBSD 每个文件历史的注释清单 [可以在网上查看](https://svnweb.freebsd.org/)。

```sh
#REV         #WHO #DATE                                        #TEXT

176410        bde 2008-02-19 07:42:46 -0800 (Tue, 19 Feb 2008) #include <sys/cdefs.h>
176410        bde 2008-02-19 07:42:46 -0800 (Tue, 19 Feb 2008) __FBSDID("$FreeBSD$");
  2116        jkh 1994-08-19 02:40:01 -0700 (Fri, 19 Aug 1994)
  2116        jkh 1994-08-19 02:40:01 -0700 (Fri, 19 Aug 1994) /* __ieee754_rem_pio2f(x,y)
  8870    rgrimes 1995-05-29 22:51:47 -0700 (Mon, 29 May 1995)  *
176552        bde 2008-02-25 05:33:20 -0800 (Mon, 25 Feb 2008)  * return the remainder of x rem pi/2 in *y
176552        bde 2008-02-25 05:33:20 -0800 (Mon, 25 Feb 2008)  * use double precision for everything except passing x
152535        bde 2005-11-16 18:20:04 -0800 (Wed, 16 Nov 2005)  * use __kernel_rem_pio2() for large x
  2116        jkh 1994-08-19 02:40:01 -0700 (Fri, 19 Aug 1994)  */
  2116        jkh 1994-08-19 02:40:01 -0700 (Fri, 19 Aug 1994)
176465        bde 2008-02-22 07:55:14 -0800 (Fri, 22 Feb 2008) #include <float.h>
176465        bde 2008-02-22 07:55:14 -0800 (Fri, 22 Feb 2008)
  2116        jkh 1994-08-19 02:40:01 -0700 (Fri, 19 Aug 1994) #include "math.h"
```

通过 `svn blame` 生成的注释源代码清单

**使用 gatekeeper（门控者）。** 任命一名 *gatekeeper* 来监控 FreeBSD 开发，留意可能影响你产品的变化。

**向上游报告 bug。** 如果你注意到正在使用的 FreeBSD 代码中有 bug，请提交 [bug 报告](https://www.freebsd.org/support/bugreports/)。这一步可以确保下次你从上游获取代码时不必再次修复该 bug。

**利用 FreeBSD 的发布工程成果。** 使用来自 -STABLE 开发分支的代码。这些开发分支由 FreeBSD 的发布工程和安全团队正式支持，并且包含经过测试的代码。

**捐赠代码以降低成本。** 开发产品成本的主要部分是维护工作。通过向项目捐赠非关键代码，你可以使你的代码得到比通常更多的曝光，从而有助于发现更多的 bug、安全漏洞以及性能问题，并得到修复。

**有效获取支持。** 对于有紧迫期限的产品，建议你聘请或与有 FreeBSD 经验的开发者或公司达成咨询协议。你可以通过 [FreeBSD 相关的就业邮件列表](https://lists.freebsd.org/subscription/freebsd-jobs) 寻找人才。FreeBSD 项目还维护了 [咨询顾问和咨询公司名录](https://www.freebsd.org/commercial/consult_bycat/)，他们从事 FreeBSD 相关工作。[BSD 认证小组](http://www.bsdcertification.org/) 提供所有主要 BSD 派生操作系统的认证。

对于不太紧急的需求，你可以在 [项目邮件列表](https://lists.freebsd.org/) 上寻求帮助。寻求帮助时可以参考 [如何聪明地提问](https://docs.freebsd.org/en/articles/building-products/#Ray2004) 这篇指南。

**公开宣传你的参与。** 你不必公开宣传你使用 FreeBSD，但这样做有助于你的努力和项目的发展。

让 FreeBSD 社区知道你的公司使用 FreeBSD，有助于提高吸引高质量人才的机会。广泛支持 FreeBSD 也意味着开发者中对它的关注度更高，这反过来为你的未来奠定了更健康的基础。

**支持 FreeBSD 开发者。** 有时候，将所需功能加入 FreeBSD 的最直接方法是支持已经在处理相关问题的开发者。帮助可以从硬件捐赠到直接的财政援助不等。在一些国家，向 FreeBSD 项目捐赠可以享受税收优惠。该项目有专门的 [捐赠联络员](https://www.freebsd.org/donations/) 来协助捐赠者。项目还维护一个网页，开发者在这里 [列出他们的需求](https://www.freebsd.org/donations/wantlist/)。

作为政策，FreeBSD 项目会在其网站上 [承认](https://docs.freebsd.org/en/articles/contributors/) 所有收到的贡献。

## 4. 结论

FreeBSD 项目的目标是创建并免费提供高质量操作系统的源代码。通过与 FreeBSD 项目的合作，你可以在多个产品开发场景中降低开发成本，并缩短上市时间。

我们考察了 FreeBSD 项目的特性，阐明了它为什么是成为组织产品战略一部分的优秀选择。然后，我们探讨了该项目的主流文化，并审视了与其开发者有效互动的方法。本文最后列出了可能对与项目合作的组织有帮助的最佳实践。

## 参考文献

\[Carp1996] [The Architectural Principles of the Internet](http://www.ietf.org/rfc/rfc1958.txt) B. Carpenter. The Internet Architecture Board.The Internet Architecture Board. Copyright® 1996.

\[ComGuide] [Committer’s Guide](https://docs.freebsd.org/en/articles/committers-guide/) The FreeBSD Project. Copyright® 2005.

\[GoldGab2005] [Innovation Happens Elsewhere: Open Source as Business Strategy](http://dreamsongs.com/IHE/IHE.html) Ron Goldman. Richard Gabriel. Copyright® 2005. Morgan-Kaufmann.

\[Hub1994] [Contributing to the FreeBSD Project](https://docs.freebsd.org/en/articles/contributing/) Jordan Hubbard. Copyright® 1994-2005. The FreeBSD Project.

\[McKu1999] [Soft Updates: A Technique for Eliminating Most Synchronous Writes in the Fast Filesystem](http://www.usenix.org/publications/library/proceedings/usenix99/mckusick.html) Kirk McKusick. Gregory Ganger. Copyright® 1999.

\[McKu1999-1] [Twenty Years of Berkeley Unix: From AT&T-Owned to Freely Redistributable](http://www.oreilly.com/catalog/opensources/book/kirkmck.html) Marshall Kirk McKusick. [Open Sources: Voices from the Open Source Revolution](http://www.oreilly.com/catalog/opensources/book/toc.html) O’Reilly Inc.. Copyright® 1993.

\[Mon2005] [Why you should use a BSD style license for your Open Source Project](https://docs.freebsd.org/en/articles/bsdl-gpl/) Bruce Montague. The FreeBSD Project. Copyright® 2005.

\[Nik2005] [A project model for the FreeBSD Project](https://docs.freebsd.org/en/books/dev-model/) Niklas Saers. Copyright® 2005. The FreeBSD Project.

\[Nor1993] [Tutorial on Good Lisp Programming Style](http://www.norvig.com/luv-slides.ps) Peter Norvig. Kent Pitman. Copyright® 1993.

\[Nor2001] [Teach Yourself Programming in Ten Years](http://www.norvig.com/21-days.html) Peter Norvig. Copyright® 2001.

\[Ray2004] [How to ask questions the smart way](http://www.catb.org/~esr/faqs/smart-questions.html) Eric Steven Raymond. Copyright® 2004.

\[RelEngDoc] [FreeBSD Release Engineering](https://docs.freebsd.org/en/articles/releng/) Murray Stokely. Copyright® 2001. The FreeBSD Project.
