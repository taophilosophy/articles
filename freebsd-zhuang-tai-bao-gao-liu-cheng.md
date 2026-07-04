# FreeBSD 状态报告流程

- 原文：[FreeBSD Status Report Process](https://docs.freebsd.org/en/articles/freebsd-status-report-process/)

FreeBSD 状态报告每季度发布一次，向公众提供了项目的最新动态，它们通常会附带来自开发者峰会的特别报告。由于状态报告是我们最为显眼的沟通形式之一，因此它们非常重要。

在本文档中以及与 FreeBSD 状态报告相关的其他地方，*状态报告* 这个术语既指代每季度发布的文档，也指其中的单个条目。

## 1. 为作者提供的写作指南

本节提供了一些写作状态报告条目的建议，也包括如何提交条目的说明。

*不要担心你不是英语母语者。[状态团队](mailto:status@FreeBSD.org)会检查你的条目，并修正拼写和语法错误。*

### 1.1. 介绍你的工作

*不要假设阅读报告的人了解你的项目。*

状态报告的受众广泛。它们通常是 FreeBSD 网站上的头条新闻，是人们想了解 FreeBSD 是什么时首先会读到的内容之一。看一个例子：

```sh
新增 abc(4) 支持，包括 frobnicator 兼容性。
```

如果读者熟悉 UNIX 手册页，他们会知道 `abc(4)` 是某种设备。但读者为什么需要关心呢？它是何种设备？对比以下版本：

```sh
代码树中新增了 abc(4) 驱动程序，带来了对 Yoyodyne 公司的 Frobnicator 系列网络接口的支持。
```

现在读者知道 `abc` 是一款网络接口驱动程序。即便他们不使用 Yoyodyne 的产品，你也传达了 FreeBSD 在网络设备方面的支持正在不断增强。

### 1.2. 显示你工作的意义

*状态报告不仅仅是告诉大家做了什么，它们还需要解释为什么要做这些事情。*

继续前面的例子。现在我们支持 Yoyodyne Frobnicator 卡，为什么这件事值得一提？它们广泛使用吗？是否在某些流行设备中使用？是否在 FreeBSD 已经（或希望）占有一席之地的特定领域中使用？它们是否是世界上最快的网卡？状态报告通常会包含类似这样的内容：

```sh
我们将 Cyberdyne Systems T800 导入到代码树中。
```

然后就停了。也许读者是 Cyberdyne 的忠实粉丝，知道 T800 带来了哪些激动人心的新特性，但这不太可能。更有可能的是，他们只是模糊听说过你导入的内容（尤其是导入到 Ports 树中：记住那里面还有超过 35,000 项其他内容）。列出一些新特性或修复的 bug，告诉他们为什么我们拥有新版本是件好事。

### 1.3. 告诉我们一些新信息

*不要重复相同的状态报告条目。*

请记住，状态报告不仅仅是对项目状态的报告，它们还是项目状态变化的报告。如果某个项目正在进行中，用几句话介绍一下它，但接下来要用报告的其余部分谈论新工作。自上次报告以来有什么进展？还剩下什么工作？什么时候可能完成（或者如果“完成”并不适用，那什么时候可能准备好供更广泛的使用、测试、生产部署等）？

### 1.4. 赞助

*不要忘记提及你的赞助商。*

如果你或你的项目获得了赞助、来自某人的奖学金，或者你作为承包商或员工为某公司工作，请在报告中提到。赞助商总是很高兴你感谢他们的资助，同时，能展示他们以这种方式积极支持 FreeBSD 项目，这对他们也有益。最后，这有助于 FreeBSD 更好地了解其重要的消费者。

### 1.5. 待办事项

*如果需要帮助，请明确说明！*

某方面是否需要帮助？有没有任务是其他人可以完成的？你可以用两种方式来使用状态报告中的待办事项部分：请求帮助，或者快速概述剩余的工作量。如果参与项目的人手已足够，或者项目处于增加人手也无法加快进度的状态，那么后一种方式更为合适。列出一些正在进行的大任务，可能还可以标明谁专注于每个任务。

列出任务，提供足够的细节让人们知道他们是否可能有能力完成，并邀请人们与你联系。

### 1.6. 提交报告

你可以通过以下方法提交你的报告：

- 提交 [Phabricator review](https://reviews.freebsd.org/)，并将 *status* 组添加到审阅者列表。你应该将报告放在 `doc/website/content/en/status/` 的适当子目录中（如果缺少此子目录，创建它）；
- 通过 [GitHub 镜像](https://github.com/freebsd/freebsd-doc) 提交 pull request 到文档仓库。你应该将报告放在 `doc/website/content/en/status` 的适当子目录中（如果缺少此子目录，创建它）；
- 发送电子邮件至 [status-submissions@FreeBSD.org](mailto:status-submissions@FreeBSD.org)，并附上你的报告。

可以参考 [AsciiDoc 示例报告模板](https://www.freebsd.org/status/report-sample.adoc)。

## 2. 编辑指南

本节介绍了状态报告的审阅和发布流程。

| 状态报告主页                                   | [https://www.FreeBSD.org/status/](https://www.freebsd.org/status/)                               |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------ |
| 状态报告存档 GitHub 仓库（用于 2017Q4 至 2022Q4 的报告） | [https://github.com/freebsd/freebsd-quarterly](https://www.github.com/freebsd/freebsd-quarterly) |
| 状态团队主要电子邮件地址                             | [status@FreeBSD.org](mailto:status@FreeBSD.org)                                                  |
| 提交报告的电子邮件地址                              | [status-submissions@FreeBSD.org](mailto:status-submissions@FreeBSD.org)                          |
| 用于接收状态报告征集通知的邮件列表                         | [freebsd-status-calls@FreeBSD.org](https://lists.freebsd.org/subscription/freebsd-status-calls)  |
| Phabricator 状态团队主页                       | [https://reviews.freebsd.org/project/88/](https://reviews.freebsd.org/project/profile/88/)       |

### 2.1. 时间线

状态团队始终接受报告，但主要的收集过程发生在每个季度的最后一个月，即 3 月、6 月、9 月和 12 月。1 月、4 月、7 月和 10 月则专注于整理上一季度提交的报告。已完成季度的报告仍可在该月的前 14 天内接受，但不能再晚。状态报告的发布会在报告准备好后尽快完成，同样在这些月份内。

会发出明确的状态报告征集通知，并附上截止日期提醒。

非状态团队成员对提交报告的审阅工作应在 1 月、4 月、7 月和 10 月中旬基本完成（第三方审阅期）。也就是说，除了错别字或轻微的文稿编辑外，状态团队应该能够从 15 日开始整理提交的报告。

|           | 第一季度     | 第二季度     | 第三季度      | 第四季度      |
| --------- | -------- | -------- | --------- | --------- |
| 第一次报告征集通知 | 3 月 1 日  | 6 月 1 日  | 9 月 1 日   | 12 月 1 日  |
| 剩余 1 个月提醒 | 3 月 15 日 | 6 月 15 日 | 9 月 15 日  | 12 月 15 日 |
| 剩余 2 周提醒  | 3 月 31 日 | 6 月 30 日 | 9 月 30 日  | 12 月 31 日 |
| 提交截止日期和第三方审阅期 | 4 月 14 日 | 7 月 14 日 | 10 月 14 日 | 1 月 14 日  |

### 2.2. 征集报告

状态报告的征集通知会发送给以下收件人：

- [freebsd-status-calls@FreeBSD.org 邮件列表](https://lists.freebsd.org/subscription/freebsd-status-calls)；
- 所有上次提交状态报告的人（他们可能有更新或进一步的改进）；
- 根据季节变化：
  - 各种会议组织者：
    - 3 月份的 [AsiaBSDCon](mailto:secretary@asiabsdcon.org)（第一季度）；
    - 5 月份的 [BSDCan](mailto:info@bsdcan.org)（第二季度）；
  - 各种会议参与者：
    - 9 月至 10 月的 EuroBSDcon（第三、四季度）；EuroBSDcon 作为一家组织对写 FreeBSD 状态报告不感兴趣——至少在 2019 年 10 月时并非如此：其原因是该会议并非专门针对 FreeBSD。因此，应向参与会议的 FreeBSD 社区成员征集关于该事件的报告。
  - 谷歌编程之夏的[学生](mailto:soc-students@FreeBSD.org)和他们的[导师](mailto:soc-mentors@FreeBSD.org)。

发送状态报告征集通知的最简单方法是使用 [sendcalls perl 脚本](https://cgit.freebsd.org/doc/tree/tools/sendcalls/sendcalls)，该脚本位于文档 git 仓库的 **tools/sendcalls** 目录中。该脚本会自动发送通知给所有预定的收件人。也可以通过定时任务来使用，例如：

```sh
0      0       1,15,31 3,12        *       cd ~/doc/tools/sendcalls && git pull && ./sendcalls -s 'Lorenzo Salvadore'
0      0       1,15,30 6,9        *       cd ~/doc/tools/sendcalls && git pull && ./sendcalls -s 'Lorenzo Salvadore'
```

>**重要**
>
>如果你负责发送状态报告的征集通知，并且确实使用了 cron 作业，请在 freefall 上运行它并签上你的名字，以便在出现问题时能够推断出是谁配置了该 cron 作业。同时，请将上面的示例更新为包含你的名字，作为额外的安全措施。


也可以考虑像[过去一样](https://forums.freebsd.org/threads/call-for-freebsd-2014q4-october-december-status-reports.49812/)在论坛上发布状态报告的征集通知。

### 2.3. 构建报告

提交的报告会在收到后进行审查，并合并到 **doc/website/content/en/status/** 的适当子目录中。在报告更新的同时，状态团队之外的人也可以审查各个条目并提出修正建议。

通常，内容审查过程的最后一步是在名为 **intro.adoc** 的文件中撰写介绍：只有在所有报告收集完毕后，才能写出好的介绍。如果可能，最好让不同的人写介绍，以增加多样性：不同的人会带来不同的观点，有助于保持新鲜感。

待所有报告和介绍准备好后，需要创建 **\_index.adoc** 文件：这是报告按类别分发并排序的文件。

### 2.4. 发布报告

当状态报告的所有文件准备就绪后，就可以发布它了。

首先，编辑 **doc/website/content/en/status/\_index.adoc**：更新下一个截止日期并添加新报告的链接。然后将更改推送到仓库，状态团队检查一切是否按预期工作。

接着，将新闻条目添加到 **doc/website/data/en/news/news.toml** 中。

以下是新闻条目的示例：

```ini
[[news]]
date = "2021-01-16"
title = "2020 年 10 月-12 月状态报告"
description = "2020 年 10 月到 12 月的状态报告现已发布，包含 42 个条目。"
```

报告的 HTML 版本构建并上线后，使用 [w3m(1)](https://man.freebsd.org/cgi/man.cgi?query=w3m&sektion=1&format=html) 将网站转储为纯文本，例如：

```sh
% w3m -cols 80 -dump https://www.FreeBSD.org/status/report-2021-01-2021-03/ > /tmp/report-2021-01-2021-03.txt
```

[w3m(1)](https://man.freebsd.org/cgi/man.cgi?query=w3m&sektion=1&format=html) 完全支持 Unicode。`-dump` 仅输出 HTML 代码的文本渲染，之后可以删除其中的一些元素，而 `-cols` 确保所有内容都换行到 80 列。

在介绍和第一个条目之间会添加渲染后报告的链接。

最后，报告准备好发送，切换邮件的显示方式（报告应嵌入邮件正文中），并确保它以 UTF-8 编码。

发送两封电子邮件，主题格式为 `FreeBSD 状态报告 - <第一/第二/第三/第四> 季度 <年份>`：

- 一封发送到 [freebsd-announce@FreeBSD.org](https://lists.freebsd.org/subscription/freebsd-announce);

>**重要**
>
> 这封邮件必须获得批准，因此，如果你负责发送此邮件，请确保有人批准（如果处理时间过长，请邮件联系 [postmaster](mailto:postmaster@FreeBSD.org)）。

- 一封发送到 [freebsd-hackers@FreeBSD.org](https://lists.freebsd.org/subscription/freebsd-hackers)，并抄送到 [freebsd-current@FreeBSD.org](https://lists.freebsd.org/subscription/freebsd-current)、[freebsd-stable@FreeBSD.org](https://lists.freebsd.org/subscription/freebsd-stable)，密送 `developers@FreeBSD.org`。
