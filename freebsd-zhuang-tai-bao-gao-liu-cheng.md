# FreeBSD 状态报告流程

版权 © 2023 FreeBSD 文档项目

<details open="" data-immersive-translate-walked="97d547f6-9acd-411f-912a-412f974adf2a"><summary data-immersive-translate-walked="97d547f6-9acd-411f-912a-412f974adf2a" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

Git 和 Git 标志是 Software Freedom Conservancy, Inc.（Git 项目的公司总部）在美国和/或其他国家的注册商标或商标。

GitHub 是 GitHub Inc. 在美国和其他国家注册的商标。

许多制造商和卖家用来区分其产品的名称被宣称为商标。如果这些名称出现在本文档中，并且 FreeBSD 项目知道该商标声明，则这些名称后面会跟有“™”或“®”符号。

</details>

---

FreeBSD 状态报告每季度发布，为公众提供项目进展的一般视图，并经常由开发者峰会的特别报告增补。由于它们是我们最可见的沟通形式之一，因此它们非常重要。

在本文档及与 FreeBSD 状态报告相关的其他地方，状态报告这个表达在一起被用来指代每季度发布的文档以及其中包含的单个条目。

## 1. 作家须知

此部分提供了来自技术撰写经验丰富的 David Chisnall 的一些建议，内容包括如何撰写状态报告条目的说明。还提供了提交条目的指导。

如果您不是以英语为母语，不用担心。状态团队将检查您的条目拼写和语法，并为您进行修正。

### 1.1. 介绍您的工作

*不要假设阅读报告的人了解你的项目。*

状态报告具有广泛的分发。它们通常是 FreeBSD 网站上的头条新闻之一，也是人们想要了解 FreeBSD 是什么的首要内容之一。考虑这个例子：

```
abc(4) support was added, including frobnicator compatibility.
```

如果阅读此文的人熟悉 UNIX 手册页面，他们会知道 abc(4) 是某种设备。但读者为什么应该在意呢？这是什么类型的设备？与此版本进行比较：

```
A new driver, abc(4), was added to the tree, bringing support for
Yoyodyne's range of Frobnicator network interfaces.
```

现在读者知道 abc 是一个网络接口驱动程序。即使他们不使用任何 Yoyodyne 产品，您也已经传达了 FreeBSD 对网络设备的支持正在改善。

### 1.2. 展示您工作的重要性

*状态报告不仅仅是告诉每个人事情已经完成，还需要解释为什么要这样做。*

继续之前的示例。我们现在支持 Yoyodyne Frobnicator 卡片是件有趣的事情。它们是很普及吗？它们在特定的热门设备中使用吗？它们在 FreeBSD 想要有存在（或想要存在）的特定领域中使用吗？它们是地球上最快的网络卡吗？状态报告经常说类似的事情：

```
We imported Cyberdyne Systems T800 into the tree.
```

然后他们停下了。也许读者是一个狂热的 Cyberdyne 粉丝，知道 T800 带来了什么令人兴奋的新功能。这是不太可能的。更有可能的是，他们模糊地听说过你已经导入的任何东西（尤其是到ports树：请记住这里还有超过 30,000 其他东西…）。列举一些新功能或错误修复。告诉他们为什么我们有新版本是件好事。

### 1.3. 告诉我们一些新事物

*不要重复回收相同的状态报告项目。*

请记住，状态报告不仅仅是关于项目状态的报告，而是关于项目状态变化的报告。如果有正在进行的项目，请花几句话介绍它，然后在报告的其余部分讨论新的工作。自上次报告以来取得了哪些进展？还剩下什么工作要做？什么时候可能完成（或者，如果“完成”并不完全适用，何时可能准备好进行更广泛的使用，进行测试，或投入生产等）？

### 1.4. 赞助

*不要忘记你的赞助商。*

如果您或您的项目收到赞助，来自某人的奖学金，或者您已经在公司担任承包商或雇员，请包括在内。赞助商始终会感激您感谢他们的资助，但这对他们来说也是有益的，以此方式展示他们正在积极支持该项目。最后，但并非最不重要的，这有助于 FreeBSD 了解其重要的消费者。

### 1.5. 未完成事项

*如果需要帮助，请明确说明！*

需要帮助吗？是否有其他人可以做的任务？状态报告中的开放项目部分有两种使用方式：征求帮助，或者快速概述剩余的工作量。如果项目上已经有足够的人在工作，或者增加更多的人不会加快进度，那么后者更好。列出一些正在进行的大工作项目，并可能指出每个人的关注点。

列出任务，详细到让人知道他们是否能够完成，并邀请人们联系。

### 1.6. 提交你的报告

提交报告有以下几种方法可供选择：

* 提交一个 Phabricator 评审，并将组状态添加到评审者列表中。如果缺少适当的子目录 doc/website/content/en/status/ ，请将您的报告放入其中（如有需要请创建）；
* 通过其 GitHub 镜像向文档存储库提交拉取请求。您应该将报告放在 doc/website/content/en/status 的适当子目录中（如果缺少则创建）;
* 发送电子邮件至 status-submissions@FreeBSD.org，包括您的报告。

AsciiDoc 示例报告模板可用。

## 2. 编辑说明

本部分介绍了审阅和发布流程的工作原理。

| 状态报告主网页                                                                 | [https://www.FreeBSD.org/status/](https://www.freebsd.org/status/) |
| -------------------------------------------------------------------------------- | --- |
| 存档 GitHub 存储库的状态报告（用于 2017 年第四季度到 2022 年第四季度的报告）： | [https://github.com/freebsd/freebsd-quarterly](https://www.github.com/freebsd/freebsd-quarterly) |
| 主要状态团队电子邮件地址                                                       | [status@FreeBSD.org](mailto:status@FreeBSD.org) |
| 报告提交的电子邮件地址                                                         | [status-submissions@FreeBSD.org](mailto:status-submissions@FreeBSD.org) |
| 接收状态报告电话的邮件列表                                                     | [freebsd-status-calls@FreeBSD.org](https://lists.freebsd.org/subscription/freebsd-status-calls) |
| Phabricator 状态团队主页                                                       | [https://reviews.freebsd.org/project/88/](https://reviews.freebsd.org/project/profile/88/) |

### 2.1. 时间表

报告总是由状态团队接受，但主要的收集过程发生在每个季度的最后一个月，因此在三月、六月、九月和十二月。在这些月份将会发出明确的状态报告请求。一月、四月、七月和十月的月份专用于整理上一个季度提交的报告；这可能包括等待迟交的报告。状态报告的发布在同样的月份进行，一旦报告准备好就会发布。

所有报告提交的截止日期都可以通过电子邮件发送给状态团队来延长，直到延长的截止日期，即季度结束后的第八天。来自ports管理团队的条目默认为延长的截止日期，因为状态报告和季度ports分支之间的重叠。

由非状态团队成员审核的提交报告应在一月/四月/七月/十月中旬（第三方审核截止日期）基本完成。也就是说，除了拼写错误或其他轻微的复制编辑外，状态团队应该能够在 15 日之后很快开始整理提交的报告。请注意，这不是完全冻结，状态团队仍可能接受此时的审核。

|                | 第一季度   | 第二季度   | 第三季度    | 第四季度       |
| ---------------- | ------------ | ------------ | ------------- | ---------------- |
| 第一次呼叫报告 | 3 月 1 日  | 6 月 1 日  | 9 月 1 日   | 12 月 1 日     |
| 提醒剩余 2 周  | 三月十五日 | 六月十五日 | 九月十五日  | 十二月十五日   |
| 最后提醒       | 3 月 24 日 | 6 月 24 日 | 9 月 24 日  | 12 月 24 日    |
| 标准截止日期   | 3 月 31 日 | 六月三十日 | 九月三十日  | 十二月三十一日 |
| 截止日期延长   | 4 月 8 日  | 7 月 8 日  | 十月八日    | 一月八日       |
| 第三方审查冰泥 | 4 月 15 日 | 7 月 15 日 | 10 月 15 日 | 1 月 15 日     |

### 2.2. 要求报告

发送状态报告的接收者如下：

* FreeBSD.org 邮件列表 freebsd-status-calls；
* 所有上次状态报告的提交者（他们可能有更新或进一步改进）；
* 并且，根据季节不同：

  * 各种会议组织者:

    * 3 月的 AsiaBSDCon（第一季度）;
    * 5 月的 BSDCan（第二季度）;
    * 九月至十月的 EuroBSDcon（第三季度）。EuroBSDcon 作为一个组织，不愿意为 FreeBSD 撰写报告（至少在 2019 年 10 月还没有：其理由是该会议并非专门针对 FreeBSD），因此关于此活动的报告应向参加过此活动的 FreeBSD 社区成员询问；
  * Google Summer of Code 学生及其导师。

发送状态报告的最简单方法是使用 doc git 存储库中 tools/sendcalls 目录下的 sendcalls perl 脚本。该脚本会自动向所有预定收件人发送呼叫。它也可以通过 cron 作业使用，例如：

```
0      0       1,15,24 3,6,9,12        *       cd ~/doc/tools/sendcalls && git pull && ./sendcalls -s 'Lorenzo Salvadore'
```

>如果您负责发送状态报告和确实使用 cron 作业，请在 freefall 上运行它，并用您的名字签名，以便在出现问题时可以推断出是谁配置了 cron 作业。另请使用您的名字更新上面的示例，作为额外的安全措施。 

也许也值得在论坛上要求报告，就像过去做过的那样。

### 2.3. 建立报告

提交的报告将在收到后审查并合并到 doc/website/content/en/status/ 目录的适当子目录中。在更新报告的同时，状态团队外的人员也可以审查各个条目并提出修正建议。

内容审查流程通常的最后一步是在名为 intro.adoc 的文件中编写介绍：只有在收集到所有报告后才能写出一个好的介绍。如果可能，建议请不同的人编写介绍，以增加多样性：不同的观点将使其更加新鲜。

一旦所有报告和介绍准备就绪，需要创建 _index.adoc 文件：这是将报告分配到各种类别并进行排序的文件。

### 2.4. 发布报告

当状态报告的所有文件准备就绪时，是发布的时候了。

首先编辑 doc/website/content/en/status/_index.adoc：更新下一个截止日期并添加到新报告的链接。然后将更改推送到存储库，并由状态团队检查所有工作是否正常。

然后将主网站页面的新闻条目添加到 doc/website/data/en/news/news.toml 中。

这里是新闻条目的示例：

```
[[news]]
date = "2021-01-16"
title = "October-December 2020 Status Report"
description = "The <a href="https://www.FreeBSD.org/status/report-2020-10-2020-12.html">October to December 2020 Status Report</a> is now available with 42 entries."
```

一旦报告的 HTML 版本构建完成并上线，使用 w3m(1) 将网站以纯文本形式转储，例如：

```
% w3m -cols 80 -dump https://www.FreeBSD.org/status/report-2021-01-2021-03/ > /tmp/report-2021-01-2021-03.txt
```

w3m(1)拥有完整的 Unicode 支持。 -dump 只是简单地输出 HTML 代码的文本渲染，然后可以剪裁一些元素，而 -cols 则确保一切都被包裹到 80 列。

简介和第一篇文章之间添加了渲染报告的链接。

报告终于准备好发送了，切换配置（报告应内联），并确保它以 UTF-8 编码。

会发送两封电子邮件，两封邮件的主题格式为 FreeBSD Status Report - <First/Second/Third/Fourth> Quarter <year> ：

* 一封发送到 freebsd-announce@FreeBSD.org；

>这封邮件必须被批准，因此如果您负责发送此邮件，请确保有人批准（如果时间过长，请联系邮件管理员）。 

* 一封邮件发送至 freebsd-hackers@FreeBSD.org，抄送 freebsd-current@FreeBSD.org 和 freebsd-stable@FreeBSD.org，密送 developers@FreeBSD.org 。
