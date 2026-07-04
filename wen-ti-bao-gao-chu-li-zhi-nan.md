# 问题报告处理指南

摘要

这些指南介绍了针对通过 Bugzilla 提交的 FreeBSD 问题报告（PR）的推荐处理实践。虽然这些指南是为 FreeBSD Bugzilla 数据库维护团队 [freebsd-bugbusters@FreeBSD.org](mailto:freebsd-bugbusters@FreeBSD.org) 开发的，但任何处理 FreeBSD 问题报告的人都应遵循这些指南。

---

## 1. 介绍

Bugzilla 是 FreeBSD 项目使用的问题管理系统。准确跟踪未解决的软件缺陷对 FreeBSD 的质量至关重要，正确使用该软件对项目的进展至关重要。

Bugzilla 的访问权限开放给整个 FreeBSD 社区。为了维护数据库的一致性并提供一致的用户体验，制定了涵盖 Bug 管理常见方面的准则，如提交后续、处理关闭请求等。

注意：在本文中，术语“PR”指“Bugzilla 问题报告”。

## 2. 问题报告生命周期

> **注意**
>
> 本节已过时，自 2026 年 1 月起正在重新编写。

- 报告人在网站上提交了一份 bug 报告。该 bug 处于 `Needs Triage` 状态。
- Jane Random BugBuster 确认 bug 报告包含足够的信息以便重现。如果没有，她会与报告人来回沟通以获取所需信息。此时 bug 被设置为 `Open` 状态。
- Joe Random Committer 对 PR 感兴趣并将其分配给自己，或者 Jane Random BugBuster 决定 Joe 最适合处理，并将其分配给他。应将 bug 设置为 `In Discussion` 状态。
- Joe 与发起人进行简短交流（确保所有内容都记录在审计记录中），并确定问题的原因。
- Joe 熬了一个通宵，快速制作了一个他认为能修复问题的补丁，并在后续提交中请求发起人测试它。然后，他将 PR 的状态设置为 `Patch Ready`。
- 几次迭代后，Joe 和发起人对补丁都感到满意，Joe 将其提交到 `-CURRENT`（如果问题不存在于 `-CURRENT` 中，则直接提交到 `-STABLE`），确保在提交日志中引用问题报告（如果发起人提交了全部或部分补丁，应在提交日志中致谢发起人），并在适当时开始 MFC 倒计时。该 bug 被设置为 `Needs MFC` 状态。
- 如果补丁不需要进行 MFC，Joe 会将 PR 关闭为 `Issue Resolved`。

> **注意**
>
> 许多报告提交时关于问题的信息很少，有些问题解决起来非常复杂，或者只是触及更大问题的表面；在这些情况下，获取解决问题所需的所有必要信息非常重要。如果所包含的问题无法解决，或者问题再次出现，则有必要重新打开该报告。

## 3. 问题报告状态

> **注意**
>
> 本节已过时，自 2026 年 1 月起正在重新编写。

当采取某些操作时，更新 PR 状态非常重要。状态应准确反映该问题上当前工作的状态。

示例：关于何时更改 PR 状态的小例子

当开发人员处理过 PR 并且负责人对修复感到满意时，他们将向 PR 提交后续，并将其状态更改为“feedback”。此时，发起人应在其实际环境中评估修复情况，并回应说明缺陷是否确实已修复。

问题报告可能处于以下状态之一：

`open`

初始状态；问题已指出，需要进行审查。

`analyzed`

问题经过审查，正在寻找解决方案。

`feedback`

进一步的工作需要来自发起人或社区的额外信息；可能是关于所提出解决方案的信息。

`patched`

已提交补丁，但还有一些事项（MFC，或可能需要发起人的确认）仍在等待中。

`suspended`

由于缺乏信息或资源，该问题未被处理。对于寻找项目的人来说，这是一个理想选择。如果问题完全无法解决，则会关闭而不是暂停。文档项目使用 `suspended` 状态来表示需要大量工作但目前无人有时间处理的愿望清单项。

`closed`

当任何更改已被集成、记录和测试后，或当放弃修复问题时，问题报告被关闭。

> **注意**
>
> `patched` 状态直接与 feedback 相关，因此如果发起人无法测试补丁，但它在你的测试中有效，你可以直接进入 `closed` 状态。

## 4. 问题报告类型

处理问题报告时，无论是作为直接访问问题报告数据库的开发人员，还是作为浏览数据库并提交补丁、评论、建议或更改请求的贡献者，你都会遇到几种不同类型的 PR。

- [未分配的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-unassigned)
- [已分配的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-assigned)
- [重复的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-dups)
- [陈旧的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-stale)
- [非 Bug 的 PR](https://docs.freebsd.org/en/articles/pr-guidelines/#pr-misfiled-notpr)

以下各节介绍了每种不同类型的 PR 的用途、何时属于这些类型，以及每种类型所接受的处理。

## 5. 未分配的 PR

当 PR 到达时，它们最初被分配给一个通用（占位符）受让人。这些总是以 `freebsd-` 开头。此默认值的确切值取决于类别；在大多数情况下，它对应于特定的 FreeBSD 邮件列表。以下是当前的列表，最常见的列在前面：

表 1. 默认受让人 - 最常见的

| 类型                       | 类别                  | 默认受让人         |
| -------------------------- | --------------------- | ------------------ |
| 基本系统                   | bin, conf, gnu, kern, misc | freebsd-bugs       |
| 架构特定                   | arm, amd64, i386, mips | freebsd-_arch_     |
| 架构特定：powerpc          | powerpc64             | freebsd-ppc        |
| 架构特定：riscv64          | riscv64               | freebsd-risc       |
| Ports 集合                 | ports                 | freebsd-ports-bugs |
| 随系统提供的文档           | docs                  | freebsd-doc        |
| FreeBSD 网页（不包括文档） | Website               | freebsd-www        |

表 2. 默认受让人 - 其他

| 类型                                                                                     | 类别      | 默认受让人        |
| ---------------------------------------------------------------------------------------- | --------- | ----------------- |
| 宣传推广工作                                                                             | advocacy  | freebsd-advocacy  |
| Java 虚拟机 (TM) 问题                                                                    | java      | freebsd-java      |
| 标准合规性                                                                               | standards | freebsd-standards |
| 线程库                                                                                   | threads   | freebsd-threads   |
| [usb(4)](https://man.freebsd.org/cgi/man.cgi?query=usb&sektion=4&format=html) 子系统     | usb       | freebsd-usb       |

不要惊讶于发现 PR 的提交人将其分配到了错误的类别。如果你更正了类别，请不要忘记修复指派。（特别是，我们的提交人似乎很难理解，他们的问题在 i386 系统上显现，可能是所有 FreeBSD 系统的通用问题，因此更适合归入 `kern`。当然，反过来也是成立的。）

任何人都可以将某些 PR 从这些通用受让人处重新分配。有几种类型的受让人：专门的邮件列表；邮件别名（用于某些有限兴趣的项目）；以及个人。

对于邮件列表作为受让人时，请在进行指派时使用长形式（例如，`freebsd-foo` 而不是 `foo`）；这将避免向邮件列表发送重复邮件。

> **注意**
>
> 由于自愿成为某些类型 PR 默认受让人的个人名单经常变化，因此更适合放在 [FreeBSD 维基](https://wiki.freebsd.org/AssigningPRs) 上。

以下是此类实体的示例列表；它可能不完整。

表 3. 常见受让人 - 基础系统

| 类型                                                                                                       | 建议类别 | 建议受让人             | 受让人类型 |
| ---------------------------------------------------------------------------------------------------------- | -------- | ---------------------- | ---------- |
| ARM® 架构特定问题                                                                                          | arm      | freebsd-arm            | 邮件列表   |
| MIPS® 架构特定问题                                                                                         | kern     | freebsd-mips           | 邮件列表   |
| PowerPC® 架构特定问题                                                                                      | kern     | freebsd-ppc            | 邮件列表   |
| 高级配置和电源管理 (acpi(4)) 问题                                                                          | kern     | freebsd-acpi           | 邮件列表   |
| 嵌入式或小型 FreeBSD 系统的问题（例如 NanoBSD/PicoBSD/FreeBSD-arm）                                        | kern     | freebsd-embedded       | 邮件列表   |
| FireWire® 驱动程序问题                                                                                     | kern     | freebsd-firewire       | 邮件列表   |
| 文件系统代码问题                                                                                           | kern     | freebsd-fs             | 邮件列表   |
| geom(4) 子系统问题                                                                                         | kern     | freebsd-geom           | 邮件列表   |
| [ipfw(4)](https://man.freebsd.org/cgi/man.cgi?query=ipfw&sektion=4&format=html) 子系统问题                 | kern     | freebsd-ipfw           | 邮件列表   |
| jail(8) 子系统                                                                                             | kern     | freebsd-jail           | 邮件列表   |
| Linux® 或 SVR4 仿真问题                                                                                    | kern     | freebsd-emulation      | 邮件列表   |
| 网络协议栈问题                                                                                             | kern     | freebsd-net            | 邮件列表   |
| pf(4) 子系统问题                                                                                           | kern     | freebsd-pf             | 邮件列表   |
| SCSI(4) 子系统问题                                                                                         | kern     | freebsd-scsi           | 邮件列表   |
| sound(4) 子系统问题                                                                                        | kern     | freebsd-multimedia     | 邮件列表   |
| wlan(4) 子系统和无线驱动程序的问题                                                                         | kern     | freebsd-wireless       | 邮件列表   |
| sysinstall(8) 或 bsdinstall(8) 的问题                                                                      | bin      | freebsd-sysinstall     | 邮件列表   |
| 系统启动脚本 (rc(8)) 问题                                                                                  | kern     | freebsd-rc             | 邮件列表   |
| VIMAGE 或 VNET 功能及相关代码的问题                                                                        | kern     | freebsd-virtualization | 邮件列表   |
| Xen 仿真问题                                                                                               | kern     | freebsd-xen            | 邮件列表   |

表 4. 常见受让人 - Ports 集合

| 类型                                                                                | 建议类别 | 建议受让人       | 受让人类型 |
| ----------------------------------------------------------------------------------- | -------- | ---------------- | ---------- |
| Ports 框架的问题（__不是__ 单个 port 的问题！）                                     | ports    | portmgr          | alias      |
| 由 [apache@FreeBSD.org](mailto:apache@FreeBSD.org) 维护的 port                     | ports    | apache           | 邮件列表   |
| 由 [autotools@FreeBSD.org](mailto:autotools@FreeBSD.org) 维护的 port                | ports    | autotools        | alias      |
| 由 [doceng@FreeBSD.org](mailto:doceng@FreeBSD.org) 维护的 port                      | ports    | doceng           | alias      |
| 由 [eclipse@FreeBSD.org](mailto:eclipse@FreeBSD.org) 维护的 port                    | ports    | freebsd-eclipse  | 邮件列表   |
| 由 [gecko@FreeBSD.org](mailto:gecko@FreeBSD.org) 维护的 port                        | ports    | gecko            | 邮件列表   |
| 由 [gnome@FreeBSD.org](mailto:gnome@FreeBSD.org) 维护的 port                        | ports    | gnome            | 邮件列表   |
| 由 [hamradio@FreeBSD.org](mailto:hamradio@FreeBSD.org) 维护的 port                  | ports    | hamradio         | alias      |
| 由 [haskell@FreeBSD.org](mailto:haskell@FreeBSD.org) 维护的 port                    | ports    | haskell          | alias      |
| 由 [java@FreeBSD.org](mailto:java@FreeBSD.org) 维护的 port                          | ports    | freebsd-java     | 邮件列表   |
| 由 [kde@FreeBSD.org](mailto:kde@FreeBSD.org) 维护的 port                            | ports    | kde              | 邮件列表   |
| 由 [mono@FreeBSD.org](mailto:mono@FreeBSD.org) 维护的 port                          | ports    | mono             | 邮件列表   |
| 由 [office@FreeBSD.org](mailto:office@FreeBSD.org) 维护的 port                      | ports    | freebsd-office   | 邮件列表   |
| 由 [perl@FreeBSD.org](mailto:perl@FreeBSD.org) 维护的 port                          | ports    | perl             | 邮件列表   |
| 由 [python@FreeBSD.org](mailto:python@FreeBSD.org) 维护的 port                      | ports    | freebsd-python   | 邮件列表   |
| 由 [ruby@FreeBSD.org](mailto:ruby@FreeBSD.org) 维护的 port                          | ports    | freebsd-ruby     | 邮件列表   |
| 由 [secteam@FreeBSD.org](mailto:secteam@FreeBSD.org) 维护的 port                    | ports    | secteam          | alias      |
| 由 [vbox@FreeBSD.org](mailto:vbox@FreeBSD.org) 维护的 port                          | ports    | vbox             | alias      |
| 由 [x11@FreeBSD.org](mailto:x11@FreeBSD.org) 维护的 port                            | ports    | freebsd-x11      | 邮件列表   |

如果 Ports PR 的 maintainer 是 ports 提交者，任何人都可以将其重新分配（但请注意，并非每个 FreeBSD 提交者必然是 ports 提交者，因此你不能仅凭电子邮件地址来判断。）

对于其他 PR，请不要将其重新分配给个人（除了你自己之外），除非你确信受让人真的想跟踪该 PR。这有助于避免没有人去修复特定问题的情况，因为每个人都认为受让人已经在处理它。

表 5. 常见受让人 - 其他

| 类型                                                          | 建议类别 | 建议受让人 | 受让人类型 |
| ------------------------------------------------------------- | -------- | ---------- | ---------- |
| 问题报告数据库的问题                                          | bin      | bugmeister | alias      |
| Bugzilla [网页表单](https://bugs.freebsd.org/submit/) 的问题 | doc      | bugmeister | alias      |

## 6. 已分配的 PR

如果一个 PR 的 `responsible` 字段设置为一个 FreeBSD 开发人员的用户名，则表示该 PR 已移交给该具体人员进行进一步工作。

除了受让人或 bugmeister 之外，任何人都不应该改动已分配的 PR。如果你有评论，请提交后续。如果出于某种原因你认为该 PR 应更改状态或重新分配，请向受让人发送消息。如果受让人在两周内未回复，请取消分配该 PR，然后自行处理。

## 7. 重复的 PR

如果你发现多个描述同一问题的 PR，请选择包含最多有用信息的一个，并关闭其他 PR，明确说明取代它的 PR 的编号。如果几个 PR 包含互不重叠的有用信息，请向其中一个 PR 提交后续，包含所有缺失的信息，包括对其他 PR 的引用；然后关闭其他 PR（这些 PR 现在已被完全取代）。

## 8. 陈旧的 PR

如果 PR 超过六个月未被修改，则视为陈旧。按照以下流程处理陈旧的 PR：

- 如果 PR 包含足够的细节，请尝试在 `-CURRENT` 和 `-STABLE` 中重现问题。如果成功，请提交后续详细描述你的发现，并尝试找人来指派它。如适当，将状态设置为 `analyzed`。
- 如果你知道 PR 描述的问题是使用错误（不正确的配置或其他原因）的结果，请提交后续说明发起人做错了什么，然后关闭 PR，并注明理由为“User error”或“Configuration error”。
- 如果 PR 描述的错误已在 `-CURRENT` 和 `-STABLE` 中修正，请关闭它并附上说明各分支中修正日期的消息。
- 如果 PR 描述的错误已在 `-CURRENT` 中修正，但在 `-STABLE` 中未修正，请尝试查明修正者计划何时 MFC，或尝试找到其他人（也许是你自己？）来完成。将状态设置为 `patched`，并将其分配给将执行 MFC 的人。
- 在其他情况下，请询问发起人确认问题在更新版本中是否仍然存在。如果发起人一个月内没有回复，请使用“Feedback timeout”注释关闭 PR。

## 9. 非 Bug 的 PR

开发人员发现看起来应该已发布到 FreeBSD 问题报告邮件列表或其他列表的 PR，应关闭该 PR，并在评论中告知提交人为什么这并不真正是一个 PR，以及该消息应该发布到何处。

Bugzilla 用于接收 PR 的电子邮件地址已作为 FreeBSD 文档的一部分发布，并已在网站上公布和列出。这意味着垃圾邮件发送者找到了它们。

每当你关闭这些 PR 之一时，请执行以下操作：

- 将组件设置为 `junk`（在 `Supporting Services` 下）。
- 将 Responsible 设置为 `nobody@FreeBSD.org`。
- 将 State 设置为 `Issue Resolved`。

将类别设为 `junk` 可以明显表明 PR 中没有有用内容，并有助于减少主要类别中的混乱。

## 10. 进一步阅读

这是一个与正确编写和处理问题报告相关的资源列表。这并不是完整的清单。

- 如何编写 FreeBSD 问题报告 - PR 发起人指南。
