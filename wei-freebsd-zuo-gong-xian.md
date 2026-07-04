# 参与 FreeBSD

- 原文位于 [Contributing to FreeBSD](https://docs.freebsd.org/en/articles/contributing/)

## 摘要

本文介绍了个人和组织参与 FreeBSD 项目的各种方式。

所以你想参与 FreeBSD 吗？那太好了！FreeBSD *依靠* 用户群的贡献才能生存。你的贡献不仅会得到感谢，它们对于 FreeBSD 的持续发展也至关重要。

来自全球的众多且不断增长的贡献者，年龄段和技术专长领域各不相同，都在开发 FreeBSD。要干的工作总是比能干活的人多，更多的帮助总是受欢迎的。

作为志愿者，你所做的事仅限于你想做的。不过，我们的确希望你能够意识到，FreeBSD 社区中的其他成员对你的期望。在决定是否志愿参与之前，你可能需要考虑这一点。

FreeBSD 项目负责整个操作系统环境，而不单单是内核和一些零散的工具。因此，我们的 **TODO** 列表涵盖了各种各样的任务：从文档编写、Beta 测试和展示，到系统安装器和高度专业化的内核开发。几乎所有技能水平的人，几乎在任何领域，都能为 FreeBSD 做出贡献。

我们也鼓励从事与 FreeBSD 相关商业活动的实体与我们联系。你是否需要特殊的扩展来使你的产品正常工作？只要你的请求不过于离奇，你会发现我们乐于接受。你是否正在开发一款增值产品？请告知我们！我们或许能在某些方面合作。自由软件世界正在挑战许多现有的关于软件开发、销售和维护的假设，我们敦促你至少重新审视它。

## 1. 需要什么

以下任务和子项目的列表代表了各种 **TODO** 列表和用户请求的结合体。

### 1.1. 持续的非程序员任务

许多参与 FreeBSD 的人并不是程序员。项目包括文档编写者、网页设计师和支持人员。这些人所需要做的只是投入时间并愿意学习。

1. 定期阅读 FAQ 和手册。如果发现任何解释不清、含糊不清、过时或错误的内容，请告诉我们。更好的是，给我们提供修正（AsciiDoc 并不难学习，但也可以提交纯文本格式）。
2. 帮助将 FreeBSD 文档翻译成你的母语。如果你的语言已经有了文档，你可以帮助翻译更多文档，或者验证现有翻译是否最新且正确。首先可以查看 [FreeBSD 文档翻译 FAQ](https://docs.freebsd.org/en/books/fdp-primer/#translations)。你不必承诺翻译每一篇 FreeBSD 文档，作为志愿者，你可以按自己的意愿决定翻译多少。一旦有人开始翻译，其他人几乎总是会加入进来。如果你只有时间和精力翻译一部分文档，请翻译安装说明。
3. 偶尔（或定期）阅读 [FreeBSD 通用问题邮件列表](https://lists.freebsd.org/subscription/freebsd-questions)。分享你的专业知识并帮助他人解决问题可以让人很有满足感；有时你甚至能自己学到新东西！这些论坛也可以是改进事项的灵感来源。

### 1.2. 持续的程序员任务

这里列出的大多数任务可能需要大量的时间投入、对 FreeBSD 内核的深入了解，或者两者兼备。不过，也有很多适合“周末黑客”的有用任务。

1. 如果你使用 FreeBSD-CURRENT 并且有良好的互联网连接，有一台名为 `current.FreeBSD.org` 的机器每天构建一次完整版本——不时尝试从中安装最新版本，并报告安装过程中的任何失败。
2. 了解已经提出的更改或问题。可能有你可以做出建设性评论的问题，或者你可以测试补丁。你甚至可以尝试自己修复其中一个问题。
3. 一种方法是阅读 [FreeBSD 问题报告邮件列表](https://lists.freebsd.org/subscription/freebsd-bugs)（针对通过 Bugzilla 提交的问题）。
4. 如果你知道某个 bug 修复已成功应用到 -CURRENT，但经过一段合理时间（通常是几周）后尚未合并到 -STABLE，请礼貌提醒提交者。
5. 将贡献的软件移到源代码树中的 **src/contrib** 目录。
6. 确保 **src/contrib** 中的代码是最新的。
7. 使用开启额外警告的方式构建源代码树（或只是部分代码），并清理警告。也可以从我们的 [CI](https://ci.freebsd.org/) 获取构建警告列表，选择构建并查看 “LLVM/Clang 警告”。
8. 修复那些使用了弃用做法（例如使用 `gets()` 或包含 **malloc.h**）的 Port 警告。
9. 如果你贡献了任何 Port，并且你做了 FreeBSD 特定的修改，请将你的补丁发回给原作者（这将使他们在发布下一个版本时让你的生活更轻松）。
10. 获取正式标准的副本，如 POSIX®。比较 FreeBSD 的行为与标准要求的行为。如果行为不同，特别是在规范的微妙或晦涩角落，提交问题报告。如果可能，想办法修复它，并在问题报告中附上补丁。如果你认为标准是错的，请向标准化机构提出该问题。
11. 为这个列表建议更多的任务！

### 1.3. 处理 Bugzilla 问题报告数据库

[FreeBSD 问题报告列表](https://bugs.freebsd.org/search/) 显示了所有当前活跃的问题报告和功能增强请求，这些报告和请求由 FreeBSD 用户通过 Bugzilla 提交。该数据库包括程序员和非程序员任务。浏览这些未关闭的 PR，看看是否有任何任务吸引你的兴趣。有些可能是非常简单的任务，只需要多一双眼睛来审查并确认修复是否合适。其他的可能要复杂得多，甚至可能没有任何修复内容。

从那些尚未分配给其他人的 PR 开始。如果 PR 已分配给其他人，但看起来是你能处理的任务，可以给被分配的人发邮件，询问是否可以继续处理—他们可能已经准备好了补丁，或者有其他想法可以和你讨论。

### 1.4. 持续的 Ports 任务

Ports 集合是一项持续进行中的工作。我们希望为用户提供易于使用、始终保持最新的高质量第三方软件库。我们需要人们捐献一些时间和精力来帮助我们实现这一目标。

所有人都可以参与，并且有许多不同的方式来参与。为 Ports 做出贡献是回馈项目的极好的方式。无论你想要持续的角色，还是某个适合闲暇时的有趣挑战，我们都很高兴能获得你的帮助！

你可以通过以下几种简单方式为保持 Ports 树的更新和良好运作做出贡献：

- 找到一些很酷或有用的软件并为其[创建 Port](https://docs.freebsd.org/en/books/porters-handbook/)。
- 有大量的 Port 没有维护者。成为维护者并[接管未维护的 Port](https://docs.freebsd.org/en/articles/contributing/#adopt-port)。
- 如果你已经创建或接管了 Port，了解[Port 维护者面临的挑战](https://docs.freebsd.org/en/articles/contributing/#maintain-port)。
- 当你寻找快速挑战时，可以[查找并修复损坏的 Port](https://docs.freebsd.org/en/articles/contributing/#fix-broken)。

### 1.5. 从 Ideas 页面选择一个项目

[FreeBSD 的志愿者项目和想法列表](https://wiki.freebsd.org/IdeasPage)也提供给那些愿意为 FreeBSD 项目贡献的人。该列表会定期更新，包含程序员和非程序员的项目，并为每个项目提供相关信息。

## 2. 如何参与

对系统的参与通常可以归入以下五类之一或多个：

### 2.1. Bug 报告和一般评论

*一般性的* 技术兴趣的想法或建议，应该发送到 [FreeBSD 技术讨论邮件列表](https://lists.freebsd.org/subscription/freebsd-hackers)。同样，有兴趣参与这些讨论的人（并且能忍受 *高* 邮件量！）可以订阅 [FreeBSD 技术讨论邮件列表](https://lists.freebsd.org/subscription/freebsd-hackers)。有关此和其他邮件列表的更多信息，请参见 [FreeBSD 手册](https://docs.freebsd.org/en/books/handbook/#eresources-mail)。

如果你要向 src 仓库提交简单的补丁，请考虑将其作为 [PR](https://github.com/freebsd/freebsd-src/pulls)（Pull Request）提交到项目的 GitHub 镜像。合适的提交应满足以下要求：

- 补丁已准备好或几乎准备好提交。提交者应该能够在少于 10 分钟额外工作的情况下提交此补丁。
- 它通过了所有 GitHub CI 作业。
- 你可以快速响应反馈。
- 补丁修改的文件数不超过约 10 个，且修改行数少于约 200 行。大于这个范围的修改可能是可以接受的，或者你可能会被要求将补丁分为多个更小的 PR。
- 每个逻辑变化都是 PR 中的一个单独的提交。每个提交的日志应遵循 [提交日志指南](https://docs.freebsd.org/en/articles/committers-guide/#commit-log-message)。
- 所有提交应包含你的名字和有效的电子邮件地址，并按照你希望在 FreeBSD 仓库中看到的方式标记为作者。不能使用虚假的 github.com 地址。
- PR 的范围在审查过程中不应改变。如果审查提出的修改会扩展范围，请创建独立的 PR。
- 修复提交应与它所修复的提交压缩合并。你的分支中的每个提交都应该适合 FreeBSD 仓库。
- 提交应包括一个或多个 `Signed-off-by:` 行，附上全名和电子邮件地址，以证明 [开发者来源证书](https://developercertificate.org/)。

更新 PR 时，请使用强制推送进行 rebase，而不是使用合并提交。更复杂的修改可以作为 PR 提交，但如果它们太大、难以管理、长期无进展、需要在社区中进一步讨论或需要大量修订，它们可能会被关闭。请避免创建大而广泛的清理补丁：它们太大且缺乏良好审查所需的重点。投递错误的补丁可能会被重定向到更合适的论坛来解决。

提交到 Ports 仓库的 PR 可能会得到或者不会得到开发者的处理，取决于开发者的意愿。目前，如果你遵循 Ports 提交过程[贡献 Ports](https://docs.freebsd.org/en/articles/contributing/#ports-contributing)，你会获得更好的体验。

文档团队也接受通过 GitHub 提交的 PR，但尚未为其建立任何政策。

如果你发现了 bug 或者提交了特定更改，请使用 [bug 提交表单](https://bugs.freebsd.org/submit/)进行报告。尽量填写每个字段。如果补丁不超过 65KB，可以直接在报告中附带补丁。如果补丁适合应用到源代码树中，请在报告的摘要中加入 `[PATCH]`。提交补丁时，*不要*使用复制粘贴，因为复制粘贴会将制表符转为空格，导致补丁无法使用。当补丁大于 20KB 时，考虑在上传之前将其压缩（例如使用 [gzip(1)](https://man.freebsd.org/cgi/man.cgi?query=gzip&sektion=1&format=html) 或 [bzip2(1)](https://man.freebsd.org/cgi/man.cgi?query=bzip2&sektion=1&format=html)）。

提交报告后，你应该会收到确认信息和追踪号码。请保存此追踪号码，以便更新我们有关问题的详细信息。

有关如何编写良好问题报告的更多信息，请参见 [此文章](https://docs.freebsd.org/en/articles/problem-reports/)。

### 2.2. 文档更改

文档更改由 [FreeBSD 文档项目邮件列表](https://lists.freebsd.org/subscription/freebsd-doc) 监督。请查看 [FreeBSD 文档项目指南](https://docs.freebsd.org/en/books/fdp-primer/) 获取完整的操作说明。使用与其他 bug 报告相同的方法提交文档更改（即使是小的更改也很欢迎！）。

### 2.3. 修改现有源代码

对现有源代码的添加或更改是稍微复杂的过程，这很大程度上取决于你与当前 FreeBSD 开发状态的同步程度。FreeBSD 有专门的持续发布版本，称为"FreeBSD-CURRENT"，该版本通过多种方式提供，方便活跃的开发者使用。请参阅 [FreeBSD 手册](https://docs.freebsd.org/en/books/handbook/#current-stable) 以获取有关如何获取和使用 FreeBSD-CURRENT 的更多信息。

遗憾的是，使用较旧的源代码意味着你的更改可能会过时，或者与 FreeBSD 的当前版本存在较大的差异，难以重新集成。为了减少这种可能性，建议订阅 [FreeBSD 公告邮件列表](https://lists.freebsd.org/subscription/freebsd-announce) 和 [FreeBSD-CURRENT 邮件列表](https://lists.freebsd.org/subscription/freebsd-current)，这些邮件列表讨论的是系统的当前状态。

假设你能够获取相对最新的源代码作为更改的基础，接下来的步骤是使用 [diff(1)](https://man.freebsd.org/cgi/man.cgi?query=diff&sektion=1&format=html) 命令生成差异并提交给 FreeBSD 维护者。

提交补丁时首选使用统一输出格式，由 `diff -u` 生成。

```sh
% diff -u oldfile newfile
```

或者

```sh
% diff -u -r -N olddir newdir
```

这将生成给定源文件或目录层次结构的统一差异。

查看 [diff(1)](https://man.freebsd.org/cgi/man.cgi?query=diff&sektion=1&format=html) 获取更多信息。

一旦你有了补丁集（你可以使用 [patch(1)](https://man.freebsd.org/cgi/man.cgi?query=patch&sektion=1&format=html) 命令进行测试），你可以作为 Bugzilla 问题报告提交它们以供 FreeBSD 纳入。*不要*直接将 diff 发送到 [FreeBSD 技术讨论邮件列表](https://lists.freebsd.org/subscription/freebsd-hackers)，否则它们会丢失！我们非常感谢你的贡献（这是志愿者项目！）；由于我们很忙，可能无法立即处理，但它将保留在 PR 数据库中，直到我们处理。通过在报告的摘要中加入 `[PATCH]` 来标示你的提交。

如果你觉得合适（例如，你添加、删除或重命名了文件），可以将你的更改打包成 `tar` 文件。

如果你的更改涉及敏感问题（例如你不确定其进一步分发的版权问题），请直接发送到 [core@FreeBSD.org](mailto:core@FreeBSD.org) 而不是作为 bug 报告提交。[core@FreeBSD.org](mailto:core@FreeBSD.org) 会送达较小的群体，他们负责处理 FreeBSD 大部分日常工作。请注意，这个团队也*非常忙*，所以只有在确实需要时才应发送邮件给他们。

有关编码风格的一些信息，请参考 [intro(9)](https://man.freebsd.org/cgi/man.cgi?query=intro&sektion=9&format=html) 和 [style(9)](https://man.freebsd.org/cgi/man.cgi?query=style&sektion=9&format=html)。我们希望你在提交代码之前至少能了解这些信息。

### 2.4. 新代码或重要的增值包

在进行大量工作或向 FreeBSD 添加重要新功能的情况下，几乎总是需要将更改作为 `tar` 文件提交，或上传到 Web 或 FTP 站点供其他人访问。如果你没有 Web 或 FTP 站点的访问权限，可以在合适的 FreeBSD 邮件列表上请求某人来为你托管这些更改。

当涉及大量代码时，版权问题也总是敏感话题。FreeBSD 偏好使用自由软件许可证，例如 BSD 或 ISC。像 GPLv2 这样的 Copyleft 许可证有时也允许。可以在 [FreeBSD 许可政策](https://www.freebsd.org/internal/software-license/) 文章中找到完整的许可列表。

### 2.5. 捐赠资金和硬件

我们总是非常乐意接受捐赠，以推动 FreeBSD 项目的事业，在这样的志愿者项目中，少量的捐赠可以起到很大的作用！硬件捐赠对于扩展我们支持的外设列表也非常重要，因为我们通常没有足够的资金购买这些设备。

#### 2.5.1. 捐赠资金

[FreeBSD 基金会](https://www.freebsdfoundation.org/) 是一家非营利的、免税的基金会，旨在促进 FreeBSD 项目的目标。作为一家 501(c)3 实体，基金会通常免于美国联邦所得税和科罗拉多州的所得税。捐赠给免税实体的款项通常可以从应纳税联邦收入中扣除。

捐赠可以通过支票方式发送到：

```sh
The FreeBSD Foundation
3980 Broadway Street
STE #103-107
Boulder CO 80304
USA
```

FreeBSD 基金会也接受 [在线捐赠](https://www.freebsdfoundation.org/donate/)，通过多种支付方式。

有关 FreeBSD 基金会的更多信息，请参阅 [FreeBSD 基金会——简介](https://people.freebsd.org/~jdp/foundation/announcement.html)。如需通过电子邮件联系基金会，请写信至 [info@FreeBSDFoundation.org](mailto:info@FreeBSDFoundation.org)。

#### 2.5.2. 捐赠硬件

FreeBSD 项目非常乐意接受可以充分利用的硬件捐赠。如果你有兴趣捐赠硬件，请联系 [捐赠联络办公室](https://www.freebsd.org/donations/)。

## 3. 参与 Ports

### 3.1. 接管未维护的 Ports

#### 3.1.1. 选择未维护的 Port

接管未维护的 Port 是很好的参与方式。未维护的 Port 只有在有人自愿为其工作时才会更新和修复。目前有大量未维护的 Port。建议从接管一个你常用的 Port 开始。

未维护的 Port 的 `MAINTAINER` 会被设置为 `ports@FreeBSD.org`。许多未维护的 Port 可能有待更新，可以通过 [FreeBSD Ports distfile 扫描器](https://portscout.freebsd.org/ports@freebsd.org.html) 查看。

在 [PortsFallout](https://portsfallout.com/fallout?port=&maintainer=ports%40FreeBSD.org) 上，可以看到带有错误的未维护 Port 的列表。

一些 Port 由于其依赖关系和二级 Port 关系，会影响大量其他 Port。通常，我们希望在维护此类 Port 之前，维护者能有一定的经验。

你可以通过查看名为 **INDEX** 的主 Port 索引，来了解某 Port 是否有依赖关系或二级 Port。（该文件的名称根据 FreeBSD 的版本不同而有所不同；例如，**INDEX-13**）。一些 Port 有条件依赖关系，这些依赖关系不会包含在默认的 **INDEX** 构建中。我们希望你通过查看其他 Port 的 **Makefile** 来识别这些 Port。

#### 3.1.2. 如何接管 Port

首先，请确保你了解 [Port 维护者的挑战](https://docs.freebsd.org/en/articles/contributing/#maintain-port)。同时阅读 [Porter’s Handbook](https://docs.freebsd.org/en/books/porters-handbook/)。*请不要承诺你无法轻松处理的任务。*

你可以随时请求接管任何未维护的 Port。只需将 `MAINTAINER` 设置为你的电子邮件地址，并发送包含更改的 PR（问题报告）。如果该 Port 有构建错误或需要更新，你可能希望在同一个 PR 中包含其他更改。这将有所帮助，因为许多提交者不太愿意将维护权交给没有 FreeBSD 经验的人员。提交修复构建错误或更新 Port 的 PR 是建立经验记录的最佳途径。

将你的 PR 提交到产品 `Ports & Packages`。提交者将审查你的 PR，提交更改，并最终关闭 PR。有时这个过程可能需要一些时间（提交者也是志愿者 :)）。

### 3.2. Port 维护者的挑战

本节将帮助你了解为什么 Port 需要维护，并概述 Port 维护者的责任。

#### 3.2.1. 为什么 Port 需要维护

创建 Port 是一次性的任务。确保 Port 保持最新、能继续构建和运行则需要持续的维护工作。维护者是那些愿意花费部分时间来实现这些目标的人。

Port 需要维护的最主要原因是将最新、最好的第三方软件带给 FreeBSD 社区。另一个挑战是随着 Ports 集合框架的发展，保持各个 Port 的正常运行。

作为维护者，你需要管理以下挑战：

- **新软件版本和更新。** 新版本和现有 Port 软件的更新不断发布，这些需要被纳入 Ports 集合，以提供最新的软件。
- **依赖关系的变化。** 如果你的 Port 的依赖关系发生了重要变化，可能需要更新 Port，以确保它能够继续正常工作。
- **影响依赖 Port 的变化。** 如果其他 Port 依赖于你维护的 Port，你对该 Port 的更改可能需要与其他维护者进行协调。
- **与其他用户、维护者和开发者的互动。** 作为维护者，承担支持角色是工作的一部分。不期望你提供一般支持（但如果你愿意，我们欢迎）。你应该提供的是 FreeBSD 特有问题的协调点。
- **排查 Bug。** Port 可能会受到 FreeBSD 特有 Bug 的影响。当这些 Bug 被报告时，你需要进行调查、找到并修复它们。在它们进入 Ports 集合之前，彻底测试 Port 以找出潜在问题，当然更好。
- **Port 基础设施和政策的变化。** 有时，构建 Port 和包的系统会进行更新，或者可能会提出新的基础设施建议。你应该关注这些变化，以防你的 Port 受影响，并需要更新。
- **基本系统的变化。** FreeBSD 正在不断开发中。软件、库、内核甚至政策的变化都可能导致 Port 需要相应调整。

#### 3.2.2. 维护者的责任

##### 3.2.2.1. 保持 Port 的更新

本节概述了保持 Port 更新的流程。

这是一个概述。有关升级 Port 的更多信息，可以参见 [Porter’s Handbook](https://docs.freebsd.org/en/books/porters-handbook/)。

1. 关注更新
   监控上游供应商的新版本、更新和安全修复。公告邮件列表或新闻网页在这方面很有帮助。有时用户会联系你，询问你的 Port 什么时候更新。如果你忙于其他事情或因任何原因无法及时更新，问问他们是否愿意通过提交更新来帮忙。

   你也可能会收到来自 `FreeBSD Ports Version Check` 的自动电子邮件，通知你 Port 的 distfile 有更新版本。更多有关该系统的信息（包括如何停止未来的邮件）将在消息中提供。

2. 合并更改
   当更新发布时，将这些更改合并到 Port 中。你需要能够生成原始 Port 与更新 Port 之间的补丁。

3. 审查和测试
   彻底审查和测试你的更改：

   - 在尽可能多的平台和架构上构建、安装并测试你的 Port。Port 在一个分支或平台上可能正常工作，但在另一个平台上则会失败。
   - 确保你的 Port 依赖关系完整。推荐的做法是安装自己的 Port tinderbox。有关更多信息，请参考 [Port 维护者和贡献者资源](https://docs.freebsd.org/en/articles/contributing/#resources)。
   - 检查打包列表是否是最新的。包括添加任何新文件和目录，删除未使用的条目。
   - 使用 [portlint(1)](https://man.freebsd.org/cgi/man.cgi?query=portlint&sektion=1&format=html) 作为指南，验证你的 Port。有关使用 portlint 的重要信息，请参见 [Port 维护者和贡献者资源](https://docs.freebsd.org/en/articles/contributing/#resources)。
   - 考虑你的更改是否会导致其他 Port 发生故障。如果是这样，与那些 Port 的维护者协调这些更改。特别是，如果你的更新更改了共享库版本，至少需要为依赖的 Port 增加 `PORTREVISION`，以便自动工具（如 [ports-mgmt/poudriere](https://cgit.freebsd.org/ports/tree/ports-mgmt/poudriere/)）能够自动升级这些 Port。

4. 提交更改
   通过提交 PR 来提交你的更新，解释更改并提供原始 Port 与更新 Port 之间的差异补丁。有关如何写好 PR 的信息，请参见 [编写 FreeBSD 问题报告](https://docs.freebsd.org/en/articles/problem-reports/)。

   >**注意**
   >
   >请不要提交完整 Port 的 [shar(1)](https://man.freebsd.org/cgi/man.cgi?query=shar&sektion=1&format=html) 压缩包；而应使用 [git-format-patch(1)](https://man.freebsd.org/cgi/man.cgi?query=git-format-patch&sektion=1&format=html) 或 [diff(1)](https://man.freebsd.org/cgi/man.cgi?query=diff&sektion=1&format=html) `-ruN`。这样，提交者可以更容易地查看所做的更改。有关更多信息，请参见 Port 开发者手册中的 [升级](https://docs.freebsd.org/en/books/porters-handbook/#port-upgrading) 部分。

5. 等待
   在某个时候，提交者会处理你的 PR。可能需要几分钟，也可能需要一两周，所以请耐心等待。如果超过这个时间，请在邮件列表（例如 [FreeBSD ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports)）、IRC：#bsdports（EFNet）或 #freebsd-ports（Libera）上寻求帮助。

6. 提供反馈
   如果提交者发现你更改中的问题，他们很可能会将其反馈给你。迅速响应可以帮助更快地提交你的 PR，并在解决问题时保持良好的沟通。

7. 最终
   你的更改将被提交，Port 将得到更新。PR 会由提交者关闭。就这样！

##### 3.2.2.2. 确保你的 Port 继续正确构建

本节讲述了如何发现和修复导致 Port 无法正确构建的问题。

FreeBSD 仅保证 Ports 在 `-STABLE` 分支上正常工作。理论上，你应该能够使用每个稳定分支的最新版本（因为 ABI 不应该变化），但如果你能运行该分支，那就更好了。

由于大多数 FreeBSD 安装运行在 PC 兼容机器上（即 `i386` 架构），我们期望你保持该架构上的 Port 工作。我们希望 Port 也能在 `amd64` 架构上原生运行。如果你没有这些机器，完全可以请求帮助。

>**注意**
>
>非 `x86` 机器的常见失败模式是原始程序员假设了例如指针是 `int` 类型，或者使用了较宽松的旧版 gcc 编译器。越来越多的应用程序作者正在重构他们的代码以消除这些假设，但如果作者没有积极维护代码，你可能需要自己处理这些问题。

以下是确保你的 Port 能够成功构建所需执行的任务：

1. **关注构建失败**
   检查你的邮件，查看来自 `pkg-fallout@FreeBSD.org` 的邮件和 [distfiles 扫描器](http://portscout.freebsd.org/)，以查看无法构建的 Port 是否过时。

2. **收集信息**
   一旦意识到存在问题，收集有助于解决问题的信息。由 `pkg-fallout` 报告的构建错误会附带日志，你可以通过这些日志查看构建失败的位置。如果错误是由用户报告的，请要求他们提供可能帮助诊断问题的信息，例如：

   - 构建日志
   - 用于构建 Port 的命令和选项（包括在 **/etc/make.conf** 中设置的选项）
   - 通过 [pkg-info(8)](https://man.freebsd.org/cgi/man.cgi?query=pkg-info&sektion=8&format=html) 显示的已安装软件包列表
   - 通过 [uname(1)](https://man.freebsd.org/cgi/man.cgi?query=uname&sektion=1&format=html) `-a` 显示的 FreeBSD 版本
   - 他们的 Port 集合上次更新时间
   - 他们的 Port 树和 **INDEX** 上次更新时间

3. **调查并找到解决方案**
   不幸的是，这没有简单的过程可以遵循。不过请记住：如果你遇到困难，向他人寻求帮助！[FreeBSD ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports) 是个很好的起点，且上游开发者通常非常乐于提供帮助。

4. **提交更改**
   与更新 Port 一样，你现在应该整合更改、进行审查和测试，提交你的更改到 PR 中，并在需要时提供反馈。

5. **将补丁发送给上游作者**
   在某些情况下，你需要对 Port 做补丁，以使其在 FreeBSD 上运行。有些（但不是所有）上游作者会接受这些补丁并将其合并到下一版本中。如果是这样，这甚至可能帮助他们在其他 BSD 系统上的用户，避免重复劳动。请考虑将适用的补丁作为礼貌发送给作者。

##### 3.2.2.3. 调查与 Port 相关的 bug 报告和 PR

本节讲述了如何发现并修复 bug。

FreeBSD 特有的 bug 通常是由于构建和运行环境的假设不适用于 FreeBSD 所引起的。你不太可能遇到这种问题，但它可能更为微妙且难以诊断。

确保你的 Port 继续按预期工作需要执行的任务如下：

1. **回应 bug 报告**
   Bug 可能通过电子邮件报告给你，来自 [Problem Report 数据库](https://bugs.freebsd.org/search/)，也可能直接由用户报告给你。

   你应在 14 天内回应 PR 和其他报告，但请尽量不要拖延这么长时间。尽早回应，即使只是告知你需要更多时间才能处理 PR。

   如果你 14 天后仍未回应，所有提交者都可以通过 `maintainer-timeout` 从你未回应的 PR 中提交更改。

2. **收集信息**
   如果报告 bug 的人没有提供修复方案，你需要收集足够的信息，以便生成修复方案。

   如果 bug 是可重现的，你可以自己收集大部分所需的信息。如果无法重现，要求报告 bug 的人提供信息，例如：

   - 他们的操作步骤、预期的程序行为和实际行为的详细描述
   - 用于触发 bug 的输入数据副本
   - 他们的构建和执行环境信息—例如，已安装软件包的列表和 [env(1)](https://man.freebsd.org/cgi/man.cgi?query=env&sektion=1&format=html) 的输出
   - 核心转储
   - 堆栈跟踪

3. **消除错误报告**
   一些 bug 报告可能是错误的。例如，用户可能仅仅是错误地使用了程序；或者他们安装的软件包可能过时，需要更新。有时，报告的 bug 并非 FreeBSD 特有。在这种情况下，将 bug 报告给上游开发者。如果问题是你可以修复的，你还可以给 Port 打补丁，以便在下一次上游发布前应用修复。

4. **找到解决方案**
   与构建错误一样，你需要找出问题的解决方案。同样，如果遇到困难，记得寻求帮助！

5. **提交或批准更改**
   与更新 Port 一样，你现在应该整合更改、进行审查和测试，并将你的更改提交到 PR 中（如果该问题已存在 PR，你可以发送后续消息）。如果其他用户已在 PR 中提交了更改，你也可以发送后续消息，告知你是否批准这些更改。

##### 3.2.2.4. 提供支持

作为维护者，提供支持是工作的一部分——不是针对软件本身的一般支持——而是针对 Port 和任何 FreeBSD 特有的怪癖和问题。用户可能会就疑问、建议、故障和补丁联系你。大多数时候，他们的通信将与 FreeBSD 相关。

偶尔，你可能需要展现你的外交技巧，友好地将寻求一般支持的用户指引到合适的资源。较少情况下，你可能会遇到某人询问为什么 `RPMS` 没有更新，或者如何在 Foo Linux 上运行该软件。趁机告诉他们，你的 Port 是最新的（如果真是这样的话！），并建议他们尝试使用 FreeBSD。

有时，用户和开发者会认为你是忙碌的人，你的时间很宝贵，并且会为你做一些工作。例如，他们可能会：

- 提交 Bugzilla PR 或向你发送更新 Port 的补丁，
- 调查并提供对 PR 的修复，或者
- 以其他方式提交对你 Port 的更改。

在这种情况下，你的主要责任是及时回应。再次提醒，非响应维护者的超时期限是 14 天。超过这个期限，未批准的更改可能会被提交。他们为你做了这些工作，因此请尽量及时回应。然后尽快审查、批准、修改或与他们讨论更改。

如果你能够让他们感到他们的贡献受到重视（而且确实应该受到重视），那么你将更有可能说服他们未来为你做更多的事情 :-)。

### 3.3. 寻找并修复损坏的 Port

有一些非常好的地方可以找到需要关注的 Port。

你可以使用问题报告数据库的 [网页接口](https://bugs.freebsd.org/search) 搜索和查看未解决的 Bugzilla PR。大多数 Ports 的 PR 都是更新，但通过稍微搜索和浏览摘要，你应该能找到一些有趣的工作内容。

[PortsFallout](https://portsfallout.com/) 显示了从 FreeBSD 包构建中收集的 Port 问题。

对于已维护的 Port，发送更改也是可以的，但请记得在操作之前询问维护者，看看他们是否正在处理该问题。

一旦你找到了一个 bug 或问题，收集信息、调查并修复它！如果已有 PR，请跟进该 PR。如果没有，请创建新的 PR。你的更改将会被审查，并且如果一切检查无误，将被提交。

### 3.4. 何时退出

随着你的兴趣和承诺的变化，你可能会发现自己没有时间继续进行某些（或所有）Ports 的贡献。这是可以理解的！如果你不再使用某个 Port 或由于时间或兴趣的变化失去了作为维护者的意愿，请告知我们。这样，我们可以继续让其他人尝试解决现有的 Port 问题，而不必等你的回应。请记住，FreeBSD 是志愿者项目，所以如果维护 Port 不再有趣了，可能是时候让别人来接手了！

无论如何，Ports 管理团队（`portmgr`）保留在你长期未积极维护 Port 的情况下重置你的维护权的权利。（目前，这个时间为 3 个月。）我们所说的是，存在未解决的问题或待处理的更新，并且在这段时间内未处理。

### 3.5. Port 维护者和贡献者的资源

[Porter’s Handbook](https://docs.freebsd.org/en/books/porters-handbook/) 是你了解 Port 系统的搭车客指南。请随时查阅！

[编写 FreeBSD 问题报告](https://docs.freebsd.org/en/articles/problem-reports/) 讲述了如何最佳地制定和提交问题报告。遵循这篇文章将大大帮助我们减少处理你的 PR 所需的时间。

[问题报告数据库](https://bugs.freebsd.org/bugzilla/query.cgi)。

[FreeBSD Ports distfile 扫描器（portscout）](http://portscout.freebsd.org/) 可以显示无法获取 distfiles 的 Port。你可以检查你自己的 Port，或者使用它查找需要更新 `MASTER_SITES` 的 Port。

[ports-mgmt/poudriere](https://cgit.freebsd.org/ports/tree/ports-mgmt/poudriere/) 是测试 Port 的最全面方法，涵盖了从安装、打包到卸载的整个周期。文档位于 [poudriere GitHub 仓库](https://github.com/freebsd/poudriere)。

[portlint(1)](https://man.freebsd.org/cgi/man.cgi?query=portlint&sektion=1&format=html) 是可以验证你的 Port 是否符合许多重要风格和功能准则的应用程序。portlint 是简单的启发式应用程序，因此你应该 **仅将其作为指南**。如果 portlint 提示的更改似乎不合理，请查阅 [Porter's Handbook](https://docs.freebsd.org/en/books/porters-handbook/) 或寻求建议。

[FreeBSD Ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports) 是讨论与 Ports 相关的常规话题的地方。这是寻求帮助的好地方。你可以 [订阅，或阅读和搜索列表存档](https://lists.freebsd.org/)。阅读 [FreeBSD Ports bug 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports-bugs) 和 [SVN 提交消息（head/ 分支的 Port 树）](https://lists.freebsd.org/pipermail/svn-ports-head) 的存档也可能会很有帮助。

[PortsFallout](https://portsfallout.com/) 是帮助查找 [FreeBSD 包 Fallout 存档](https://lists.freebsd.org/archives/freebsd-pkg-fallout/) 的地方。

## 4. 在其他领域入门

你是否正在寻找本文未提及的有趣入门内容？FreeBSD 项目有多个 Wiki 页面，包含新贡献者可以从中获得入门灵感的领域。

[Junior Jobs](https://wiki.freebsd.org/JuniorJobs) 页面列出了可能对刚接触 FreeBSD 并希望从事有趣工作以试水的人感兴趣的项目。

[Ideas Page](https://wiki.freebsd.org/IdeasPage) 包含了项目中各种“有则更好”或“有趣的”工作内容。
