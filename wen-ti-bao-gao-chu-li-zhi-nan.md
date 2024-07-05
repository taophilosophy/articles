# 问题报告处理指南

<details open="" data-immersive-translate-walked="aa33f219-cbc2-47a0-a708-10f8db3db829"><summary data-immersive-translate-walked="aa33f219-cbc2-47a0-a708-10f8db3db829" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

制造商和销售商用来区分其产品的许多名称都被声明为商标。在本文档中出现这些名称时，如果 FreeBSD 项目意识到其商标声明，这些名称将后跟“™”或“®”符号。

</details>

 摘要

这些指南描述了针对 FreeBSD 问题报告（PRs）的建议处理实践。虽然这些指南是为 FreeBSD PR 数据库维护团队 freebsd-bugbusters@FreeBSD.org 制定的，但任何与 FreeBSD PRs 一起工作的人都应遵循这些指南。

---

## 1. 介绍

Bugzilla 是 FreeBSD 项目使用的问题管理系统。准确跟踪未解决的软件缺陷对 FreeBSD 的质量至关重要，正确使用该软件对项目的进展至关重要。

Bugzilla 的访问权限开放给整个 FreeBSD 社区。为了维护数据库的一致性并提供一致的用户体验，已经制定了涵盖 Bug 管理常见方面的准则，如提交后续处理、处理关闭请求等。

## 2.问题报告生命周期

* 报告人在网站上提交了一个 bug 报告。该 bug 处于 Needs Triage 状态。
* 简·随机 BugBuster 确认 bug 报告具有足够的信息以便重现。如果没有，她会与报告人来回沟通以获取所需信息。此时 bug 被设置为 Open 状态。
* 乔·随机提交者对 PR 感兴趣并将其分配给自己，或者简·随机 BugBuster 决定乔最适合处理，并将其分配给他。应将 bug 设置为 In Discussion 状态。
* 乔与发起人进行简短交流（确保所有内容都记录在审计记录中），并确定问题的原因。
* 乔通宵达旦，草拟了一个他认为修复问题的补丁，并在后续提交中请求发起人测试它。然后，他将 PR 的状态设置为 Patch Ready 。
* 几次迭代后，乔和发起人都满意于补丁，乔将其提交到 -CURRENT （如果问题不存在于 -CURRENT 中，则直接提交到 -STABLE ），确保在提交日志中引用问题报告（如果提交者提交了全部或部分补丁，还要给予发起人荣誉），并且必要时开始 MFC 倒计时。该错误被设置为 Needs MFC 状态。
* 如果补丁不需要进行 MFC，则 Joe 会将 PR 关闭为 Issue Resolved 。

|  | 许多 PR 提交的信息很少，关于问题的信息很少，有些问题解决起来非常复杂，或者只是触及更大问题的表面；在这些情况下，获取解决问题所需的所有必要信息非常重要。如果无法解决所含问题，或者问题再次出现，则有必要重新打开 PR。 |
| -- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

## 3. 问题报告状态

更新 PR 状态是非常重要的，当采取某些操作时。状态应准确反映 PR 上的工作当前状态。

示例 1。关于何时更改 PR 状态的小例子

当开发人员已经处理过 PR 并且负责的人感觉修复工作正常时，他们将提交 PR 的后续，并将其状态更改为“反馈”。此时，发起者应在其上下文中评估修复情况，并响应指示缺陷是否已修复。

问题报告可能处于以下状态之一：

开放初始状态；问题已指出，需要进行审查。

分析问题已经审查，正在寻找解决方案。

进一步的反馈需要来自创始人或社区的额外信息；可能涉及所提出解决方案的信息。

补丁已提交，但还有一些事情（MFC，或者可能需要创始人的确认）仍在等待中。

暂停由于缺乏信息或资源而未解决该问题。这是寻找可承担项目的人的首选。如果无法解决问题，则会关闭而不是暂停。文档项目使用“暂停”来表示愿望清单项，这需要大量工作，目前没有人有时间。

一个问题报告在集成任何更改、记录和测试后关闭，或在放弃修复问题时关闭。

|  | “已修复”状态直接与反馈相关，因此，如果原始提交者无法测试补丁，但它在您的测试中有效，您可以直接进入“已关闭”状态。 |
| -- | ---------------------------------------------------------------------------------------------------------------------- |

## 4. 问题报告类型

处理问题报告时，无论是作为直接访问问题报告数据库的开发人员，还是作为浏览数据库并提交补丁、评论、建议或更改请求的贡献者，您都会遇到几种不同类型的 PR。

* [ 未分配的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-unassigned)
* [ 已分配的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-assigned)
* [ 重复的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-dups)
* [ 陈旧的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-stale)
* [ 非 Bug 的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-misfiled-notpr)

以下各节描述了每种不同类型的 PR 用于什么，当一个 PR 属于这些类型之一时，以及每种不同类型接收到的处理。

## 5. 未分配的 PRs

当 PR 到达时，它们最初被分配给一个通用（占位符）受让人。这些总是以 freebsd- 开头。此默认值的确切值取决于类别；在大多数情况下，它对应于特定的 FreeBSD 邮件列表。以下是当前的列表，最常见的列表首先列出：

默认受让人 - 最常见的

| 类型                       | 类别                                            | 默认受让人          |
| ---------------------------- | ------------------------------------------------- | --------------------- |
| 基本系统                   | bin, conf, gnu, kern, misc                  | freebsd-bugs     |
| 架构特定                   | alpha，amd64，arm，i386，ia64，powerpc，sparc64 | freebsd-*arch*     |
| ports 集合                 | ports                                           | freebsd-ports-bugs |
| 随系统提供的文档           | docs                                            | freebsd-doc     |
| FreeBSD 网页（不包括文档） | Website                                            | FreeBSD             |

表 2. 默认的指派者 - 其他

| 类型              | 分类 | 默认受理人      |
| ------------------- | ------ | ----------------- |
|advocacy efforts|advocacy|freebsd-advocacy|
|Java Virtual Machine™ problems|java|freebsd-java|
|standards compliance|standards|freebsd-standards|
|threading libraries|threads|freebsd-threads|
|[usb(4)](https://man.freebsd.org/cgi/man.cgi?query=usb&sektion=4&format=html) subsystem|usb|freebsd-usb|

不要感到惊讶，发起 PR 的人员将其分配给了错误的类别。如果您更正了类别，请不要忘记修复分配关系。(特别是，我们的提交者似乎很难理解他们的问题在 i386 系统上显现，这可能是适用于所有 FreeBSD 系统的通用问题，因此更适合于... 当然，相反的情况也成立。)

某些 PR 可能会被任何人重新分配，远离这些通用的受让人。有几种类型的受让人：专门的邮件列表；邮件别名（用于某些限定兴趣项目）；和个人。

对于那些邮件列表为受让人的情况，请在进行分配时使用长形式（例如， freebsd-foo 而不是 foo ）；这将避免向邮件列表发送重复邮件。

|  | 由于自愿成为某些类型的 PR 的默认受让人的个人名单经常发生变化，因此更适合用于 FreeBSD 维基。 |
| -- | --------------------------------------------------------------------------------------------- |

这是一份此类实体的示例列表；它可能不完整。

常见受让人 - 基础系统

| 类型                                                                      | 建议类别 | 建议受让人             | 受让人类型   |
| --------------------------------------------------------------------------- | ---------- | ------------------------ | -------------- |
| ARM® 架构特定问题                                                        | arm      | freebsd-arm            | 邮寄清单     |
| 特定于 MIPS®体系结构的问题                                               | kern     | freebsd-mips         | 邮件列表     |
| 问题特定于 PowerPC®架构                                                  | kern     | freebsd-ppc            | 邮件列表     |
| 高级配置和电源管理问题(acpi(4))                                           | kern     | freebsd-acpi           | 邮件列表     |
| 异步传输模式（ATM）驱动程序问题                                           | kern     |freebsd-atm   | 邮件列表     |
| 在嵌入式或小型 FreeBSD 系统中出现问题（例如 NanoBSD/PicoBSD/FreeBSD-arm） | kern     | 	freebsd-embedded        | 邮件列表     |
| FireWire® 驱动程序问题                                                   | kern    | freebsd-firewire       | 邮件列表     |
| 文件系统代码问题                                                          |kern    | freebsd-fs          | 邮件列表     |
| 与 geom(4)子系统的问题                                                    | kern     |freebsd-geom        | 邮件列表     |
| problem with the [ipfw(4)](https://man.freebsd.org/cgi/man.cgi?query=ipfw&sektion=4&format=html) subsystem                                               | kern     | freebsd-ipfw           | 邮件列表     |
| 集成服务数字网络（ISDN）驱动程序问题                                      | kern     | freebsd-isdn           | 邮件列表     |
| jail(8) 子系统                                                            | kern    | freebsd-jail       | 邮件列表     |
| 与 Linux®或 SVR4 仿真的问题                                              | kern     | 	freebsd-emulation          | 邮件列表     |
| 网络堆栈问题                                                              | kern     | freebsd-net          | 邮件列表     |
| pf(4) 子系统问题                                                          | kern | freebsd-pf             | 邮件列表     |
| SCSI(4) 子系统存在问题                                                    | kern    | freebsd-scsi           | 邮件列表     |
| 声音(4)子系统存在问题                                                     | kern     | freebsd-multimedia	         | 邮件列表     |
| wlan(4) 子系统和无线驱动程序的问题                                        | kern    |	freebsd-wireless    | 邮件列表     |
| sysinstall(8) 或 bsdinstall(8) 的问题                                     | bin      | freebsd-sysinstall     | 邮件列表     |
| 系统启动脚本出现问题（rc(8)）                                             | kern    | freebsd-rc             | 邮件列表     |
| 有关 VIMAGE 或 VNET 功能及相关代码的问题                                  | kern     | freebsd-virtualization | 邮件列表 |
| Xen 仿真问题                                                              | kern    | freebsd-xen            | 邮件列表     |

表 4. 常见受让人 - Ports 集合

| 类型                                         | 推荐的类别 | 建议的被分配人  | 分配人类型 |
| ---------------------------------------------- | ------------ | ----------------- | ------------ |
|problem with the ports framework (*not* with an individual port!)|ports|portmgr|alias|
|port which is maintained by [apache@FreeBSD.org](mailto:apache@FreeBSD.org)|ports|apache|邮件列表|
|port which is maintained by [autotools@FreeBSD.org](mailto:autotools@FreeBSD.org)|ports|autotools|alias|
|port which is maintained by [doceng@FreeBSD.org](mailto:doceng@FreeBSD.org)|ports|doceng|alias|
|port which is maintained by [eclipse@FreeBSD.org](mailto:eclipse@FreeBSD.org)|ports|freebsd-eclipse|邮件列表|
|port which is maintained by [gecko@FreeBSD.org](mailto:gecko@FreeBSD.org)|ports|gecko|邮件列表|
|port which is maintained by [gnome@FreeBSD.org](mailto:gnome@FreeBSD.org)|ports|gnome|邮件列表|
|port which is maintained by [hamradio@FreeBSD.org](mailto:hamradio@FreeBSD.org)|ports|hamradio|alias|
|port which is maintained by [haskell@FreeBSD.org](mailto:haskell@FreeBSD.org)|ports|haskell|alias|
|port which is maintained by [java@FreeBSD.org](mailto:java@FreeBSD.org)|ports|freebsd-java|邮件列表|
|port which is maintained by [kde@FreeBSD.org](mailto:kde@FreeBSD.org)|ports|kde|邮件列表|
|port which is maintained by [mono@FreeBSD.org](mailto:mono@FreeBSD.org)|ports|mono|邮件列表|
|port which is maintained by [office@FreeBSD.org](mailto:office@FreeBSD.org)|ports|freebsd-office|邮件列表|
|port which is maintained by [perl@FreeBSD.org](mailto:perl@FreeBSD.org)|ports|perl|邮件列表|
|port which is maintained by [python@FreeBSD.org](mailto:python@FreeBSD.org)|ports|freebsd-python|邮件列表|
|port which is maintained by [ruby@FreeBSD.org](mailto:ruby@FreeBSD.org)|ports|freebsd-ruby|邮件列表|
|port which is maintained by [secteam@FreeBSD.org](mailto:secteam@FreeBSD.org)|ports|secteam|alias|
|port which is maintained by [vbox@FreeBSD.org](mailto:vbox@FreeBSD.org)|ports|vbox|alias|
|port which is maintained by [x11@FreeBSD.org](mailto:x11@FreeBSD.org)|ports|freebsd-x11|邮件列表|


Ports具有是ports合作者的 PR 可能被任何人重新分配（但请注意，并非每个 FreeBSD 合作者必然是ports合作者，因此您不能仅通过电子邮件地址来判断。）

对于其他 PR，请不要将其重新分配给个人（除非您确信受让人真的想跟踪该 PR。这将有助于避免这样一种情况：由于每个人都认为受让人已经在解决特定的问题，所以没有人去解决这个问题的情况。

表 5. 常见受让人 - 其他

| 类型                       | 建议类别 | 建议的受让人 | 受让人类型 |
| ---------------------------- | ---------- | -------------- | ------------ |
|problem with PR database|bin|bugmeister|alias|
|problem with Bugzilla [web form](https://bugs.freebsd.org/submit/).|doc|bugmeister|alias|


## 6. 已分配的 PR

如果一个 PR 的 responsible 字段设置为一个 FreeBSD 开发人员的用户名，则表示该 PR 已移交给该具体人员进行进一步工作。

任何人都不应该触碰已分配的 PR，除了受让人或缺陷大师。如果您有评论，请提交后续。如果出于某种原因您认为该 PR 应更改状态或重新分配，请向受让人发送消息。如果受让人在两周内未回复，请取消分配该 PR，然后随意处理。

## 7. 重复的 PR

如果您找到描述同一问题的多个 PR，请选择包含最多有用信息的一个，并关闭其他 PR，明确说明替代 PR 的编号。如果几个 PR 包含互不重叠的有用信息，请在后续中向其中一个提交所有缺失的信息，包括对其他 PR 的引用；然后关闭其他 PR（这些 PR 现在完全被替代）。

## 8. 过期的 PR

如果 PR 在六个月内未被修改，则视为过期。执行以下过程处理过期的 PR：

* 如果 PR 包含足够的细节，请尝试在 -CURRENT 和 -STABLE 中重现问题。如果成功，请提交后续报告详细描述您的发现，并尝试找人来分配任务。如适当，将状态设置为“已分析”。
* 如果 PR 描述的问题是你知道是使用错误 (不正确的配置或其他原因) 的结果，请提交一篇后续说明原始作者做错了什么，然后关闭 PR，并注明理由为“用户错误”或“配置错误”。
* 如果 PR 描述的错误已在 -CURRENT 和 -STABLE 中修正，请关闭并附上修正日期的消息。
* 如果 PR 描述的错误已在 -CURRENT 中修正，但在 -STABLE 中未修正，请尝试查明修正者计划何时 MFC，或尝试找到其他人（也许是你自己？）来完成。将状态设置为“已修补”，将其分配给将执行 MFC 的人。
* 在其他情况下，请询问发起人确认问题在更新版本中是否仍然存在。如果发起人一个月内没有回复，请使用“反馈超时”注释关闭 PR。

## 9. 非错误修复 PR

开发人员发现看起来应该已发布到 FreeBSD 问题报告邮件列表或其他列表的 PR，应关闭该 PR，并在评论中告知提交者为什么这并不真正是一个 PR，以及该消息应该发布到何处。

Bugzilla 监听用于接收 PR 的电子邮件地址已作为 FreeBSD 文档的一部分发布，并已在网站上公布和列出。这意味着垃圾邮件发送者找到了它们。

每当您关闭这些 PR 之一时，请执行以下操作：

* 将组件设置为 junk （在 Supporting Services 下。
* 设置责任人为 nobody@FreeBSD.org 。
* 将状态设置为 Issue Resolved 。

将类别设为 junk 可以明显表明 PR 中没有有用内容，并有助于减少主要类别中的混乱。

## 10. 进一步阅读

这是一个与正确编写和处理问题报告相关的资源列表。这并不是完整的清单。

* 如何编写 FreeBSD 问题报告 - PR 发起者指南。
