# 使用 FreeBSD 开发产品

<details open="" data-immersive-translate-walked="8484330c-179c-41c7-ae2c-9eb9d2d5762e"><summary data-immersive-translate-walked="8484330c-179c-41c7-ae2c-9eb9d2d5762e" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

制造商和销售商用于区分其产品的许多名称被声明为商标。如果这些名称出现在本文件中，并且 FreeBSD 项目意识到了商标声明，这些名称将跟随“™”或“®”符号。

</details>

 摘要

FreeBSD 项目是一个全球范围内的、基于志愿者的、协作开发的项目，致力于开发一款便携且高质量的操作系统。FreeBSD 项目以宽松的许可证发布其产品的源代码，旨在鼓励其代码的使用。与 FreeBSD 项目合作可以帮助组织减少上市时间、降低工程成本并提高产品质量。

本文探讨了在电器和软件产品中使用 FreeBSD 代码时面临的问题。文章突出了 FreeBSD 的特性，说明了它作为产品开发的优秀基础。文章最后建议组织与 FreeBSD 项目合作的一些“最佳实践”。

---

## 1. Introduction

FreeBSD 今天作为一种高性能服务器操作系统而广为人知。它部署在全球数百万个 Web 服务器和面向互联网的主机上。FreeBSD 代码还构成了许多产品的重要部分，包括网络路由器、防火墙、存储设备，以及个人计算机。FreeBSD 的部分功能还被用于商业收缩包软件中（详见 FreeBSD 作为构建块的）。

在本文中，我们将 FreeBSD 项目视为软件工程资源——一组构建块和流程，您可以使用它们来构建产品。

虽然 FreeBSD 的源代码向公众免费分发，但要充分享受项目工作的好处，组织需要与项目合作。在本文的后续部分中，我们将讨论与项目合作的有效方式以及在这样做时需要避免的陷阱。

读者须知。作者认为本文列出的 FreeBSD 项目特性在文章构思和撰写时（2005 年）基本属实。然而，读者应该记住，开源社区使用的实践和流程随时间可能会发生变化，因此本文中的信息应被视为指示性而非规范性。

### 1.1. 目标受众

这份文件对以下广泛群体的人可能感兴趣：

* 产品公司的决策者，希望找到方法来提高其产品质量，在长期内降低上市时间和降低工程成本。
* 寻求在利用“开源”方面的最佳实践的技术顾问。
* 对开源项目动态感兴趣的行业观察者。
* 希望使用 FreeBSD 并寻找回馈方式的软件开发人员。

### 1.2. 文章目标

阅读本文后，您应该拥有：

* 理解 FreeBSD 项目的目标及其组织结构。
* 理解其开发模型和发布工程流程。
* 了解传统企业软件开发流程与 FreeBSD 项目使用的流程之间的区别。
* 了解项目使用的沟通渠道及您可以期待的透明度水平。
* 了解与项目协作的最佳方式-如何最有效地降低工程成本，改善上市时间，管理安全漏洞，并在 FreeBSD 项目发展的同时保留与您产品的未来兼容性。

### 1.3. 文章结构

文章的其余部分结构如下：

* 作为一组构建模块的 FreeBSD 介绍了 FreeBSD 项目，探讨了其组织结构，关键技术和发布工程过程。
* 与 FreeBSD 合作介绍了与 FreeBSD 项目合作的方法。它考察了企业在与 FreeBSD 等志愿项目合作时遇到的常见陷阱。
* 结论总结。

## 2. FreeBSD 作为一组构建模块

FreeBSD 是构建产品的极好基础：

* FreeBSD 源代码是以宽松的 BSD 许可证分发的，最大程度地促进其在商业产品中的采用，减少麻烦。
* FreeBSD 项目具有可以利用的优秀工程实践。
* 项目提供了对其工作过程的异常透明度，使使用其代码的组织能够有效规划未来。
* FreeBSD 项目的文化延续自加利福尼亚大学伯克利分校的计算机科学研究组 [McKu1999-1]，促进了高质量的工作。FreeBSD 中的一些特性定义了技术的最新水平。

[GoldGab2005] 更详细地探讨了在商业上使用开源的原因。对于组织而言，使用 FreeBSD 组件在其产品中的好处包括更短的上市时间、更低的开发成本和更低的开发风险。

### 2.1. 使用 FreeBSD 进行构建

这里是一些组织如何使用 FreeBSD 的方式：

* 作为库和实用程序测试代码的上游来源。通过成为项目的“下游”，组织利用了上游代码接收的新功能、错误修复和测试。
* 作为嵌入式操作系统（例如用于 OEM 路由器和防火墙设备）。在这种模型中，组织使用定制的 FreeBSD 内核和应用程序集，以及用于其设备的专有管理层。 OEM 受益于 FreeBSD 项目上游添加的新硬件支持，并从基础系统接收的测试中受益。FreeBSD 附带一个自助开发环境，可轻松创建这种配置。
* 作为 Unix 兼容环境，用于高端存储和网络设备的管理功能，运行在独立的处理器“刀片”上。FreeBSD 提供用于创建专用操作系统和应用程序映像的工具。其 BSD Unix API 的实现是成熟和经过测试的。 FreeBSD 还可以为高端设备的其他组件提供稳定的交叉开发环境。
* 作为从全球开发人员团队获得广泛测试和支持的工具，用于非关键“知识产权”。在此模型中，组织向 FreeBSD 项目贡献有用的基础架构框架（例如，请参阅 netgraph(3)）。代码获得的广泛曝光有助于快速识别性能问题和错误。顶尖开发人员的参与还会导致对贡献组织也有益的基础架构扩展。
* 作为支持诸如 RTEMS 和 eCOS 等嵌入式操作系统的交叉开发的开发环境。在 FreeBSD 的 36000 多个应用程序中，有许多完整的开发环境被移植和打包。
* 作为在另一个专有操作系统中支持类 Unix API 的一种方式，增加其对应用程序开发人员的吸引力。在专有操作系统中，FreeBSD 的内核和应用程序的部分被“移植”以在其他任务旁边运行。稳定且经过充分测试的 Unix™ API 实现的可用性可以降低将流行应用程序移植到专有操作系统的工作量。由于 FreeBSD 提供了其内部工作的高质量文档，并且拥有有效的漏洞管理和发布工程流程，因此保持最新状态的成本保持较低。

### 2.2. 技术

FreeBSD 项目支持众多技术。以下列出了其中一些:

* 一个完整的系统，可以跨越多种架构进行托管:
* 一个模块化的对称多处理能力内核，带有可加载内核模块和灵活易用的配置系统。
* 在接近机器速度的情况下支持 Linux™ 和 SVR4 二进制文件的仿真。支持二进制 Windows™ (NDIS) 网络驱动程序。
* 用于许多编程任务的库：存档程序，FTP 和 HTTP 支持，线程支持，以及完整的类 POSIX™ 编程环境。
* 安全功能：强制访问控制（mac(9)），jail (jail(2))，ACL 和内核中的加密设备支持。
* 网络功能：防火墙、QoS 管理、支持许多扩展的高性能 TCP/IP 网络。 FreeBSD 的内核 Netgraph（netgraph(4)）框架允许内核网络模块以灵活的方式连接在一起。
* 支持存储技术：光纤通道、SCSI、软件和硬件 RAID、ATA 和 SATA。 FreeBSD 支持许多文件系统，其本机 UFS2 文件系统支持软更新、快照和非常大的文件系统大小（每个文件系统 16TB）[McKu1999]。
  FreeBSD 的内核 GEOM（geom(4)）框架允许内核存储模块以灵活的方式组合。
* 超过 36000 个移植应用程序，包括商业和开源应用程序，均通过 FreeBSD ports进行管理。

### 2.3. 组织结构

FreeBSD 的组织结构是非等级的。

本质上，FreeBSD 有两种贡献者，FreeBSD 的一般用户和对源代码库有写访问权（在行话中称为提交者）的开发人员。

第一组中有成千上万的贡献者；FreeBSD 的绝大多数贡献都来自这一组的个人。对代码库的提交权限（写访问权）授予那些持续为项目做出贡献的个人。提交权限伴随着额外的责任，新提交者会被分配导师来帮助他们学习规则。

![freebsd organization](https://docs.freebsd.org/images/articles/building-products/freebsd-organization.png)

图 1. FreeBSD 组织结构

冲突解决是由从提交者组中选出的九名“核心团队”执行的。

FreeBSD 没有“公司”提交者。单个提交者需要对他们引入代码的更改负责。FreeBSD 提交者指南[ComGuide]记录了提交者的规则和责任。

FreeBSD 的项目模型在[Nik2005]中被详细检查。

### 2.4. FreeBSD 发布工程流程

FreeBSD 的发布工程流程在确保其发布版本质量上起着重要作用。在任何时候，FreeBSD 的志愿者支持多个代码行（FreeBSD 发布分支）：

* 新功能和破坏性代码输入开发分支，也被称为-CURRENT 分支。
* -STABLE 分支是定期从 HEAD 分支分支出来的代码线。只有经过测试的代码才允许进入-STABLE 分支。新特性只有在-CURRENT 分支中经过测试并稳定后才允许进入。
* -RELEASE 分支由 FreeBSD 安全团队维护。只允许针对关键问题的 bug 修复进入-RELEASE 分支。

![freebsd branches](https://docs.freebsd.org/images/articles/building-products/freebsd-branches.png)

图 2. FreeBSD 发布分支

代码行在有用户和开发者的兴趣时会持续存在。

机器架构被分为“层级”；第一层级的架构由项目的发布工程团队和安全团队全面支持，第二层级的架构在最佳努力的基础上提供支持，实验性的架构组成第三层级。支持的架构列表是 FreeBSD 文档集的一部分。

发行工程团队在项目的网站上发布了 FreeBSD 未来版本的路线图。路线图中列出的日期并不是最终期限；FreeBSD 的发布时间取决于其代码和文档的准备情况。

FreeBSD 的发布工程过程在 [RelEngDoc] 中描述。

## 3. 与 FreeBSD 合作

像 FreeBSD 这样的开源项目提供非常高质量的成品代码。

使用高质量的源代码可以降低初始开发成本，但随着时间推移，管理变更的成本开始主导。随着计算环境多年来的变化和新的安全漏洞的发现，您的产品也需要变化和适应。使用开源代码最好不是一次性活动，而是一个持续的过程。最好与活跃的社区合作，即具有活跃社区、明确目标和透明工作风格的项目。

* FreeBSD 拥有一个活跃的开发者社区。在撰写本文时，来自世界各有人口的大陆的成千上万的贡献者参与其中，并且有超过 300 人可以访问该项目的源代码库。
* FreeBSD 项目的目标是 [Hub1994]:

  * 开发高质量的操作系统，适用于流行的计算机硬件，并，
  * 将我们的工作以自由许可证提供给所有人。
* FreeBSD 拥有开放和透明的工作文化。该项目中几乎所有讨论都通过电子邮件进行，在公共邮件列表上也进行了归档以供后人参考。项目的政策已记录并维护在修订控制下。对该项目的参与对所有人开放。

### 3.1. 理解 FreeBSD 文化

要能有效地与 FreeBSD 项目合作，您需要理解该项目的文化。

由志愿者驱动的项目遵循与营利性企业不同的规则。企业在进入开源世界时常犯的一个常见错误是低估这些差异。

动机。在 FreeBSD 中，大多数贡献都是在没有金钱奖励的情况下自愿完成的。激励个人的因素是复杂的，从利他主义到对 FreeBSD 试图解决的问题感兴趣。在这种环境中，“优雅从不是可选的”[Nor1993]。

长远视角。FreeBSD 的根源可以追溯到近二十年前加利福尼亚大学伯克利分校的计算机科学研究组的工作。一些最初的 CSRG 开发人员仍与该项目有关。

该项目重视长期视角[Nor2001]。在项目中经常遇到的首字母缩略词是 DTRT，它代表“做正确的事情”。

开发过程。 计算机程序是沟通工具：在一个级别上，程序员使用精确的符号向一个工具（编译器）传达他们的意图，该工具将其指令翻译为可执行代码。在另一个级别上，相同的符号用于两个程序员之间的意图传达。

在项目中很少使用正式规范和设计文档。清晰和书写良好的代码以及书写良好的变更日志（一个样本变更日志条目）代替了它们。FreeBSD 的开发通过“粗略一致和可运行的代码”[Carp1996]进行。

```
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

一个样本变更日志条目

程序员之间的沟通通过使用共同的编码标准风格得以加强。

通信渠道。 FreeBSD 的贡献者遍布世界各地。 电子邮件（在较小程度上，IRC）是该项目中首选的沟通方式。

### 3.2. 与 FreeBSD 项目合作的最佳实践

我们现在来看一些在产品开发中充分利用 FreeBSD 的最佳实践。

为长期计划设置有助于跟踪 FreeBSD 开发的过程。例如：

跟踪 FreeBSD 源代码。该项目通过 svnsync 使其 SVN 存储库镜像变得容易。拥有完整的源代码历史在调试复杂问题时非常有用，并且可以深入了解原始开发人员的意图。使用功能强大的源代码控制系统，可以轻松地在上游 FreeBSD 代码库和您自己的内部代码之间合并更改。

使用 svn blame 生成的带注释源代码清单显示了通过样本变更日志引用的文件的部分带注释清单。每行源代码的祖先清晰可见。显示了 FreeBSD 的每个文件的历史的带注释清单可在网上找到。

```
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

使用 svn blame 生成的带注释源代码清单

使用一位门卫。指定一位门卫来监视 FreeBSD 的开发，留意可能会影响您产品的变更。

报告问题上游。 如果您注意到您正在使用的 FreeBSD 代码中存在错误，请提交错误报告。 这一步有助于确保您下次从上游获取代码时无需修复错误。

利用 FreeBSD 的发布工程努力从 FreeBSD 的-STABLE 开发分支中使用代码。 这些开发分支得到 FreeBSD 发布工程和安全团队的正式支持，并由经过测试的代码组成。

捐助代码以减少成本开发产品相关成本中的主要部分是维护成本。 通过向项目捐赠非关键代码，您的代码会得到比其他情况下更广泛的曝光。 这反过来又导致更多的错误和安全漏洞被发现并修复，以及性能异常被识别和修复。

为有严格期限的产品有效获取支持的建议是，建议您雇佣开发人员或与具有 FreeBSD 经验的公司签订咨询协议。 FreeBSD 相关的就业邮件列表是一个寻找人才的有用沟通渠道。 FreeBSD 项目维护着承担 FreeBSD 工作的顾问和咨询公司的画廊。 BSD 认证组为所有主要的 BSD 衍生操作系统提供认证。

对于不太关键的需求，您可以在项目邮件列表上寻求帮助。在寻求帮助时可以遵循的有用指南在[Ray2004]中给出。

宣传你的参与您并不需要宣传您对 FreeBSD 的使用，但这样做有助于您的努力以及项目的努力。

让 FreeBSD 社区知道您的公司使用 FreeBSD 有助于提高吸引高质量人才的机会。对 FreeBSD 的大量支持意味着开发人员对其有更多的关注。这反过来为您的未来打下更健康的基础。

支持 FreeBSD 开发者有时，将期望的功能直接引入 FreeBSD 的最直接方法是支持已经在研究相关问题的开发者。支持范围从硬件捐赠到直接财务援助不等。在一些国家，对 FreeBSD 项目的捐赠享有税收优惠。该项目有专门的捐赠联络人协助捐赠者。该项目还维护一个网页，开发者在其中列出他们的需求。

作为政策，FreeBSD 项目会在其网站上承认收到的所有捐赠。

## 4. 结论

FreeBSD 项目的目标是创建并赠送高质量操作系统的源代码。通过与 FreeBSD 项目合作，您可以减少开发成本，并改善在许多产品开发方案中的上市时间。

我们检查了 FreeBSD 项目的特点，使其成为组织产品战略的绝佳选择。然后，我们研究了该项目的主流文化，并检查了与其开发者有效互动的方法。本文最后给出了一系列能帮助与该项目合作的组织的最佳实践清单。

## 参考文献

[Carp1996] [The Architectural Principles of the Internet](http://www.ietf.org/rfc/rfc1958.txt) B. Carpenter. The Internet Architecture Board.The Internet Architecture Board. Copyright® 1996.

[ComGuide] [Committer’s Guide](https://docs.freebsd.org/en/articles/committers-guide/) The FreeBSD Project. Copyright® 2005.

[GoldGab2005] [Innovation Happens Elsewhere: Open Source as Business Strategy](http://dreamsongs.com/IHE/IHE.html) Ron Goldman. Richard Gabriel. Copyright® 2005. Morgan-Kaufmann.

[Hub1994] [Contributing to the FreeBSD Project](https://docs.freebsd.org/en/articles/contributing/) Jordan Hubbard. Copyright® 1994-2005. The FreeBSD Project.

[McKu1999] [Soft Updates: A Technique for Eliminating Most Synchronous Writes in the Fast Filesystem](http://www.usenix.org/publications/library/proceedings/usenix99/mckusick.html) Kirk McKusick. Gregory Ganger. Copyright® 1999.

[McKu1999-1] [Twenty Years of Berkeley Unix: From AT&amp;T-Owned to Freely Redistributable](http://www.oreilly.com/catalog/opensources/book/kirkmck.html) Marshall Kirk McKusick. [Open Sources: Voices from the Open Source Revolution](http://www.oreilly.com/catalog/opensources/book/toc.html) O’Reilly Inc.. Copyright® 1993.

[Mon2005] [Why you should use a BSD style license for your Open Source Project](https://docs.freebsd.org/en/articles/bsdl-gpl/) Bruce Montague. The FreeBSD Project. Copyright® 2005.

[Nik2005] [A project model for the FreeBSD Project](https://docs.freebsd.org/en/books/dev-model/) Niklas Saers. Copyright® 2005. The FreeBSD Project.

[Nor1993] [Tutorial on Good Lisp Programming Style](http://www.norvig.com/luv-slides.ps) Peter Norvig. Kent Pitman. Copyright® 1993.

[Nor2001] [Teach Yourself Programming in Ten Years](http://www.norvig.com/21-days.html) Peter Norvig. Copyright® 2001.

[Ray2004] [How to ask questions the smart way](http://www.catb.org/~esr/faqs/smart-questions.html) Eric Steven Raymond. Copyright® 2004.

[RelEngDoc] [FreeBSD Release Engineering](https://docs.freebsd.org/en/articles/releng/) Murray Stokely. Copyright® 2001. The FreeBSD Project.

---

FreeBSD 的源代码库包含了该项目自创立以来的历史，并且有 CD-ROM 可用，其中包含 CSRG 的早期代码。
