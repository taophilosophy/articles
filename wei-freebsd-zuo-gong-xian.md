# 参与 FreeBSD

商标

FreeBSD 是 FreeBSD 基金会的注册商标。

IEEE，POSIX 和 802 是美国电气和电子工程师协会的注册商标。

制造商和销售商用来区分其产品的许多名称被称为商标。在本文件中出现这些名称时，如果 FreeBSD 项目意识到商标声明，这些名称旁边都会加上“™”或“®”符号。

摘要

本文介绍了个人或组织可以为 FreeBSD 项目做出贡献的不同方式。

---

所以你想为 FreeBSD 做贡献吗？那太好了！FreeBSD 依靠其用户群的贡献来生存。你的贡献不仅受到赞赏，而且对 FreeBSD 的持续增长至关重要。

大量且不断增加的国际贡献者，年龄和技术专长各异，正在开发 FreeBSD。总是有比可用人员更多的工作要做，并且总是需要更多的帮助。

作为志愿者，你做什么仅取决于你想做什么。不过，我们确实要求你了解 FreeBSD 社区其他成员对你的期望。在决定志愿服务之前，你可能需要考虑这一点。

FreeBSD 项目负责整个操作系统环境，而不仅仅是一个内核或一些分散的工具。因此，我们的 TODO 列表涵盖了非常广泛的任务：从文档编写、测试和演示，到系统安装程序和高度专业化的内核开发。任何技能水平的人，几乎在任何领域，都几乎肯定可以帮助该项目。

也鼓励从事 FreeBSD 相关企业的商业实体与我们联系。你需要特殊扩展来使你的产品工作吗？只要你的请求不过于离谱，我们将乐于接受你的请求。你是否正在开发增值产品？请告诉我们！我们也许可以在某些方面合作。自由软件世界正在挑战许多关于软件开发、销售和维护的现有假设，我们敦促你至少重新审视这一点。

## 1. 需要什么

下面的任务列表和子项目列表代表了各种待办事项列表和用户请求的混合体。

### 1.1. 持续进行中的非程序员任务

许多参与 FreeBSD 的人不是程序员。该项目包括文档编写者、网页设计师和支持人员。这些人只需要投入时间并愿意学习即可为项目贡献。

1. 定期阅读常见问题集和手册。如果有任何地方解释不清楚、模糊、过时或错误，请告诉我们。更好的是，发送给我们一个修复（AsciiDoc 并不难学，但对于纯文本提交没有异议）。
2. 帮助将 FreeBSD 文档翻译成你的母语。如果你的语言已经存在文档，你可以帮助翻译其他额外的文档或验证翻译是否最新和正确。首先查看 FreeBSD 文档项目入门指南中的翻译 FAQ。通过这样做，你并不会承诺自己翻译每一份 FreeBSD 文档-作为志愿者，你可以根据自己的意愿进行任意多或少的翻译。一旦有人开始翻译，其他人几乎总会加入进来。如果你只有时间或精力翻译文档的一部分，请翻译安装说明。
3. 偶尔（甚至定期）阅读 FreeBSD 常见问题邮件列表。与他人分享你的专业知识并帮助他们解决问题是非常令人满足的；有时你甚至可能会学到新东西！这些论坛还可以为你提供改进的想法。

### 1.2. 进行中的程序员任务

这里列出的大多数任务可能需要大量的时间投入，深入了解 FreeBSD 内核，或者两者兼而有之。然而，也有许多适合“周末黑客”的有用任务。

1. 如果你正在运行 FreeBSD-CURRENT 并且拥有良好的互联网连接，有一台 current.FreeBSD.org 每天构建一个完整的发行版-不时尝试从中安装最新版本，并报告任何过程中的失败。
2. 阅读 FreeBSD 问题报告邮件列表。可能会有你可以进行建设性评论或测试补丁的问题，或者你甚至可以尝试修复其中的一个问题。
3. 如果你知道已成功应用于-CURRENT 的任何 bug 修复，但在合理时间（通常是几周）后尚未合并到-STABLE，请给提交者发送一个礼貌的提醒。
4. 将贡献的软件移动到源代码树中的 src/contrib 目录。
5. 确保 src/contrib 中的代码是最新的。
6. 使用额外警告启用构建源码树（或其中的一部分），并清理警告。 可以通过选择一个构建并检查“LLVM/Clang Warnings”来找到构建警告列表。
7. 修复 ports 的警告，这些警告做了像使用 gets() 或包括 malloc.h 的已废弃的事情。
8. 如果你已经贡献了任何 ports 并且不得不进行 FreeBSD 特定的更改，请将你的补丁发送回原作者（这将使你在他们发布下一个版本时更容易）。
9. 获取正式标准的副本，如 POSIX®。比较 FreeBSD 的行为与标准要求的行为。如果行为有所不同，特别是在规范的微妙或隐蔽的角落，提交一个 PR。如果可以的话，想办法修复它并在 PR 中包含补丁。如果认为标准有误，请请求标准机构考虑这个问题。
10. 建议这个列表的进一步任务！

### 1.3. 通过 PR 数据库进行工作

FreeBSD PR 列表显示所有当前由 FreeBSD 用户提交的问题报告和改进建议。PR 数据库包括程序员和非程序员的任务。浏览未关闭的 PR，看看有没有什么引起你兴趣的内容。其中有一些可能是非常简单的任务，只需要额外的眼睛来审查并确认 PR 中的修复是否正确。其他一些可能要复杂得多，或者甚至根本没有包含修复。

从尚未分配给其他人的 PR 开始。如果 PR 已分配给其他人，但看起来你有能力处理，请给被分配的人发送电子邮件，询问是否可以开始处理该问题-他们可能已经准备好要测试的补丁，或者有更多你可以与他们讨论的想法。

### 1.4. 正在进行的 Ports 任务

Ports 是一个永远在进展中的工作。我们希望为用户提供一个易于使用、最新的、高质量的第三方软件库。我们需要人们捐献一些时间和精力来帮助我们实现这个目标。

任何人都可以参与进来，并且有很多不同的方式可以参与。为 ports 做贡献是回馈项目的绝佳方式。无论你是在寻找一个长期角色，还是一个雨天的有趣挑战，我们都希望得到你的帮助！

有很多简单的方法可以帮助你保持 ports 树的最新状态并保持其良好运转：

- 找一些酷炫或有用的软件，并为其创建一个 port。
- 有许多 ports 没有维护者。成为一个维护者并采纳一种无人维护的 port。
- 如果你已经创建或采纳了一个 port，请注意 port 管理者面临的挑战。
- 当你寻找一个快速的挑战时，你可以找到并修复一个损坏的 port。

### 1.5. 从“想法”页面中选择一个项目

FreeBSD 的项目列表和志愿者想法也适用于愿意为 FreeBSD 项目做出贡献的人。该列表定期更新，包含程序员和非程序员的项目信息。

## 2. 如何贡献

系统的贡献通常分为以下 5 个类别之一：

### 2.1. 报告漏洞和一般评论

一般技术兴趣的想法或建议应通过电子邮件发送到 FreeBSD 技术讨论邮件列表。同样，对此类事物感兴趣（并对大量邮件有耐心！）的人可以订阅 FreeBSD 技术讨论邮件列表。有关此事和其他邮件列表的更多信息，请参阅 FreeBSD 手册。

如果你提交了一个简单的补丁到 src 仓库，请考虑将其提交到项目的 GitHub 镜像作为拉取请求。适合提交的内容应包括：

- 它已经准备就绪，或者几乎可以提交了。提交者应该能够在不到 10 分钟的额外工作时间内提交这个补丁。
- 它通过了所有的 GitHub CI 作业。
- 你可以快速回应反馈。
- 它涉及的文件不到大约 10 个，更改不超过大约 200 行。比这更大的更改可能是可以的，或者你可能会被要求提交更易处理的更大的多个拉取请求。
- 每个逻辑更改都是拉取请求内的单独提交。每个更改的提交消息应遵循提交日志指南。
- 所有提交都有你的姓名和有效电子邮件地址，就像你想要在 FreeBSD 存储库中看到的那样作为作者。无法使用虚假的 github.com 地址。
- 在审查过程中，拉取请求的范围不应更改。如果审查建议扩展范围，请创建一个独立的拉取请求。
- 修复提交应与它们所修复的提交合并。你分支中的每个提交都应适用于 FreeBSD 的存储库。
- 提交应包含一个或多个 Signed-off-by: 行，其中写有全名和电子邮件地址，证明开发者证书的来源。

当更新拉取请求时，请使用强制推送而不是合并提交。更复杂的更改可以作为拉取请求提交，但如果太大、过于笨重、变得不活跃、需要社区进一步讨论或需要大量修订，则可能会被关闭。请避免创建大范围的清理补丁：它们太大，并且缺乏进行良好审查所需的焦点。错误的补丁可能会被重定向到更合适的论坛以解决补丁问题。

根据开发者的喜好，提交到 ports 仓库的拉取请求可能会或可能不会得到处理。目前，如果你遵循 ports 的提交流程，你将获得更好的体验。贡献给 ports。

文档团队也通过 GitHub 接受拉取请求，但尚未为此建立任何政策。

如果你发现错误或正在提交特定更改，请使用错误提交表单报告它。尽量填写每个错误报告字段。除非它们超过 65KB，在报告中直接包括任何补丁。如果补丁适合应用于源代码树，请在报告的摘要中放置 [PATCH] 。在包含补丁时，请勿使用剪切和粘贴，因为剪切和粘贴会将制表符转换为空格并使其无法使用。当补丁比 20KB 大很多时，请考虑在上传之前压缩它们（例如使用 gzip(1)或 bzip2(1)）。

提交报告后，你应该收到确认信息以及一个跟踪号码。保留此跟踪号码，以便你可以向我们提供有关问题的详细信息。

请参阅有关如何撰写良好问题报告的文章。

### 2.2. 文档更改

文档更改由 FreeBSD 文档项目邮件列表监督。请查阅 FreeBSD 文档项目入门指南获取完整的说明。使用与任何其他 bug 报告相同的方法提交修改和变更（即使是小的变更也欢迎！）。

### 2.3. 现有源代码的更改

对现有源代码的添加或更改是一个相对棘手的问题，很大程度上取决于你与当前 FreeBSD 开发状态的落后程度。FreeBSD 有一个特殊的持续发布版本，称为“FreeBSD-CURRENT”，它以多种方式提供，方便积极参与系统开发的开发人员使用。有关获取和使用 FreeBSD-CURRENT 的更多信息，请参阅 FreeBSD 手册。

使用较旧的源代码工作不幸意味着你的更改有时可能过于陈旧或分歧过大，难以重新集成到 FreeBSD 中。通过订阅 FreeBSD 公告邮件列表和 FreeBSD-CURRENT 邮件列表（讨论系统当前状态的地方），可以在一定程度上减少这种情况的发生。

假设你能够设法获取相对最新的源代码以进行更改，下一步就是生成一组 diff 并发送给 FreeBSD 维护人员。这可以使用 diff(1) 命令完成。

提交补丁的首选 diff(1) 格式是由 diff -u 生成的统一输出格式。

```
% diff -u oldfile newfile
```

或者

```
% diff -u -r -N olddir newdir
```

将为给定的源文件或目录层次结构生成一组统一的 diff。

有关更多信息，请参阅 diff(1)。

一旦你有一组差异（可以使用 patch(1) 命令进行测试），请将它们提交给 FreeBSD 作为 bug 报告的一部分。不要仅仅将差异发送到 FreeBSD 技术讨论邮件列表，否则它们可能会丢失！我们非常感谢你的提交（这是一个志愿项目！）；因为我们很忙，可能无法立即处理，但它将保留在 PR 数据库中，直到我们处理。在报告的简介中包含 [PATCH] 表明你的提交。

如果你认为合适（例如，你已添加、删除或重命名文件），请将你的更改捆绑成一个 tar 文件。使用 shar(1) 创建的归档也是可以接受的。

如果你的更改涉及潜在敏感性，比如你不确定版权问题，控制其进一步分发，则应将其直接发送至 core@FreeBSD.org，而不是提交为错误报告。 core@FreeBSD.org 只能联系到少数几个在 FreeBSD 上进行日常工作的人。请注意，这个团队也非常忙，因此你只应在真正必要时才发邮件给他们。

请参考 intro(9) 和 style(9) 以获取有关编码风格的一些信息。在提交代码之前，我们希望你至少了解一下这些信息。

### 2.4. 新代码或主要增值软件包

在 FreeBSD 中，如果要对大型工作进行重大贡献，或者向其添加重要的新功能，几乎总是需要将更改发送为 tar 文件或上传到 Web 或 FTP 站点，以便其他人访问。如果你无法访问 Web 或 FTP 站点，请在适当的 FreeBSD 邮件列表上询问是否有人可以为你托管这些更改。

在处理大量代码时，版权的敏感问题也总是会出现。FreeBSD 偏爱诸如 BSD 或 ISC 之类的自由软件许可证。有时也允许诸如 GPLv2 之类的 Copyleft 许可证。完整列表可以在核心团队许可政策页面上找到。

### 2.5. 金钱或硬件

我们总是非常高兴接受捐款以促进 FreeBSD 项目的事业，在像我们这样的志愿者努力中，一点点捐款可以发挥很大的作用！硬件捐赠对于扩大我们支持的外设清单也非常重要，因为我们通常缺乏资金来自己购买这些物品。

#### 2.5.1. 捐赠资金

FreeBSD 基金会是一个非营利、免税的基金会，旨在促进 FreeBSD 项目的目标。作为一个 501(c)3 实体，基金会通常免于美国联邦所得税和科罗拉多州所得税。对免税实体的捐赠通常可以从应税联邦收入中扣除。

捐赠可以以支票形式寄至：

美国科罗拉多州博尔德市 Broadway Street 3980 号 STE＃103-107 自由软件基金会

自由软件基金会还能够通过各种支付选项接受在线捐赠。

更多关于 FreeBSD 基金会的信息可以在 FreeBSD 基金会简介中找到。要通过电子邮件联系基金会，请写信至 info@FreeBSDFoundation.org。

#### 2.5.2. 捐赠硬件

FreeBSD 项目欣然接受可以好好利用的硬件捐赠。如果你有意捐赠硬件，请联系捐赠联络处。

## 3. 贡献于 ports

### 3.1. 接管一个无人维护的 port

#### 3.1.1. 选择一个无人维护的 port

接管 ports 的维护工作是参与的好方法，这些项目目前没有人维护。只有当有人自愿投入工作时，这些 ports 才会得到更新和修复。有许多未维护的 ports。建议从你经常使用的 port 开始采纳。

未维护的 ports 其 MAINTAINER 设置为 ports@FreeBSD.org 。许多未维护的 ports 可能有待更新，这可以在 FreeBSD Ports distfile 扫描器中看到。

在 PortsFallout 上可以看到一些有错误的未维护 ports 列表。

一些 ports 由于依赖关系和次要 port 关系影响了许多其他人。通常，我们希望人们在维护这种 ports 之前具有一些经验。

你可以通过查看名为 INDEX 的 ports 的主索引来判断一个 port 是否有依赖或次要 ports。（FreeBSD 的发布版本的文件名各不相同；例如，INDEX-13。）一些 ports 具有非默认 INDEX 构建中未包含的有条件的依赖关系。我们期望你能够通过查看其他 ports 的 Makefile 来识别这样的 ports。

#### 3.1.2. 如何采用 port

首先确保你理解你的 port 维护者所面临的挑战。还要阅读《Porter's Handbook》。请不要承担超出你能舒适处理的工作量。

你可以随时申请维护任何未维护的 port。只需将 MAINTAINER 设置为你自己的电子邮件地址，并发送带有这一更改的 PR（问题报告）。如果 port 存在构建错误或需要更新，你可能希望在同一 PR 中包含其他任何更改。这将有助于提高成功率，因为许多提交者不愿意将维护权委托给没有与 FreeBSD 有已知记录的人。提交修复构建错误或更新 ports 的 PR 是建立信誉的最佳方式之一。

将你的 PR 提交给产品 Ports & Packages 。提交者将审核你的 PR，提交更改，最终关闭 PR。有时这个过程可能需要一点时间（提交者也是志愿者）。

### 3.2。port 维护者的挑战

本节将让你了解为什么需要维护 ports，并概述 port 维护者的职责。

#### 3.2.1。为什么需要维护 ports

创建 port 是一项一次性任务。确保 port 是最新的并持续构建和运行需要持续的维护工作。维护者是那些抽出一部分时间来实现这些目标的人。

ports 需要进行维护的首要原因是为了将第三方软件的最新功能带给 FreeBSD 社区。另一个挑战是在 Ports 框架不断发展的同时保持个别 ports 的工作。

作为维护者，你将需要解决以下挑战：

- 新软件版本和更新。现有的移植软件的新版本和更新随时可用，需要将其纳入 Ports 中，以提供最新的软件。
- 依赖项的变更。如果对 port 的依赖关系进行了重大更改，可能需要更新它，以确保其继续正常工作。
- 影响依赖 ports 的变更。如果其他 ports 依赖于你维护的 port，你对 port 的更改可能需要与其他维护者协调。
- 与其他用户、维护者和开发者互动。作为维护者的一部分是承担支持角色。你不需要提供一般支持（但如果你选择这样做，我们欢迎）。你应该提供的是一个协调点，解决关于你 ports 的 FreeBSD 特定问题。
- Bug 追踪。port 可能会受到 FreeBSD 特定的 bug 影响。你需要在这些 bug 报告时进行调查、找到并修复这些 bug。在 port 进入 Ports Collection 之前，彻底测试以发现问题更好。
- ports 基础设施和政策的变化。偶尔，用于构建 ports 和软件包的系统会更新，或对基础设施产生影响的新建议出台。你应当了解这些变化，以防你的 ports 受到影响并需要更新。
- FreeBSD 正在持续开发中。软件、库、内核甚至政策的变化都可能导致端口的后续变更需求。

#### 3.2.2. 维护者职责

##### 3.2.2.1. 保持更新你的 Ports 

本节概述了保持你的 ports 最新的流程。

这是一个概述。有关升级 port 的详细信息，请参阅《波特手册》。

1. 关注更新，监视上游供应商关于软件新版本、更新和安全补丁的信息。公告邮件列表或新闻网页对此非常有用。有时用户会联系你，并询问你的 port 何时会更新。如果你正忙于其他事务或因任何原因暂时无法更新，请询问他们是否愿意帮助你提交更新。
   你可能还会收到来自 FreeBSD Ports Version Check 的自动电子邮件，通知你可用的 port 的 distfile 的新版本。有关该系统的更多信息（包括如何停止将来的电子邮件），将在消息中提供。
2. 合并更改 当它们变为可用时，请将更改合并到 port 中。 你需要能够在原始 port 和更新的 port 之间生成补丁。
3. 审查和测试 彻底审查和测试你的更改：

   - 在尽可能多的平台和架构上构建、安装和测试你的 port。一个 port 在一个分支或平台上成功，在另一个分支或平台上失败是很常见的。
   - 确保你的 port 的依赖项完整。推荐的做法是安装你自己的 ports tinderbox。有关更多信息，请参阅维护人员和贡献者的资源。
   - 检查打包清单是否最新。这涉及添加任何新文件和目录并删除未使用的条目。
   - 使用 portlint(1) 作为指南，验证你的 port。请参阅资源，了解 ports 的维护者和贡献者的重要信息，以了解如何使用 portlint。
   - 考虑你的 port 的更改是否会导致其他 ports 发生故障。如果是这种情况，请与这些 ports 的维护者协调更改。如果你的更新更改了共享库版本，这一点尤为重要；在这种情况下，至少，相关的 ports 需要进行 PORTREVISION 增加，以便它们可以通过自动化工具（例如 ports-mgmt/poudriere）自动升级。

4. 提交更改：通过提交包含更改说明和补丁的 PR，发送你的更新。请参考《编写 FreeBSD 问题报告》，了解如何撰写优秀的 PR。 
   
>请不要提交整个 port 的 shar(1) 归档文件，而是使用 git-format-patch(1) 或 diff(1) -ruN 。这样，提交者可以更容易地看到所做的具体更改。Porter’s Handbook 中关于升级的部分有更多信息。 
   
   
5. 等待 在某个阶段，会有提交者处理你的 PR。这可能需要几分钟，也可能需要一到两周的时间——所以请耐心等待。如果需要更长时间，请在邮件列表（FreeBSD ports 邮件列表）、IRC：EFNet 上的 #bsdports 或 Libera 上的 #freebsd-ports 等地方寻求帮助。
6. 给出反馈 如果提交者发现你的更改有问题，他们很可能会把它反馈给你。及时回复将有助于更快地提交你的 PR，并且在尝试解决任何问题时更有利于保持对话的连贯性。
7. 最后，你的更改将被提交，你的 port 将被更新。然后提交者将关闭 PR。就是这样！

##### 3.2.2.2. 确保你的 ports 继续正确构建

本节是关于发现并修复阻止你的 ports 正确构建的问题。

FreeBSD 仅保证 Ports 在 -STABLE 分支上运行。理论上，你应该能够通过运行每个稳定分支的最新版本来实现运行（因为 ABI 不应该更改），但如果你可以运行该分支，那甚至更好。

由于大多数 FreeBSD 安装都在 PC-兼容机器上运行（被称为 i386 架构），我们希望你继续在该架构上使 port 正常运行。我们希望 ports 也在在 amd64 架构上原生运行。如果你没有这两种类型的计算机，向他人寻求帮助是完全合理的。

>针对非 x86 机器的通常故障模式是原始程序员假设，例如指针是 int -s，或者使用了相对宽松的较旧的 gcc 编译器。越来越多的应用程序作者正在重新编写他们的代码以消除这些假设-但如果作者没有积极维护他们的代码，则你可能需要自行操作。

这些是你需要执行的任务，以确保你的 port 能够构建：

1. 注意构建失败 检查你的邮件以查看来自 pkg-fallout@FreeBSD.org 和 distfiles 扫描器的邮件，以查看是否有任何构建失败的 port 已过期。
2. 收集信息 一旦你意识到问题，收集信息以帮助你解决它。 pkg-fallout 报告的构建错误附带日志，将向你显示构建失败的位置。如果用户向你报告失败，请要求他们发送可能有助于诊断问题的信息，例如：

   - 构建日志
   - 用于构建 port 的命令和选项（包括在/etc/make.conf 中设置的选项）
   - 通过 pkg-info(8)显示的系统上已安装软件包列表
   - 它们正在运行的 FreeBSD 版本，如 uname(1)所示 -a
   - 当他们的 ports 上次更新时间
   - 当他们的 ports 树和 INDEX 上次更新时间

3. 调查并找到解决方案。不幸的是，没有一个简单的流程可以遵循来做到这一点。不过，请记住：如果遇到困难，请寻求帮助！FreeBSD ports 邮件列表是一个好的开始之处，上游开发人员通常也会提供帮助。
4. 提交更改。就像更新 port 一样，现在你应该集成更改，审查和测试，提交你的更改至 PR，并根据需要提供反馈。
5. 将补丁发送给上游作者。在某些情况下，你将不得不对 port 进行补丁以使其在 FreeBSD 上运行。有些（但不是所有）上游作者会接受这些补丁，并将其合并到下一个版本中。如果是这样，这甚至有助于其他基于 BSD 的系统上的用户，并可能节省重复的工作。请考虑作为一种礼貌将任何适用的补丁发送给作者。

##### 3.2.2.3. 调查与你的 port 相关的 bug 报告和 PRs。

此部分是关于发现和修复 bug。

FreeBSD 特定的 bug 通常是由于对构建和运行环境的假设不适用于 FreeBSD 导致的。你遇到此类问题的可能性较小，但可能更微妙和难以诊断。

这些是你需要执行的任务，以确保你的 port 继续正常工作：

1. 响应 bug 报告。Bug 可能通过电子邮件通过问题报告数据库向你报告。也可能直接由用户向你报告。
   你应该在 14 天内响应 PR 和其他报告，但请尽量不要等那么久。尽快做出响应，即使只是告知需要更多时间才能处理该 PR。
   如果你在 14 天后仍未回复，任何提交者都可以通过 maintainer-timeout 从你未回复的 PR 中提交。
2. 收集信息 如果报告 bug 的人没有提供修复，你需要收集能让你生成修复的信息。
   如果 bug 可重现，你可以自己收集大部分所需信息。如果不能，请报告 bug 的人帮你收集信息，例如：

   - 他们行为的详细描述，预期程序行为和实际行为
   - 用于触发错误的输入数据的副本
   - 关于他们的构建和执行环境的信息 - 例如，已安装软件包的列表和 env(1)的输出
   - 核心转储
   - 堆栈跟踪

3. 消除错误报告。有些错误报告可能是错误的。例如，用户可能只是错误地使用了程序；或者他们安装的软件包可能已过时，需要更新。有时报告的错误与 FreeBSD 无关。在这种情况下，将错误报告给上游开发人员。如果错误在你的能力范围之内修复，你还可以对 port 进行补丁，以便在下一个上游版本发布之前应用修复程序。
4. 处理解决方案与构建错误一样，你需要解决问题的方法。再次提醒，如果遇到困难，请及时寻求帮助！
5. 提交或批准更改就像更新 port 一样，现在你应该整合更改，审查和测试，并将更改提交到 PR 中（或者如果已经存在有关问题的 PR，则发送跟进）。如果其他用户在 PR 中提交了更改，你也可以发送跟进，说明你是否批准这些更改。

##### 3.2.2.4. 提供支持

作为维护者的一部分，你的工作之一是提供支持 - 不仅仅是针对软件本身 - 而是针对 port 和任何 FreeBSD 特有的怪癖和问题。用户可能会因为问题、建议、困难或补丁而联系你。大多数时候，他们的来信会与 FreeBSD 特定相关。

偶尔你可能需要运用外交技巧，友好地指引寻求一般支持的用户到适当的资源上。较少时会有人询问为什么 RPMS 不是最新的，或者如何让软件在 Foo Linux 下运行。趁机告诉他们你的 port 是最新的（如果是的话），并建议他们尝试 FreeBSD。

有时候，用户和开发者会认为你是一个时间宝贵的忙碌人，会为你做一些工作。例如，他们可能会：

- 提交一个 PR 或发送补丁来更新你的 port,
- 调查并可能提供 PR 的修复，或
- 否则提交更改到你的 port.

在这些情况下，你的主要义务是及时回复。再次提醒，非响应维护者的超时时间为 14 天。在此期限后，变更可以未经批准而提交。他们已经为你费心做了这些工作；所以请尽量及时回复。然后尽快审核、批准、修改或与他们讨论这些变更。

如果你能让他们感受到他们的贡献是被认可的（确实应该如此），那么你将更有机会说服他们在未来为你做更多的事情 :-).

### 3.3. 找到并修复损坏的 port

在一些很好的地方，可以找到需要关注的问题 port。

你可以使用网页界面访问问题报告数据库，搜索并查看未解决的 PR。大多数 ports PR 都是更新，但通过一点搜索和浏览摘要，你应该能找到一些有趣的工作。

PortsFallout 显示从 FreeBSD 软件包构建中收集的 port 问题。

对于维护的 port 也可以发送更改，但请记得询问维护者是否已经在处理该问题。

一旦发现 bug 或问题，收集信息，调查并修复！如果已有现有的 PR，请继续跟进。否则创建一个新的 PR。你的更改将被审核，如果一切检查无误，将被提交。

### 3.4. 何时放弃

随着你的兴趣和承诺的变化，你可能会发现自己再也没有时间继续进行某些（或全部）你的贡献。那没关系！如果你不再使用某个 port 或者对成为维护者失去了时间或兴趣，请告诉我们。这样，我们可以让其他人尝试解决 port 的现有问题，而不必等待你的回复。请记住，FreeBSD 是一个志愿者项目，如果维护 port 不再有趣，那么也许是时候让别人来做了！

无论如何，Ports 管理团队（ portmgr ）保留重新设置你的维护权限的权限，如果你在一段时间内没有积极维护你的 port。（目前，这被设置为 3 个月。）我们的意思是，在那段时间内存在未解决的问题或未处理的更新。

### 3.5. ports 维护者和贡献者资源

Porter's Handbook 是你的 ports 系统的搭车指南。记得随身携带！

编写 FreeBSD 问题报告介绍了如何最佳地构建和提交 PR。2005 年提交了超过一万 ports 个 PR！遵循本文将极大地帮助我们减少处理你的 PR 所需的时间。

问题报告数据库。

FreeBSD Ports distfile scanner (portscout)可以显示无法获取的 distfiles。你可以自行检查，也可以使用它查找需要更新其 MASTER_SITES 的 ports。

ports-mgmt/poudriere 是测试 port 的最彻底的方式，涵盖整个安装、打包和卸载周期。文档位于 poudriere GitHub 存储库中。

portlint(1)是一个应用程序，可用于验证你 port 符合许多重要的风格和功能指南。portlint 是一个简单的启发式应用程序，因此你应该将其仅用作指南。如果 portlint 建议看似不合理的更改，请参阅 Porter's Handbook 或寻求建议。

FreeBSD ports 邮件列表用于一般 ports 相关讨论。这是一个寻求帮助的好地方。你可以订阅，或阅读和搜索列表存档。阅读 FreeBSD ports bugs 邮件列表的存档和 SVN 提交消息以了解 head/ 分支的树也可能会感兴趣。

PortsFallout 是帮助搜索 FreeBSD 软件包问题存档的地方。

## 4. 在其他领域开始入门

寻找一些有趣的东西来开始，这些东西在本文中没有提到吗？FreeBSD 项目有几个维基页面，其中新贡献者可以从中获取如何开始的想法。

初级作业页面列出了一些项目，这些项目可能会引起刚刚开始使用 FreeBSD 并希望尝试有趣事物的人的兴趣。

创意页面包含项目中可以解决的各种“不错”或“有趣”的事物。
