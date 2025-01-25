# 如何从 FreeBSD-questions 邮件列表中获得最佳结果

<details open="" data-immersive-translate-walked="0b57dc3f-b9a4-4087-a732-0f8e10cb9e52"><summary data-immersive-translate-walked="0b57dc3f-b9a4-4087-a732-0f8e10cb9e52" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

Microsoft、IntelliMouse、MS-DOS、Outlook、Windows、Windows Media 和 Windows NT 是 Microsoft Corporation 在美国和/或其他国家的注册商标或商标。

Motif，OSF/1 和 UNIX 是 The Open Group 在美国和其他国家的注册商标，IT DialTone 和 The Open Group 是 The Open Group 的商标。

QUALCOMM 和 Eudora 是 QUALCOMM Incorporated 的注册商标。

许多制造商和销售商用来区分其产品的标识被声明为商标。在本文档中出现这些标识的地方，FreeBSD 项目已经意识到商标声明，并在标识后面加上了“™”或“®”符号。

</details>

## 摘要

本文档为希望准备发送电子邮件到 FreeBSD-questions 邮件列表的人提供了有用的信息。提供了建议和提示，以最大化读者收到有用回复的机会。

本文档会定期发布到 FreeBSD-questions 邮件列表。

---

## 1. 引言

FreeBSD-questions 是一个由 FreeBSD 项目维护的邮件列表，旨在帮助有关 FreeBSD 常规使用问题的人们。另一组， FreeBSD-hackers ，讨论更高级的问题，如未来的开发工作。

>"黑客"一词与侵入他人计算机无关。后者的正确术语是"骇客"，但大众媒体尚未了解。FreeBSD 的黑客们坚决反对破解安全性，并且与此毫无关系。有关黑客的更详细描述，请参阅埃里克·雷蒙德的《如何成为黑客》。 

这是一篇常规帖子，旨在帮助那些从 FreeBSD-questions 寻求建议的人（“新手”），也帮助那些回答问题的人（“黑客”）。

不可避免地，两个群体的不同观点会导致一些摩擦。新手们指责黑客们傲慢、自负且不乐于助人，而黑客们则指责新手愚蠢、不会读懂简单的英文，并希望所有事情都能轻松获得。当然，这两种说法都有一定道理，但大多数观点源自挫折感。

在本文中，我希望做一些事情来减轻这种挫折，帮助每个人从 FreeBSD-questions 中获得更好的结果。在接下来的部分，我会推荐如何提交问题；之后，我们将看看如何回答问题。

## 2. 如何订阅 FreeBSD-questions

FreeBSD-questions 是一个邮件列表，所以您需要邮件访问权限。将您的 WWW 浏览器指向 FreeBSD 常见问题邮件列表。在名为"订阅或取消订阅在线"的部分填写"您的电子邮件地址"字段，然后点击"订阅"。或者发送电子邮件至 freebsd-questions+subscribe@freebsd.org。

您将收到来自 mlmmj 的确认邮件；按照附上的说明完成您的订阅。

## 3. 如何取消订阅 FreeBSD-questions

打开您的 WWW 浏览器，转到 FreeBSD 常见问题邮件列表。在标题为"在线订阅或取消订阅"的部分，填写"您的电子邮件地址"字段，然后点击"取消订阅"。或发送电子邮件至 freebsd-questions+unsubscribe@freebsd.org。

您将收到来自 mlmmj 的确认消息；按照其中包含的说明完成取消订阅。

## 4. 我应该问 -questions 还是 -hackers ？

两个邮件列表处理有关 FreeBSD 的一般问题， FreeBSD-questions 和 FreeBSD-hackers 。在某些情况下，很难确定应该问哪个组。然而，以下标准应该对 99%的所有问题有帮助：

1. 如果问题是一般性质的，请问 FreeBSD-questions 。例如可能涉及有关安装 FreeBSD 或使用特定 UNIX® 实用程序的问题。
2. 如果你认为问题与一个 bug 有关，但不确定，或者你不知道如何查找，发送信息到 `FreeBSD-questions`。
3. 如果问题与 bug 有关，并且你 _确定_ 它是一个 bug（例如，你能指出代码中发生问题的位置，并且可能有一个修复方法），那么发送信息到 `FreeBSD-hackers`。
4. 如果问题与 FreeBSD 的增强功能有关，并且你可以提出关于如何实现这些功能的建议，那么发送信息到 `FreeBSD-hackers`。

还有许多其他专门的邮件列表，满足更特定的兴趣。上面提到的标准仍然适用，最好遵循它们，因为这样你更有可能获得好的结果。

## 5. 在提交问题之前

在向邮件列表之一提问之前，您可以（也应该）自己做一些事情。

- 尝试独自解决问题。如果您发布的问题表明您已经尝试解决问题，通常会吸引更多人的正面关注。尝试自己解决问题也会增强您对 FreeBSD 的理解，并最终让您能够利用自己的知识帮助他人，回答邮件列表上发布的问题。
- 阅读手册页面和 FreeBSD 文档（可以安装在/usr/doc 目录下或通过 http://www.FreeBSD.org 访问），特别是手册和常见问题解答。
- 浏览和/或搜索邮件列表的存档，查看您的问题或类似问题是否已经在列表上提出（并可能得到解答）。您可以在 https://www.FreeBSD.org/mail 和 https://www.FreeBSD.org/search/#mailinglists 分别浏览和/或搜索邮件列表的存档。
- 使用搜索引擎，如 Google 或 Yahoo，寻找问题的答案。

## 6. 如何提交问题

在向 FreeBSD-questions 提交问题时，请考虑以下几点：

- 记住，没有人会因回答 FreeBSD 问题而得到报酬。他们是出于自愿才这样做的。通过提交一个表述明晰、提供尽可能多相关信息的问题，你可以积极地影响这种自愿行为。通过提交一个不完整、难以辨认或粗鲁的问题，你可以消极地影响这种自愿行为。即使你遵循这些规则，也有可能向 FreeBSD-questions 发送消息而不会得到答复。如果不遵循这些规则，得不到答复的可能性更大。接下来的内容中，我们将探讨如何最大程度地利用你向 FreeBSD-questions 提出问题的机会。
- 并不是每个回答 FreeBSD 问题的人都会阅读每一条消息：他们会查看主题行并决定是否感兴趣。显然，指定一个主题符合你的利益。"FreeBSD 问题"或"帮助"不足够。如果根本没有提供主题，很多人将不会打扰阅读。如果你的主题不够具体，可以回答问题的人可能就不会阅读它。
- 排版你的消息使之易读，且请不要大声喊叫!!!!!。我们明白很多人并非以英语为母语，我们会尽量包容这一点，但读一封充满拼写错误或没有换行的消息真的很痛苦。不要低估一封格式混乱的邮件消息产生的影响，这不仅仅影响 FreeBSD-questions 邮件列表。你的邮件消息就是其他人看到的你，如果格式混乱，每段只有一行，拼写错误或错误百出，那会给人留下很差的印象。
  很多格式混乱的消息来自糟糕的邮件发送程序或配置不当的邮件发送程序。已知以下邮件发送程序会发送格式不良的消息，而你可能并不知情：

  - exmh
  - Microsoft® Exchange
  - 尽量不要使用 MIME：许多人使用不太兼容 MIME 的邮件客户端。

- 确保您的时间和时区设置正确。这看起来可能有点愚蠢，因为您的消息仍然能够送达，但您试图联系的许多人每天可能会收到数百封消息。他们经常会按主题和日期对收件箱中的消息进行排序，如果您的消息不在第一个答复之前到达，他们可能会认为自己漏掉了，从而不去查看。
- 不要在同一条消息中包含无关的问题。首先，长篇消息往往会吓跑人们，其次，要让所有能够回答所有问题的人都阅读消息更加困难。
- 尽可能提供详细信息。这是一个困难的领域，我们需要详细阐述您需要提交的信息，但以下是一个开始：

  - 几乎每种情况下，知道您运行的 FreeBSD 版本是很重要的。这对于 FreeBSD-CURRENT 尤为重要，您还应指定源代码的日期，尽管当然不应将有关-CURRENT 的问题发送到 FreeBSD-questions。
  - 对于可能与硬件有关的任何问题，请告诉我们您的硬件信息。在怀疑的情况下，假设可能是硬件问题。您使用什么类型的 CPU？有多快？什么样的主板？多少内存？什么外设？这里需要进行判断，但是 dmesg(8)命令的输出通常非常有用，因为它不仅告诉您正在运行的硬件类型，还告诉您 FreeBSD 的版本。
  - 如果您收到错误消息，请不要说"我收到错误消息"，应该说（例如）"我收到了错误消息'无法连接到主机'"。
  - 如果您的系统发生崩溃，请不要说"我的系统崩溃了"，应该说（例如）"我的系统崩溃了，并显示消息'自由 vnode 不是'"。
  - 如果您在安装 FreeBSD 时遇到困难，请告诉我们您使用的硬件。特别重要的是要知道您机器上安装的板卡的 IRQs 和 I/O 地址。
  - 如果 PPP 运行困难，描述配置。您使用哪个版本的 PPP？您使用什么样的身份验证？您有静态 IP 地址还是动态 IP 地址？日志文件中收到了什么样的消息？

- 您需要提供的大部分信息都是程序输出，比如 dmesg(8)，或控制台消息，通常出现在/var/log/messages 中。不要试图通过重新输入文本来复制这些信息；这很麻烦，你很可能会出错。要发送日志文件内容，可以复制文件并使用编辑器将信息裁剪为相关信息，或者剪切并粘贴到您的消息中。对于像 dmesg(8)这样的程序输出，请将输出重定向到一个文件并包含在其中。例如，

  ```
  % dmesg > /tmp/dmesg.out
  ```

  这将信息重定向到文件/tmp/dmesg.out。

- 如果你做了这一切，但仍然没有得到答案，可能有其他原因。例如，问题太复杂，没有人知道答案，或者知道答案的人不在线。如果一周后还没有得到答案，重新发送消息可能会有帮助。但是，如果你第二次发送消息仍然没有得到答案，那么你可能无法从这个论坛得到答案。一遍又一遍地重新发送相同的消息只会让你不受欢迎。

总之，假设你知道以下问题的答案（是的，在每种情况下都是相同的问题）。你选择这两个问题中你更愿意回答哪一个：

例子 1。消息 1

```
Subject: HELP!!?!??
I just can't get hits damn silly FereBSD system to
workd, and Im really good at this tsuff, but I have never seen
anythign sho difficult to install, it jst wont work whatever I try
so why don't you guys tell me what I doing wrong.
```

示例 2。信息 2

```
Subject: Problems installing FreeBSD

I've just got the FreeBSD 2.1.5 CDROM from Walnut Creek, and I'm having a lot
of difficulty installing it.  I have a 66 MHz 486 with 16 MB of
memory and an Adaptec 1540A SCSI board, a 1.2GB Quantum Fireball
disk and a Toshiba 3501XA CDROM drive.  The installation works just
fine, but when I try to reboot the system, I get the message
Missing Operating System.
```

## 7。如何回答问题

通常，您会希望发送额外信息来回答您已经发送的问题。这样做的最佳方式是回复您最初的消息。这有三个优点：

1. 你包含原始消息文本，这样人们就会知道你在说什么。不过，别忘了删掉不必要的文本。
2. 主题行中的文本保持不变（你确实记得放一个进去了吧？）。许多邮件客户端会按主题对消息进行排序。这有助于将消息分组在一起。
3. 头部中的消息引用号将指向先前的消息。一些邮件客户端，比如 mutt，可以将消息线程化，显示消息之间的确切关系。

## 8. 如何回答问题

在回答 FreeBSD-questions 的问题之前，请考虑：

1. 许多提交问题的要点同样适用于回答问题。请阅读它们。
2. 有人已经回答了这个问题吗？检查的最简单方法是按主题对你的收件箱排序：然后（希望）你会看到问题后面跟着任何答案，全部在一起。如果有人已经回答了，这并不意味着你不能再发送另一个答案。但是最好先阅读所有其他答案。
3. 你有没有超出已经说过的内容可以贡献的东西？通常，“是的，我也是”这样的回答并没有太大帮助，尽管也有例外，比如当有人描述他们遇到的问题时，他们不知道是他们的错还是硬件或软件出了问题。如果你发送了一个“我也是”的答案，你应该同时包含任何进一步相关的信息。
4. 你确定你理解了问题吗？非常频繁地，提问的人会感到困惑或者表达得不够清楚。即使对系统有最好的理解，也很容易发送一个没有回答问题的回复。这并不会帮助：你会让提问的人比以前更加沮丧或困惑。如果没有其他人回答，而你也不太确定，你总是可以要求更多信息。
5. 你确定你的答案是正确的吗？如果不是，等一天左右。如果没有其他人想出更好的答案，你仍然可以回复并说，例如，“我不知道这是否正确，但是因为没有其他人回复，为什么不尝试用一只青蛙代替你的 ATAPI CDROM 呢？”。
6. 除非有充分的理由，回复发件人和 FreeBSD-questions。FreeBSD-questions 上有许多“潜水者”：他们通过阅读他人发送和回复的消息来学习。如果你将一条对大家有兴趣的消息从列表中移除，就等于剥夺了这些人的信息。要小心处理群发回复；很多人发送带有成百上千个抄送地址的消息。如果是这种情况，请确保适当地修剪 Cc: 行。
7. 包括原始消息中的相关文本。将其修剪到最小限度，但不要过度。即使对于没有阅读原始消息的人，也应该能理解你在谈论什么。
8. 使用某种技术来识别哪些文本来自原始消息，哪些文本是您添加的。我个人发现在原始消息前面添加“>”符号效果最好。在“>”后留下空格，并在您的文本和原始文本之间留下空行都能使结果更易读。
9. 将您的回复放在正确的位置（在要回复的文本之后）。阅读每个回复都在要回复的文本之前的回应线程非常困难。
10. 大多数电子邮件客户端在回复时会通过在主题行前添加诸如“Re: ”的文本来更改主题行。如果您的电子邮件客户端不自动执行此操作，则应手动执行。
11. 如果提交者未遵守格式规范（行太长，主题行不当），请修正。在主题行不正确的情况下（例如“HELP!!??”），将主题行更改为（比如）“Re: Difficulties with sync PPP（原主题: HELP!!??）”。这样其他人要跟踪讨论会更容易。在这种情况下，适当说明你做了什么以及为什么这样做，但尽量不要冒犯。如果你觉得不冒犯就无法回答，请不要回答。
    如果只是想回复一条信息因为其格式不佳，只需回复提交者，而不是整个列表。如果愿意，可以直接用此消息回复他。
