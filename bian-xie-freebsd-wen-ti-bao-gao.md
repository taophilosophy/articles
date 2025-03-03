# 编写 FreeBSD 问题报告

<details open="" data-immersive-translate-walked="71abfe93-e674-4bca-af82-b98b7c8b7681"><summary data-immersive-translate-walked="71abfe93-e674-4bca-af82-b98b7c8b7681" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

IBM、AIX、OS/2、PowerPC、PS/2、S/390 和 ThinkPad 是国际商业机器公司在美国、其他国家或两者都是的商标。

Intel、Celeron、Centrino、Core、EtherExpress、i386、i486、Itanium、Pentium 和 Xeon 是 Intel Corporation 或其子公司在美国和其他国家的商标或注册商标。

Sun、Sun Microsystems、Java、Java Virtual Machine、JDK、JRE、JSP、JVM、Netra、OpenJDK、Solaris、StarOffice、SunOS 和 VirtualBox 是 Sun Microsystems, Inc. 在美国和其他国家的商标或注册商标。

制造商和销售商用于区分其产品的许多名称被视为商标。在本文档中出现这些名称时，FreeBSD 项目已经意识到商标声明，这些名称后面跟着“™”或“®”符号。

</details>

摘要

本文介绍了如何最好地制定并提交问题报告给 FreeBSD 项目。

---

## 1. 介绍

作为软件用户，最令人沮丧的经历之一是提交问题报告，却只收到一句简短且无帮助的解释，比如“不是 bug”或“虚假 PR”。同样，作为软件开发人员，最令人沮丧的经历之一是被淹没在问题报告中，这些报告实际上并不是问题报告，而是寻求支持的请求，或者几乎没有提供关于问题及如何重现问题的信息。

本文试图描述如何撰写良好的问题报告。一个好的问题报告是什么？嗯，直截了当地说，一个好的问题报告是能够被迅速分析和处理，以使用户和开发人员双方都满意的报告。

尽管本文的主要重点是 FreeBSD 的问题报告，但其中的大部分内容应该也适用于其他软件项目。

请注意，本文按主题组织，而非按时间顺序。在提交问题报告之前，请阅读整个文档，而不是把它当作一篇逐步教程。

## 2. 何时提交问题报告

问题有许多类型，并非所有问题都应该引发问题报告。当然，没有人是完美的，会有时候一个程序中看似是 bug 的问题，实际上可能是对命令语法的误解或配置文件中的拼写错误（尽管这本身有时可能表明应用程序文档质量不佳或错误处理不当）。仍然有许多情况下，提交问题报告显然不是正确的做法，只会让提交者和开发人员都感到沮丧。相反，有些情况下，可能适合提交关于其他内容而非 bug 的问题报告，比如一个改进或新功能。

那么，怎么确定哪些是错误，哪些不是？作为一个简单的经验法则，如果问题可以表达为一个问题（通常是“我该如何做 X？”或“我在哪里找到 Y？”的形式），那么这个问题就不是一个错误。情况并不总是那么非黑即白，但问题规则涵盖了大多数情况。在寻找答案时，请考虑向 FreeBSD 一般问题邮件列表提问。

在提交有关 ports 或其他不属于 FreeBSD 本身的软件的 PR 时，请考虑以下因素：

- 请不要提交简单声明应用程序的新版本可用的问题报告。当应用程序的新版本可用时，端口监控会自动通知 Ports 维护人员。欢迎更新 port 至最新版本的实际补丁。
- 对于未维护的 ports ( MAINTAINER 是 ports@FreeBSD.org )，没有包含补丁的 PR 不太可能被提交者采纳。要成为未维护的 port 的维护者，请提交包含请求的 PR（最好有补丁但不是必需的）。
- 在任何情况下，遵循 Porter’s Handbook 中描述的流程将产生最佳结果。（你可能还希望阅读有关贡献给 FreeBSD Ports 的内容。）

几乎无法修复无法重现的错误。如果错误仅发生一次，你无法重现它，并且似乎其他人也没有遇到这个问题，那么开发人员都无法重现它或找出问题所在的机会很小。这并不意味着这种情况从未发生，但这意味着你的问题报告导致错误修复的可能性很小。更糟糕的是，通常这些种类的错误实际上是由于硬盘故障或过热处理器引起的—你应该始终在可能的情况下尝试排除这些原因，然后再提交 PR。

接下来，要决定向谁提交问题报告，你需要了解组成 FreeBSD 的软件由几个不同的元素组成：

- 基本系统中的代码是由 FreeBSD 贡献者编写和维护的，例如内核、C 库和设备驱动程序（分类为 kern ）；二进制实用程序（ bin ）；手册页和文档（ docs ）；以及网页（ www ）。这些领域的所有错误都应报告给 FreeBSD 开发人员。
- 基本系统中的代码是由其他人编写和维护的，并导入 FreeBSD 并进行调整。示例包括 clang(1)和 sendmail(8)。这些领域的大多数错误应报告给 FreeBSD 开发人员；但在某些情况下，如果问题不是特定于 FreeBSD，则可能需要向原始作者报告。
- 非在基本系统中而是属于 FreeBSD Ports （类别 ports ）的个别应用程序。这些应用程序大多不是由 FreeBSD 开发人员编写的；FreeBSD 提供的仅仅是安装应用程序的框架。因此，只有在认为问题与 FreeBSD 特定相关时才向 FreeBSD 开发人员报告问题；否则，请向软件作者报告问题。

然后，确定问题是否及时。很少有什么比收到关于她已经修复的错误的问题报告更令开发人员恼火的事情。

如果问题出现在基本系统中，请先阅读有关 FreeBSD 版本的 FAQ 部分，如果你对该主题尚不熟悉。FreeBSD 无法修复任何不属于基本系统某些最新分支的问题，因此，关于旧版本的软件提出错误报告可能只会导致开发人员建议你升级到受支持的版本以查看问题是否仍会重现。安全官员团队维护受支持版本的列表。

如果问题出在 port 中，请考虑向上游提交 bug。FreeBSD 项目不能修复所有软件中的所有 bug。

## 3. 准备工作

一个好的规则是在提交问题报告之前总是先进行背景搜索。也许问题已经被报告；也许它正在邮件列表中讨论，或者最近讨论过；它甚至可能已经在比你运行的版本更新的版本中被修复。因此，你应该在提交问题报告之前检查所有明显的地方。对于 FreeBSD，这意味着：

- FreeBSD 常见问题解答（FAQ）列表。FAQ 试图回答各种问题，例如有关硬件兼容性、用户应用程序和内核配置的问题。
- 邮件列表。如果你尚未订阅，请使用 FreeBSD 网站上的可搜索档案。如果列表中尚未讨论该问题，你可以尝试发布有关该问题的消息，并等待几天，看看是否有人能发现被忽视的内容。
- 可选地，整个网络 - 使用你喜欢的搜索引擎查找有关该问题的任何参考资料。你甚至可能会从存档的邮件列表或你不知道或没有考虑搜索的新闻组中获得结果。
- 其次，可搜索的 FreeBSD PR 数据库（Bugzilla）。除非问题是最近才出现或者比较晦涩，很有可能已经有人报告过了。
- 最重要的是，尝试查看源代码中现有的文档是否解决了你的问题。对于基本的 FreeBSD 代码，你应该仔细研究系统上 /usr/src/UPDATING 的内容或者在 https://cgit.freebsd.org/src/tree/UPDATING 上的最新版本。（如果你正在从一个版本升级到另一个版本，特别是如果你要升级到 FreeBSD-CURRENT 分支，这是非常重要的信息）。
  但是，如果问题出现在作为 FreeBSD Ports 的一部分安装的某个东西上，你应该参考 /usr/ports/UPDATING（用于单个 ports）或 /usr/ports/CHANGES（用于影响整个 Ports 的更改）。https://cgit.freebsd.org/ports/tree/UPDATING 和 https://cgit.freebsd.org/ports/tree/CHANGES 也可以通过 cgit 访问。

## 4. 编写问题报告

现在你已经确定你的问题值得一份问题报告，并且这是一个 FreeBSD 问题，是时候写实际的问题报告了。在我们深入讨论用于生成和提交 PR 的程序机制之前，这里有一些提示和技巧，可确保你的 PR 最为有效。

## 5. 编写良好问题报告的技巧和诀窍

- 不要留空“摘要”行。PR 同时会发送到一个传遍全球的邮件列表（其中“摘要”用于第 Subject: 行），也会存入数据库。任何稍后浏览数据库的人，发现一个主题空白的 PR，往往会直接跳过它。请记住，PR 会一直保存在数据库中，直到有人关闭它；一个匿名的 PR 通常会在噪音中消失。
- 避免使用含糊的“摘要”行。不要假设任何阅读你的 PR 的人对你的提交有任何背景，提供的信息越多越好。例如，问题适用于系统的哪个部分？你只在安装时看到问题，还是在运行时也有？举例来说，与 Summary: portupgrade is broken 相比，看看这个更具信息量的示例： `Summary: port ports-mgmt/portupgrade coredumps on -current` 。（对于 ports，在“摘要”行中同时提供类别和端口名称尤其有帮助。）
- 如果你有补丁，请说明。有补丁的存在会大大加快报告的进展。

  - 不要使用 `{{ 0 }}` 或 `{{ 1 }}` 关键词 - 它们已不建议使用。

- 如果你是一个维护者，请表明。如果你正在维护源代码的一部分（例如，现有的 {{ 1001 }} ），你绝对应该将你的 PR 的 "类" 设置为 {{ 0 }} 。这样处理你的 PR 的任何提交者就不必检查了。
- 要具体。你提供有关你遇到的问题的信息越多，获得回应的机会就越大。

  - 包括你正在运行的 FreeBSD 版本（有一个地方可以放置，见下文）以及所在的架构。你应该说明你是从发布版运行（例如，从 CD-ROM 或下载），还是从 Git 维护的系统运行（如果是这样，请说明你所在的哈希和分支）。如果你正在跟踪 FreeBSD-CURRENT 分支，这将是某人会问的第一件事，因为修复（尤其是针对备受关注的问题）往往会很快提交，而且预期 FreeBSD-CURRENT 用户会跟进。
  - 包括你在 make.conf、src.conf 和 src-env.conf 中指定的全局选项。鉴于选项的数量是无限的，不是每种组合都可能得到充分支持。
  - 如果问题很容易重现，请包含能帮助开发人员自行重现问题的信息。如果问题可以通过特定输入演示，则尽可能包含该输入的示例，并包含实际输出和预期输出。如果这些数据很大或无法公开，那么请尝试创建一个展示相同问题的最小文件，并将其包含在 PR 中。
  - 如果这是一个内核问题，请准备好提供以下信息。（你无需默认包含这些信息，因为这只会填满数据库，但你应该包含你认为可能相关的摘录）：

    - 你的内核配置（包括你安装了哪些硬件设备）
    - 你是否启用了调试选项（例如 WITNESS ），如果是的话，当你更改该选项的意义时问题是否仍然存在
    - 任何回溯、恐慌或其他控制台输出的完整文本，或者如果生成了任何条目在 /var/log/messages 中
    - 如果你的问题与特定硬件部件有关， pciconf -l 的输出和相关部分的输出
    - 你已阅读 src/UPDATING，并且你的问题在那里未列出（肯定会有人问）
    - 无论你是否可以作为备选运行任何其他内核（这是为了排除类似硬件故障的问题，如磁盘故障和过热的 CPU，这可能伪装成内核问题）

  - 如果这是一个 ports 问题，那么请准备提供以下信息。（你不必默认包含这些信息，这只会填满数据库，但你应该包含你认为可能相关的摘录）

    - 你已安装哪个 ports
    - 覆盖 bsd.port.mk 中默认值的任何环境变量，例如 PORTSDIR
    - 你已阅读 ports/UPDATING，并且你的问题未在其中列出（肯定会有人问）

- 避免对功能的模糊请求。形式为“某人应该真的实现某种功能”的 PR 不太可能产生结果，比非常具体的请求更少。请记住，源代码对每个人都是可用的，因此如果你想要一个功能，确保其被包含的最佳方法是着手工作！还要考虑到许多类似此类的事物更适合在 freebsd-questions 上进行讨论，而不是在 PR 数据库中进行条目，如上所述。
- 确保没有其他人已经提交了类似的 PR。尽管这已经在上面提到过，但在这里重复一遍。只需一两分钟使用 https://bugs.freebsd.org/bugzilla/query.cgi 上的基于 Web 的搜索引擎进行搜索即可。（当然，每个人都会偶尔忘记这样做。）
- 每个问题报告只报告一个问题。除非它们相关，否则避免在同一报告中包含两个或更多问题。在提交补丁时，避免在同一个 PR 中添加多个功能或修复多个错误，除非它们密切相关-这样的 PR 通常需要更长时间来解决。
- 避免有争议的请求。如果你的 PR 涉及过去有争议的领域，你可能不仅要提供补丁，还要提供为什么这些补丁是“正确的事情”的理由。如上所述，仔细搜索邮件列表的存档，使用 https://www.FreeBSD.org/search/#mailinglists 总是一个很好的准备工作。
- 有礼貌。几乎任何有可能参与你的 PR 的人都是志愿者。当某人已经出于某种非金钱动机而做事时，没有人喜欢被告知他们必须做某事。这是在开源项目上随时牢记的好事。

## 6. 在开始之前

使用基于 web 的 PR 提交表单时也适用类似的考虑。要小心粘贴操作可能会改变空格或其他文本格式的情况。

最后，如果提交内容很长，请离线准备工作，这样如果提交时出现问题，不会丢失任何内容。

## 7. 附加补丁或文件

一般而言，我们建议使用 git format-patch 生成一组或一系列相对于基本分支的统一差异（例如 origin/main ）。以这种方式生成的补丁将包含 Git 哈希，并且将包含你的姓名和电子邮件地址，这样更容易让提交者应用你的补丁并正确地将你认定为作者（使用 git am ）。对于你不希望使用 git 进行的小修改，请务必使用 diff(1) 和选项 -u 创建统一差异，因为这样能为开发人员提供更多上下文，并比其他差异格式更易读。

对于内核或基本实用程序的问题，首选针对 FreeBSD-CURRENT（主 Git 分支）的补丁，因为所有新代码都应该首先在那里应用并测试。在做了适当或实质性的测试后，代码将被合并/迁移到 FreeBSD-STABLE 分支。

如果你内联附加补丁，而不是作为附件，请注意目前绝大多数问题是一些电子邮件程序倾向于将制表符渲染为空格，这将完全破坏任何打算作为 Makefile 的部分的内容。

不要将补丁作为附件发送使用 Content-Transfer-Encoding: quoted-printable 。这样将执行字符转义，整个补丁将无效。

请注意，虽然在 PR 中包含小补丁通常没问题，特别是当它们修复了 PR 中描述的问题时——尤其是大补丁和可能需要在提交之前进行大量审查的新代码应放在 web 或 ftp 服务器上，并在 PR 中包含 URL，而不是补丁。电子邮件中的补丁往往会被搞乱，而补丁越大，对于感兴趣的人来说解决它的难度就越大。此外，在网上发布补丁可以让你修改它，而无需重新提交整个补丁作为对原始 PR 的跟进。最后，大补丁只会增加数据库的大小，因为关闭的 PR 实际上并未被删除，而是被保留并标记为完成。

你还应该注意，除非你在 PR 中明确指定或在补丁本身中另有规定，否则你提交的任何补丁都将被视为在修改的原始文件相同条款下许可。

## 8. 填写表格

>你使用的电子邮件地址将成为公开信息，并可能被垃圾邮件发送者获取。你应该制定垃圾邮件处理程序，或使用临时电子邮件帐户。但是，请注意，如果你根本不使用有效的电子邮件帐户，我们将无法向你询问有关你的 PR 的问题。 

当你提交错误报告时，你将找到以下字段：

- 摘要：请用简短准确的描述填写此字段。摘要将用作问题报告电子邮件的主题，并用于问题报告列表和摘要；具有模糊摘要的问题报告往往会被忽略。
- 严重性：其中之一是 Affects only me ， Affects some people 或 Affects many people 。不要过度反应; 除非确实如此，否则请避免将问题标记为 Affects many people 。由于已经有许多人确实那样做，FreeBSD 开发人员不一定会更快地解决你的问题。
- 类别：选择一个适当的类别。你需要做的第一件事是决定问题所在系统的哪一部分。请记住，FreeBSD 是一个完整的操作系统，它安装了内核、标准库、许多外围驱动程序以及大量实用程序（“基础系统”）。但是，Ports 中有成千上万的其他应用程序。首先需要确定问题是在基本系统中还是在通过 Ports 安装的东西中。
  这里是各大类别的描述:

  - 如果问题出现在内核、库（例如标准 C 库 libc ）或基本系统中的外围驱动程序，通常会使用 kern 类别。（有一些例外；请参见下文）。一般来说，这些是手册第 2、3 或 4 节中描述的内容。
  - 如果问题出现在诸如 sh(1) 或 mount(8) 等二进制程序中，你首先需要确定这些程序是在基本系统中还是通过 Ports 添加的。如果不确定，你可以执行 whereis<span> </span><em>programname</em> 。FreeBSD 对于 Ports 的惯例是将所有内容安装在 /usr/local 下，尽管系统管理员可以覆盖此设置。对于这些内容，你将使用 ports 类别（是的，即使 port 的类别是 www ；请参见下文）。如果位置是 /bin、/usr/bin、/sbin 或 /usr/sbin，则它属于基本系统，你应该使用 bin 类别。这些都是手册第 1 或 8 节中描述的内容。
  - 如果你认为错误出现在启动 (rc) 脚本中，或者是某种其他非可执行配置文件中，则正确的类别是 conf （配置）。这些是手册第 5 节中描述的内容。
  - 如果你发现文档集（文章，书籍，手册）或网站中存在问题，正确的选择是 docs 。
  - 如果你遇到来自 port 名称为 www/<em>someportname</em> 的东西的问题，这仍属于 ports 类别。 

    还有一些更专业的类别。

  - 如果问题本应该归类为 kern ，但与 USB 子系统有关，则正确的选择是 usb 。
  - 如果问题本应该归类为 kern ，但与线程库有关，则正确的选择是 threads 。
  - 如果问题本应该属于基本系统，但与我们遵循诸如 POSIX® 之类的标准有关，则正确的选择是 standards 。
  - 如果你确信问题只会发生在你正在使用的处理器架构下，选择其中一个特定架构类别：通常情况下为 Intel 兼容机器的 32 位模式 i386 ； AMD 机器在 64 位模式下 amd64 （这也包括运行在 EMT64 模式下的 Intel 兼容机器）；较少情况下为 arm 或 powerpc 。
  - 这些类别经常被错误地用于“我不知道”问题。与其猜测，还不如直接使用 misc 。

    示例 1。架构特定类别的正确使用

    你拥有一台普通的基于 PC 的机器，并认为自己遇到了特定芯片组或特定主板的问题： i386 是正确的类别。

    示例 2. 错误使用特定体系结构的类别

    你遇到了常见总线上的插卡问题，或者特定类型的硬盘驱动器问题：在这种情况下，它可能适用于多于一个体系结构， kern 是正确的类别。

  - 如果你真的不知道问题出在哪里（或者解释似乎不符合上述情况），请使用 misc 类别。在这样做之前，你可能希望先在 FreeBSD 一般问题邮件列表中寻求帮助。你可能会被建议使用现有的类别会更好。

- 环境：应尽可能准确地描述发现问题的环境。这包括操作系统版本、包含问题的特定程序或文件的版本以及任何其他相关项目，如系统配置、影响问题的其他已安装软件等——简而言之，开发人员需要知道的一切，以重现问题发生的环境。
- 描述：对你遇到的问题进行完整而准确的描述。尽量避免对问题的原因进行猜测，除非你确定自己找到了正确的方向，因为这可能会误导开发人员对问题作出错误的假设。它应该包括重现问题所需的操作。如果你知道任何解决方法，请包括在内。这不仅可以帮助其他有同样问题的人解决问题，还可能帮助开发人员理解问题的原因。

## 9. 跟进

一旦问题报告被提交，你将通过电子邮件收到确认函，其中将包含分配给你的问题报告的跟踪号码和一个网址，你可以使用它来检查其状态。幸运的话，有人会对你的问题感兴趣并试图解决它，或者，视情况而定，解释为什么不是问题。任何状态更改都会自动通知你，你将收到其他人可能附加到你的问题报告审计跟踪上的任何评论或补丁的副本。

如果有人向你请求额外信息，或者你在初始报告中未提及或发现了某些东西，请提交跟进。导致错误未被修复的首要原因是与发起者的沟通不足。最简单的方法是使用个人 PR 网页上的评论选项，你可以从 PR 搜索页面访问该网页。

如果问题报告在问题解决后仍然保持开放状态，只需添加一条评论，说明问题报告可以关闭，并在可能的情况下解释问题是如何或何时被解决的。

有时问题报告会延迟一两个星期，保持未被处理，没有被任何人分配或评论。当出现问题报告积压增加或在假期季节时，情况会发生。当一个问题报告在几周后仍未得到关注时，值得寻找一个特别有兴趣处理它的贡献者。

有几种方法可以做到这一点，理想情况下按照以下顺序进行，每次尝试不同的沟通渠道之间间隔几天。

- 发送电子邮件到相关列表，征求对报告的意见。
- 加入相关的 IRC 频道。部分列表在这里：https://wiki.freebsd.org/IRC/Channels。告知该频道的人有关问题报告，并请求帮助。请耐心等待，在发布后留在频道中，这样来自世界各个时区的人们有机会跟上。
- 找到对报告中的问题感兴趣的提交者。如果问题出现在特定工具、二进制文件、port、文档或源文件中，请检查 Git 存储库。找到最近对文件进行实质性更改的几个提交者，并尝试通过 IRC 或电子邮件联系他们。提交者及其电子邮件列表可在《FreeBSD 贡献者》文章中找到。

记住，这些人都是志愿者，就像维护者和用户一样，所以他们可能无法立即为问题报告提供帮助。耐心和在跟进过程中保持一致性是极为建议和赞赏的。如果在跟进过程中投入足够的关心和努力，那么找到一个关注问题报告的提交者只是时间问题。

## 10. 如果出现问题

如果你发现有问题，请报告一个 bug！有一个专门用于此目的的类别。如果你无法这样做，请联系 bug 管理人员 bugmeister@FreeBSD.org。

## 11. 进一步阅读

这是一个与正确撰写和处理问题报告相关的资源列表。这并不是完整的列表。

- 如何有效报告错误：Simon G. Tatham 撰写的一篇关于撰写有用（非 FreeBSD 特定）问题报告的优秀文章。
- 问题报告处理指南：FreeBSD 开发人员如何处理问题报告的宝贵见解。
