# 提交者指南

版权 © 1999-2022 FreeBSD 文档项目

<details open="" data-immersive-translate-walked="2849f52f-ac64-4ddc-8271-61401da3f3cf"><summary data-immersive-translate-walked="2849f52f-ac64-4ddc-8271-61401da3f3cf" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

Coverity 是 Coverity, Inc. 的注册商标；Coverity Extend, Coverity Prevent 和 Coverity Prevent SQS 是 Coverity, Inc. 的商标。

IBM, AIX, OS/2, PowerPC, PS/2 和 ThinkPad 是 International Business Machines Corporation 在美国、其他国家或两者都有的商标。

Git 和 Git 标识是 Software Freedom Conservancy, Inc. 的注册商标或商标，Git 项目的公司总部位于美国和/或其他国家。

GitHub 是 GitHub Inc.在美国和其他国家注册的商标。

GITLAB 是 GitLab Inc.在美国和其他国家和地区注册的商标。

Intel、Celeron、Centrino、Core、EtherExpress、i386、i486、Itanium、Pentium 和 Xeon 是 Intel Corporation 或其子公司在美国和其他国家的商标或注册商标。

制造商和销售商用来区分其产品的许多名称都被声明为商标。在本文档中出现这些名称时，如果 FreeBSD 项目意识到商标声明，这些名称将被标注为“™”或“®”符号。

</details>

摘要

本文档为 FreeBSD 提交者社区提供信息。所有新提交者在开始之前应阅读本文档，现有提交者强烈建议定期审阅。

几乎所有的 FreeBSD 开发人员都有权限访问一个或多个存储库。然而，少数开发人员没有权限，这里的一些信息也适用于他们。（例如，有些人只有权限与问题报告数据库一起工作。）更多信息请参阅《非 committer 开发人员特定问题》。

这份文档也可能对希望了解该项目运作方式的 FreeBSD 社区成员感兴趣。

---

## 1. 行政细节

| _登录方法_                | ssh(1)，仅限协议 2                                                                                                                                                                                                                                                                                                                       |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _主 Shell 主机_           | `freefall.FreeBSD.org`                                                                                                                                                                                                                                                                                                                   |
| _参考机器_                | ref _.FreeBSD.org , universe_.freeBSD.org (参见 FreeBSD Project Hosts)                                                                                                                                                                                                                                                                   |
| _SMTP 主机_               | smtp.FreeBSD.org:587 （另请参阅 SMTP 访问设置）。                                                                                                                                                                                                                                                                                        |
| <em>src/</em> Git 仓库    | `ssh://git@gitrepo.FreeBSD.org/src.git`                                                                                                                                                                                                                                                                                                  |
| <em>doc/</em> Git 仓库    | `ssh://git@gitrepo.FreeBSD.org/doc.git`                                                                                                                                                                                                                                                                                                  |
| <em>ports/</em> Git 仓库  | `ssh://git@gitrepo.FreeBSD.org/ports.git`                                                                                                                                                                                                                                                                                                |
| _内部邮件列表_            | 开发者（技术上称为全体开发者），文档开发者，文档提交者，ports-开发者，ports-提交者，源码开发者，源码提交者。（每个项目仓库都有自己的 -developers 和 -committers 邮件列表。这些列表的存档可以在 freefall.FreeBSD.org 上的 /local/mail/repository-name-developers-archive 和 /local/mail/repository-name-committers-archive 文件中找到。） |
| _核心团队每月报告_        | /home/core/public/reports on the FreeBSD.org 集群.                                                                                                                                                                                                                                                                                       |
| _Ports 管理团队每月报告_  | 在 FreeBSD.org 集群的 /home/portmgr/public/monthly-reports。                                                                                                                                                                                                                                                                             |
| 值得注意的 src/ Git 分支: | stable/n （ n -STABLE), main （-CURRENT）                                                                                                                                                                                                                                                                                                |

ssh(1)是连接到项目主机所需的。有关更多信息，请参阅 SSH 快速入门指南。

有用的链接：

- [FreeBSD 项目内部页面](https://www.freebsd.org/internal/)
- [FreeBSD 项目主机](https://www.freebsd.org/internal/machines/)
- [FreeBSD 项目管理组](https://www.freebsd.org/administration/)

## 2. FreeBSD 的 OpenPGP 密钥

使用符合 OpenPGP（Pretty Good Privacy）标准的加密密钥来验证 FreeBSD 项目的提交者。携带重要信息的消息，如公共 SSH 密钥，可以用 OpenPGP 密钥签名，以证明它们确实来自提交者。有关更多信息，请参阅迈克尔·卢卡斯（Michael Lucas）的《PGP＆GPG：实用偏执狂的电子邮件》和 http://en.wikipedia.org/wiki/Pretty_Good_Privacy。

### 2.1. 创建密钥

可以使用现有密钥，但应首先使用文档/工具/checkkey.sh 进行检查。在这种情况下，确保密钥具有 FreeBSD 用户 ID。

对于那些尚未拥有 OpenPGP 密钥或需要生成新密钥以符合 FreeBSD 安全要求的人员，这里展示如何生成一个。

1. 安装 security/gnupg。在 ~/.gnupg/gpg.conf 中输入以下行以设置签名和新密钥偏好的最低可接受默认值（详见 GnuPG 选项文档获取更多详情）：

   ```
   # Sorted list of preferred algorithms for signing (strongest to weakest).
   personal-digest-preferences SHA512 SHA384 SHA256 SHA224
   # Default preferences for new keys
   default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 CAMELLIA256 AES192 CAMELLIA192 AES CAMELLIA128 CAST5 BZIP2 ZLIB ZIP Uncompressed
   ```

2. 生成一个密钥：

   ```
   % gpg --full-gen-key
   gpg (GnuPG) 2.1.8; Copyright (C) 2015 Free Software Foundation, Inc.
   This is free software: you are free to change and redistribute it.
   There is NO WARRANTY, to the extent permitted by law.

   Warning: using insecure memory!
   Please select what kind of key you want:
      (1) RSA and RSA (default)
      (2) DSA and Elgamal
      (3) DSA (sign only)
      (4) RSA (sign only)
   Your selection? 1
   RSA keys may be between 1024 and 4096 bits long.
   What keysize do you want? (2048) 2048
   Requested keysize is 2048 bits
   Please specify how long the key should be valid.
   	 0 = key does not expire
         <n>  = key expires in n days
         <n>w = key expires in n weeks
         <n>m = key expires in n months
         <n>y = key expires in n years
   Key is valid for? (0) 3y
   Key expires at Wed Nov  4 17:20:20 2015 MST
   Is this correct? (y/N) y
   GnuPG needs to construct a user ID to identify your key.

   Real name: Chucky Daemon
   Email address: notreal@example.com
   Comment:
   You selected this USER-ID:
   "Chucky Daemon <notreal@example.com>"

   Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
   You need a Passphrase to protect your secret key.
   ```

|     | 使用 2048 位密钥，并设置为三年到期，目前提供足够的保护（2022 年 10 月）。                                                                                                                                                                                                                                                                                                                                                                                        |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|     | 三年的密钥寿命足够短，可以淘汰因计算机性能提升而变弱的密钥，同时又足够长，可以减少密钥管理问题。                                                                                                                                                                                                                                                                                                                                                                 |
|     | 在这里使用您的真实姓名，最好与政府发布的身份证上显示的姓名相匹配，以便他人更容易验证您的身份。可能帮助他人识别您的文本可以输入到 Comment 部分中。<br /><br />在输入电子邮件地址后，会要求输入密码短语。创建安全密码短语的方法存在争议。与其建议一种方式，不如提供一些链接到描述各种方法的网站：https://world.std.com/reinhold/diceware.html, https://www.iusmentis.com/security/passphrasefaq/, https://xkcd.com/936/, https://en.wikipedia.org/wiki/Passphrase. |

保护私钥和密码短语。如果私钥或密码短语可能已经泄露或被披露，请立即通知 accounts@FreeBSD.org 并撤销密钥。

将新密钥提交显示在新提交者的步骤中。

## 3. FreeBSD 集群的 Kerberos 和 LDAP Web 密码

FreeBSD 集群需要 Kerberos 密码才能访问某些服务。Kerberos 密码也用作 LDAP Web 密码，因为 LDAP 在集群中代理到 Kerberos。需要此密码的一些服务包括：

- [Bugzilla](https://bugs.freebsd.org/bugzilla)
- [ 爬虫](https://ci.freebsd.org/)

要在 FreeBSD 集群中创建新的 Kerberos 帐户，或者使用随机密码生成器为现有帐户重置 Kerberos 密码：

```
% ssh kpasswd.freebsd.org
```

|     | 这必须在 FreeBSD.org 集群之外的机器上执行。 |
| --- | ------------------------------------------- |

Kerberos 密码也可以通过登录 freefall.FreeBSD.org 并运行以下命令手动设置：

```
% kpasswd
```

|     | 除非以前已经使用过 FreeBSD.org 集群的基于 Kerberos 的服务，否则会显示 Client unknown 。此错误意味着必须首先使用上面显示的 ssh kpasswd.freebsd.org 方法来初始化 Kerberos 帐户。 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

## 4. 提交位类型

FreeBSD 存储库具有多个组件，这些组件在组合时支持基本操作系统源代码、文档、第三方应用程序基础设施和各种维护的实用程序。当分配 FreeBSD 提交位时，指定可以使用该位的树的区域。通常，与位关联的区域反映了谁授权分配提交位。可能会在以后的某个时间添加额外的管理区域：在发生这种情况时，提交者应遵循该树区域的正常提交位分配程序，寻求适当实体的批准，并可能在一段时间内为该区域获得导师。

| _提交者类型_ | _负责人_ | _树组件_                |
| ------------ | -------- | ----------------------- |
| src          | 核心@    | 源码/                   |
| 文档         | doceng@  | doc/, ports/, src/ 文档 |
| ports        | portmgr@ | ports/                  |

早期分配的提交权限在权威领域概念发展之前可能适用于树的许多部分。然而，常识告诉我们，尚未在树的某个领域工作过的提交者在提交之前应寻求审查，从适当的负责方那里获得批准，和/或与导师合作。由于关于代码维护的规则在树的不同领域之间有所不同，这对于在较不熟悉的领域工作的提交者以及其他在树上工作的人同样有益。

鼓励提交者在正常开发过程中寻求他们工作的审查，无论工作发生在树的哪个部分。

### 其他树提交者活动政策

- 所有提交者可以修改 src/share/misc/committers-*.dot，src/usr.bin/calendar/calendars/calendar.freebsd 和 ports/astro/xearth/files。
- 文档提交者可以在不需要 src 提交者批准的情况下向 src 文件提交文档更改，例如手册页面、README、幸运数据库、日历文件和注释修复，但仍需按照常规的提交程序进行。
- 任何提交者都可以在获得具有相应权限且未被指导的提交者的“批准”的情况下更改任何其他树。被指导的提交者可以提供“审阅”，但不能提供“批准”。
- 提交者可以通过通常的过程获得额外的权限，即找到一位导师将他们提议给核心团队、文档工程团队或 Ports 管理团队，并在获得批准后，他们将被添加到“访问”列表中，并进行正常的指导期，这期间将继续涉及“批准”。

#### 4.1.1. 文档隐式（全面）批准

一些类型的修复工作得到了文档工程团队的“全面批准”<doceng@FreeBSD.org>，允许任何提交者修复文档树的任何部分中的那些问题类别。如果作者没有文档提交权限，则这些修复工作不需要经过文档提交者的批准或审查。

全面批准适用于以下类型的修复工作：

- 拼写错误
- 琐事修复标点符号、URL、日期、路径和文件名中过时或不正确的信息，以及可能让读者困惑的其他常见错误。

多年来，在文档树中授予了一些隐式许可。此列表显示了最常见的情况：

- 更改文档/content/en/books/porters-handbook/versions/_index.adoc _ __FreeBSD_version 值（Porter's Handbook），主要用于 src 提交者。
- 文档/shared/contrib-additional.adoc 附加的 FreeBSD 贡献者维护变更。
- 新提交者的所有步骤，与文档相关
- 安全公告；勘误通知；发布版本；由安全官员团队 <security-officer@FreeBSD.org> 和发布工程团队 <re@FreeBSD.org> 使用。
- 更改网站/content/en/donations/donors.adoc 由捐赠联络办公室使用<donations@FreeBSD.org>。

在任何提交之前，都需要进行构建测试；有关更多详细信息，请参阅新贡献者 FreeBSD 文档项目入门指南中的“概述”和“FreeBSD 文档构建过程”部分。

## 5. Git 入门

### 5.1. Git 基础

当人们搜索 "Git Primer" 时，会出现许多好选择。Daniel Miessler 的 A git primer 和 Willie Willus 的 Git - Quick Primer 都是很好的概述。Git 书籍也很全面，但篇幅更长 https://git-scm.com/book/en/v2。此外，还有这个网站 https://dangitgit.com/，用于解决 Git 的常见陷阱和问题，以防需要指导解决问题。最后，针对计算机科学家的介绍对于某些人在解释 Git 的世界观方面很有帮助。

本文将假设您已经阅读过它，并尽量不再详细介绍基础知识（尽管会简要介绍）。

### 5.2. Git 迷你入门手册

本入门手册的范围比旧版 Subversion 入门手册更为简单，但应涵盖基础知识。

#### 5.2.1. 范围

如果你想下载 FreeBSD，从源代码编译它，并且一般通过这种方式保持最新状态，那么这个入门指南适合你。它涵盖了获取源代码、更新源代码、二分查找，并简要介绍了如何应对一些本地更改。它涵盖了基础知识，并尽力提供良好的指引，以便在读者发现基础知识不足时，能深入了解其他内容。本指南的其他部分涵盖了与贡献项目相关的更高级主题。

本节的目标是突出跟踪源代码所需的那些 Git 要点。它们假设对 Git 有基本的理解。网上有很多 Git 的入门教程，但 Git Book 提供了其中较好的讲解。

#### 5.2.2. 开发者入门

本节介绍了提交者为开发者或贡献者推送提交的读写访问。

##### 5.2.2.1. 日常使用

|     | 在下面的示例中，将 ${repo} 替换为所需 FreeBSD 仓库的名称： doc 、 ports 、或 src 。 |
| --- | ----------------------------------------------------------------------------------- |

- 克隆存储库：

  ```
  % git clone -o freebsd --config remote.freebsd.fetch='+refs/notes/*:refs/notes/*' https://git.freebsd.org/${repo}.git
  ```

  然后你应该把官方镜像作为你的远程：

  ```
  % git remote -v
  freebsd  https://git.freebsd.org/${repo}.git (fetch)
  freebsd  https://git.freebsd.org/${repo}.git (push)
  ```

- 配置 FreeBSD 提交者数据：repo.freebsd.org 中的提交钩子会检查“Commit”字段是否与 FreeBSD.org 中提交者信息匹配。获取建议的配置的最简单方法是在 freefall 上执行 /usr/local/bin/gen-gitconfig.sh 脚本：

  ```
  % gen-gitconfig.sh
  [...]
  % git config user.name (your name in gecos)
  % git config user.email (your login)@FreeBSD.org
  ```

- 设置推送 URL：

  ```
  % git remote set-url --push freebsd git@gitrepo.freebsd.org:${repo}.git
  ```

  然后，您应该将获取和推送 URL 分开设置为最有效的配置：

  ```
  % git remote -v
  freebsd  https://git.freebsd.org/${repo}.git (fetch)
  freebsd  git@gitrepo.freebsd.org:${repo}.git (push)
  ```

  再次注意， gitrepo.freebsd.org 已被规范化为 repo.freebsd.org 。

- 安装提交消息模板挂钩：用于文档存储库：

  ```
  % cd .git/hooks
  % ln -s ../../.hooks/prepare-commit-msg
  ```

  对 ports 存储库：

  ```
  % git config --add core.hooksPath .hooks
  ```

  对 src 存储库：

  ```
  % cd .git/hooks
  % ln -s ../../tools/tools/git/hooks/prepare-commit-msg
  ```

##### 5.2.2.2. "admin" 分支

" access " 和 " mentors " 文件存储在每个库中的孤立分支 " internal/admin " 中。

以下示例是如何将 " internal/admin " 分支检出到本地分支 " admin " 的：

```
% git config --add remote.freebsd.fetch '+refs/internal/*:refs/internal/*'
% git fetch
% git checkout -b admin internal/admin
```

或者，您可以为分支 admin 添加一个工作树：

```
git worktree add -b admin ../${repo}-admin internal/admin
```

要在网页上浏览 internal/admin 分支： https://cgit.freebsd.org/${repo}/log/?h=internal/admin

要推送，请指定完整的 refspec：

```
git push freebsd HEAD:refs/internal/admin
```

#### 5.2.3. 与 FreeBSD src 树保持最新

第一步：克隆树。这会下载整个树。有两种下载方式。大多数人会希望对存储库进行深层克隆。然而，有时您可能希望进行浅克隆。

##### 5.2.3.1. 分支名称

FreeBSD-CURRENT 使用 main 分支。

main 是默认分支。

对于 FreeBSD-STABLE，分支名称包括 stable/12 和 stable/13 。

对于 FreeBSD-RELEASE，发布工程分支名称包括 releng/12.4 和 releng/13.2 。

https://www.freebsd.org/releng/ 显示:

- main 和 stable/⋯ 分支开放
- 每个分支在标记发布时都会被冻结。

例子：

- 在 releng/13.1 分支上标记 release/13.1.0。
- 在 releng/13.2 分支上打标签 release/13.2.0。

##### 5.2.3.2. 仓库

请查看管理详细信息获取 FreeBSD 源代码的最新信息。可以从该页面的 $URL 获取下面的信息。

注意：该项目不使用子模块，因为它们不适合我们的工作流程和开发模型。我们如何跟踪第三方应用程序的更改在其他地方讨论，并且通常不太关注普通用户。

##### 5.2.3.3. 深度克隆

深度克隆会拉取整个树，以及所有的历史记录和分支。这是最容易做的。它还允许您使用 Git 的工作树功能，将所有活动分支都检出到单独的目录中，但仅保留一个存储库的副本。

```
% git clone -o freebsd $URL -b branch [<directory>]
```

— 将创建一个深度复刻。 branch 应该是前一节中列出的分支之一。如果没有提供 branch ：将使用默认值（ main ）。如果没有提供 <directory> ：新目录的名称将匹配仓库的名称（doc、ports 或 src）。

如果您对历史记录感兴趣、计划进行本地更改或计划处理多个分支，则需要一个深度复刻。这也是最容易保持更新的。如果您对历史记录感兴趣，但只处理一个分支并且空间有限，您还可以使用 --single-branch 仅下载一个分支（尽管有些合并提交不会引用合并自的分支，这对于一些对历史详细版本感兴趣的用户可能很重要）。

##### 5.2.3.4. 浅层复刻

浅克隆仅复制最新的代码，但几乎不包含历史记录。当您需要构建特定版本的 FreeBSD 或者刚开始跟踪树时，这可能很有用。您也可以用它来限制历史记录只到某个特定版本。然而，请参阅下面关于这种方法的一个重要限制。

```
% git clone -o freebsd -b branch --depth 1 $URL [dir]
```

这会克隆仓库，但仅包含仓库中最新的版本。其余历史记录不会被下载。如果以后改变主意，可以执行 git fetch --unshallow 以获取旧的历史记录。

|     | 当您进行浅克隆时，您将失去 uname 输出中的提交计数。这可能会使得在发布安全通告时更难确定系统是否需要更新。 |
| --- | --------------------------------------------------------------------------------------------------------- |

##### 5.2.3.5. 建筑

一旦你下载了，建筑就像手册中描述的那样进行，例如：

```
% cd src
% make buildworld
% make buildkernel
% make installkernel
% make installworld
```

所以这里不会深入讨论。

如果您想构建定制内核，FreeBSD 手册的内核配置部分建议在 sys/${ARCH}/conf 目录下创建一个名为 MYKERNEL 的文件，并根据 GENERIC 进行修改。要使 Git 忽略 MYKERNEL 文件，可以将其添加到.git/info/exclude 文件中。

##### 5.2.3.6. 更新

更新两种类型的树使用相同的命令。这将拉取自上次更新以来的所有修订版本。

```
% git pull --ff-only
```

更新树。在 Git 中，“快进”合并是只需要设置一个新的分支指针，而不需要重新创建提交的一种合并。始终执行快进合并/拉取，将确保您拥有 FreeBSD 树的精确副本。如果您想要保持本地补丁，则这一点非常重要。

请参阅下文以了解如何管理本地更改。最简单的方法是在 git pull 命令上使用 --autostash ，但还有更复杂的选项。

#### 5.2.4. 选择特定版本

在 Git 中， git checkout 可以同时检出分支和特定版本。 Git 的版本是长哈希值，而不是顺序号。

当你检出特定版本时，只需在命令行中指定你想要的哈希值（git log 命令可以帮助你决定你可能想要哪个哈希值）：

```
% git checkout 08b8197a74
```

然后你就检出了那个版本。你将收到类似以下消息的提示：

```
Note: checking out '08b8197a742a96964d2924391bf9fdfeb788865d'.

You are in a 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 08b8197a742a hook gpiokeys.4 to the build
```

最后一行是从您正在检查的哈希生成的，以及该修订版的提交消息的第一行。哈希可以缩写为最短的唯一长度。 Git 本身在显示多少位数方面并不一致。

#### 5.2.5. 二分查找

有时候，事情会出错。上一个版本可用，但您刚刚更新的版本却不行。开发人员可能会要求您对问题进行二分查找，以追踪导致回归的提交。

使用强大的 git bisect 命令，Git 可以轻松进行变更二分查找。以下是如何使用它的简要概述。更多信息，请参阅 https://www.metaltoad.com/blog/beginners-guide-git-bisect-process-elimination 或 https://git-scm.com/docs/git-bisect。man git-bisect 页面介绍了可能出现的问题，版本构建失败时的处理方法，以及在使用除了 'good' 和 'bad' 之外的术语时该怎么办，这里不涉及这些内容。

git bisect start --first-parent 将启动二分查找过程。接下来，需要告诉它要遍历的范围。 git bisect good XXXXXX 将告诉它工作版本， git bisect bad XXXXX 将告诉它坏版本。坏版本几乎总是指 HEAD（一个特殊标签，表示当前检出的内容）。好版本将是您最后检出的那个版本。 --first-parent 参数是必需的，以防后续的 git bisect 命令尝试检出一个不包含完整 FreeBSD 源代码树的供应商分支。

```
5ef0bd68b515 (HEAD -> main, freebsd/main, freebsd/HEAD) HEAD@{0}: pull --ff-only: Fast-forward
a8163e165c5b (upstream/main) HEAD@{1}: checkout: moving from b6fb97efb682994f59b21fe4efb3fcfc0e5b9eeb to main
...
```

显示我将工作树移动到 main 分支（a816…），然后从上游更新（到 5ef0…）。在这种情况下，坏版本将是 HEAD（或者 5ef0bd68b515），好版本将是 a8163e165c5b。从输出中可以看到，HEAD@{1} 通常也有效，但如果在更新后、发现需要进行二分查找之前对 Git 树进行了其他操作，则不一定能正常工作。

先设置“好”的版本，然后设置坏的（顺序无关紧要）。当你设置坏的版本时，它会给你一些关于过程的统计信息：

```
% git bisect start --first-parent
% git bisect good a8163e165c5b
% git bisect bad HEAD
Bisecting: 1722 revisions left to test after this (roughly 11 steps)
[c427b3158fd8225f6afc09e7e6f62326f9e4de7e] Fixup r361997 by balancing parens.  Duh.
```

然后你会构建/安装该版本。如果它是好的，你会输入 git bisect good ，否则输入 git bisect bad 。如果版本无法编译，请输入 git bisect skip 。每一步之后你都会收到类似上面的消息。当你完成后，将坏的版本报告给开发者（或者自己修复 bug 并发送补丁）。 git bisect reset 将结束过程并将你带回原来的位置（通常是 main 的技巧）。再次提醒，git-bisect 手册（上面链接）是处理出错或异常情况的好资源。

#### 5.2.6. 用 GnuPG 签署提交、标签和推送

Git 知道如何签署提交、标签和推送。当您签署 Git 提交或标签时，您可以证明提交的代码来自您，并且在传输过程中没有被更改。您还可以证明提交代码的是您而不是其他人。

关于签署提交和标签的更深入文档可以在 Git 工具 - 签署您的工作 章节中找到，详见 Git 的书。

签署推送背后的理由可以在引入该功能的提交中找到。

最简单的方法是告诉 Git，你希望始终对提交、标签和推送进行签名。可以通过设置几个配置变量来实现这一点：

```
% git config --add user.signingKey LONG-KEY-ID
% git config --add commit.gpgSign true
% git config --add tag.gpgSign true
% git config --add push.gpgSign if-asked
```

|     | 为避免可能的冲突，请确保为 Git 提供一个长的密钥 ID。你可以通过以下方式获取长 ID： gpg --list-secret-keys --keyid-format LONG 。 |
| --- | ------------------------------------------------------------------------------------------------------------------------------- |

|     | 若要使用特定的子密钥，并且不让 GnuPG 将子密钥解析为主密钥，请将 ! 附加到密钥上。例如，要为子密钥 DEADBEEF 加密，使用 DEADBEEF! 。 |
| --- | --------------------------------------------------------------------------------------------------------------------------------- |

##### 5.2.6.1. 验证签名

提交签名可以通过运行 git verify-commit <commit hash> 或 git log --show-signature 来验证。

标签签名可以通过 git verify-tag <tag name> 或 git tag -v <tag name> 来验证。

#### 5.2.7. Ports 考虑因素

ports 树的操作方式相同。分支名称不同，而存储库位于不同位置。

用于 Web 浏览器的 cgit 存储库 Web 接口位于 https://cgit.FreeBSD.org/ports/ 。生产 Git 存储库位于 https://git.FreeBSD.org/ports.git 和 ssh://anongit@git.FreeBSD.org/ports.git（或者 anongit@git.FreeBSD.org:ports.git）。

GitHub 上也有一个镜像，详见外部镜像概述。最新分支是 main 。季度分支命名为 yyyyQn ，其中年为 'yyyy'，季度为 'n'。

##### 5.2.7.1. 提交信息格式

在 ports 仓库中有一个 hook 可以帮助你编写提交信息，位于 .hooks/prepare-commit-message。可以通过运行 git config --add core.hooksPath .hooks 来启用。

主要观点是提交消息应按以下方式格式化：

```
category/port: Summary.

Description of why the changes where made.

PR:	    12345
```

|     | 第一行是提交的主题，包含了 port 的变更内容和提交的摘要。应不超过 50 个字符。<br /><br />应使用空行将其与提交消息的其余部分分隔开。<br /><br />提交消息的其余部分应在 72 个字符边界处换行。<br /><br />如果存在任何元数据字段，应添加另一空行，以便与提交消息清晰区分。 |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### 5.2.8. 管理本地更改

本部分介绍了跟踪本地变更。如果您没有本地变更，可以跳过本部分。

对于所有这些项目都很重要的一点是：所有更改都是本地的，直到推送。与 Subversion 不同，Git 使用分布式模型。对用户来说，在大多数情况下，几乎没有什么区别。但是，如果您有本地更改，您可以使用相同的工具来管理它们，就像您用来从 FreeBSD 中拉取更改一样。您尚未推送的所有更改都是本地的，可以轻松修改（git rebase，下文将介绍此功能）。

##### 5.2.8.1. 保持本地更改

保留本地更改（尤其是琐碎的更改）的最简单方法是使用 git stash 。在其最简单的形式中，您可以使用 git stash 来记录更改（将其推入存储堆栈）。大多数人在更新树之前保存更改时使用这个方法。然后他们使用 git stash apply 将它们重新应用到树上。存储堆栈是一堆可以使用 git stash list 检查的更改。git-stash 手册（ https://git-scm.com/docs/git-stash）中有所有详细信息。

当您对树进行微小调整时，此方法很合适。当您有任何非琐碎的内容时，最好保留本地分支并进行变基。存储还与 git pull 命令集成：只需将 --autostash 添加到命令行即可。

##### 5.2.8.2. 保留本地分支

相对于 Subversion，在 Git 中保持本地分支要容易得多。在 Subversion 中，您需要合并提交并解决冲突。这是可以管理的，但可能会导致一段混乱的历史，如果有必要的话，要推送到上游可能会很困难，或者如果需要复制，则很难复制。 Git 也允许人们合并，但会遇到相同的问题。这是管理分支的一种方式，但它是最不灵活的。

除了合并之外，Git 还支持“变基”的概念，避免了这些问题。 git rebase 命令会在父分支的较新位置重放分支的所有提交。我们将涵盖使用它时出现的最常见情况。

###### 5.2.8.2.1. 创建一个分支

假设你想要修改 FreeBSD 的 ls 命令，使其永远不使用颜色。有很多原因这样做，但本例将使用此作为基线。FreeBSD 的 ls 命令会时不时地改变，你需要应对这些变化。幸运的是，使用 Git rebase 通常是自动的。

```
% cd src
% git checkout main
% git checkout -b no-color-ls
% cd bin/ls
% vi ls.c     # hack the changes in
% git diff    # check the changes
diff --git a/bin/ls/ls.c b/bin/ls/ls.c
index 7378268867ef..cfc3f4342531 100644
--- a/bin/ls/ls.c
+++ b/bin/ls/ls.c
@@ -66,6 +66,7 @@ __FBSDID("$FreeBSD$");
 #include <stdlib.h>
 #include <string.h>
 #include <unistd.h>
+#undef COLORLS
 #ifdef COLORLS
 #include <termcap.h>
 #include <signal.h>
% # these look good, make the commit...
% git commit ls.c
```

提交会让你进入编辑器描述你所做的更改。一旦输入后，你就在 Git 仓库中有了自己的本地分支。按照手册中的说明，像往常一样构建并安装它。Git 不同于其他版本控制系统，你必须明确告诉它要提交哪些文件。我选择在提交命令行上完成此操作，但你也可以使用 git add 来完成，许多更深入的教程中都有介绍。

###### 5.2.8.2.2. 是时候更新了

当该时机到来时，带入新版本几乎与没有分支时相同。您会像上面一样更新，但在更新之前有一条额外的命令，并且之后也有一条命令。以下假定您从未修改过的目录开始。使用干净目录开始重播操作很重要（Git 要求这样）。

```
% git checkout main
% git pull --ff-only
% git rebase -i main no-color-ls
```

这将打开一个列出其中所有提交的编辑器。对于此示例，根本不要更改它。这通常是在更新基线时所做的事情（尽管您还使用 Git rebase 命令来筛选分支中的提交）。

完成上述操作后，您必须将 ls.c 中的提交从 FreeBSD 的旧版本移到更新的版本。

有时会出现合并冲突。这没关系。不要惊慌。相反，处理它们与处理其他任何合并冲突一样。为了简单起见，我将仅描述可能出现的常见问题。完整处理方案的指针可以在本节末找到。

假设包括了上游的更改，采用了 terminfo 的根本转变以及选项名称的更改。在你更新时，可能会看到类似这样的内容：

```
Auto-merging bin/ls/ls.c
CONFLICT (content): Merge conflict in bin/ls/ls.c
error: could not apply 646e0f9cda11... no color ls
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 646e0f9cda11... no color ls
```

看起来有些吓人。如果你打开编辑器，你会看到它是一种典型的三路合并冲突解决方案，你可能从其他源代码系统中熟悉（ls.c 的其余部分已被省略）：

```
 <<<<<<< HEAD
 #ifdef COLORLS_NEW
 #include <terminfo.h>
 =======
 #undef COLORLS
 #ifdef COLORLS
 #include <termcap.h>
 >>>>>>> 646e0f9cda11... no color ls
....
The new code is first, and your code is second.
The right fix here is to just add a #undef COLORLS_NEW before #ifdef and then delete the old changes:
[source,shell]
....
#undef COLORLS_NEW
#ifdef COLORLS_NEW
#include <terminfo.h>
....
save the file.
The rebase was interrupted, so you have to complete it:
[source,shell]
....
% git add ls.c
% git rebase --continue
....
```

告诉 Git ls.c 已经被修复，并继续 rebase 操作。由于发生冲突，您将被踢进编辑器以更新必要的提交消息。如果提交消息仍然准确，只需退出编辑器。

如果在 rebase 过程中遇到困难，请不要惊慌。git rebase --abort 将使您回到干净的状态。但是，重要的是要从未修改的树开始。另外：上述提到的 git reflog 在这里非常方便，因为它将列出您可以查看、检查或挑选的所有（中间）提交。

有关此主题的更多信息，请访问 https://www.freecodecamp.org/news/the-ultimate-guide-to-git-merge-and-git-rebase/。这是一个很好的资源，用于偶尔出现但对本指南来说太过隐晦的问题。

##### 5.2.8.3. 转换到不同的 FreeBSD 分支

如果你想从 stable/12 切换到当前分支。如果你有一个深度克隆，以下内容就足够了:

```
% git checkout main
% # build and install here...
```

如果你有一个本地分支，那么有一两个注意事项。首先，rebase 将重写历史，所以你可能想要做一些保存它的事情。其次，跳转分支往往会引起更多的冲突。如果我们假装上面的示例是相对于 stable/12，那么要移动到 main ，我建议以下操作:

```
% git checkout no-color-ls
% git checkout -b no-color-ls-stable-12   # create another name for this branch
% git rebase -i stable/12 no-color-ls --onto main
```

以上操作的作用是检出 no-color-ls。然后为其创建一个新名称 (no-color-ls-stable-12)，以防需要返回到它。然后你在 main 分支上变基。这将找到当前 no-color-ls 分支的所有提交（直到它与 stable/12 分支汇合的地方），然后将它们重新应用到 main 分支上，在那里创建一个新的 no-color-ls 分支（这就是为什么我让你创建了一个占位符名称）。

### 5.3. MFC（从当前合并）流程

#### 5.3.1. 摘要

MFC 工作流程可以总结为 git cherry-pick -x 加 git commit --amend 来调整提交消息。对于多个提交，请使用 git rebase -i 将它们合并并编辑提交消息。

#### 5.3.2. 单个提交 MFC

```
% git checkout stable/X
% git cherry-pick -x $HASH --edit
```

对于 MFC 提交，例如供应商导入，您需要为挑选樱桃指定一个父提交。通常，这将是您从中挑选樱桃的分支的“第一个父提交”。

```
% git checkout stable/X
% git cherry-pick -x $HASH -m 1 --edit
```

如果出现问题，你需要使用 git cherry-pick --abort 中止 cherry-pick 或修复它并执行 git cherry-pick --continue 。

待 cherry-pick 完成，用 git push 推送。如果由于丢失提交竞争而出错，请使用 git pull --rebase 并再次尝试推送。

#### 5.3.3. MFC 到 RELENG 分支

需要批准的分支的 MFCs 需要更多的注意。无论是典型的合并还是特殊的直接提交，流程都是相同的。

- 在合并或直接提交到相应的 stable/X 分支之前，先合并到 releng/X.Y 分支。
- 为 MFC 到 releng/X.Y 分支使用 stable/X 分支中的哈希。
- 在提交消息中保留“从中挑选的樱桃”行。
- 当您在编辑器中时，请确保添加 Approved by: 行。

```
% git checkout releng/13.0
% git cherry-pick -x $HASH --edit
```

如果您忘记添加 Approved by: 行，您可以执行 git commit --amend 来编辑提交消息，然后再推送更改。

#### 5.3.4. 多次提交 MFC

```
% git checkout -b tmp-branch stable/X
% for h in $HASH_LIST; do git cherry-pick -x $h; done
% git rebase -i stable/X
# mark each of the commits after the first as 'squash'
# Update the commit message to reflect all elements of commit, if necessary.
# Be sure to retain the "cherry picked from" lines.
% git push freebsd HEAD:stable/X
```

如果推送失败是因为输掉了提交竞赛，重新操作再试一次：

```
% git checkout stable/X
% git pull
% git checkout tmp-branch
% git rebase stable/X
% git push freebsd HEAD:stable/X
```

待 MFC 完成，您可以删除临时分支：

```
% git checkout stable/X
% git branch -d tmp-branch
```

#### 5.3.5. MFC 供应商导入

供应商导入是树中唯一创建合并提交的内容，会在 main 分支中创建一个合并提交。将合并提交挑选到 stable/XX 分支中会增加额外的难度，因为合并提交有两个父提交。通常，你会想要使用第一个父提交的差异，因为那是应用于 main 的差异（尽管可能有一些例外情况）。

```
% git cherry-pick -x -m 1 $HASH
```

通常这是你想要的。这会告诉 cherry-pick 应用正确的差异。

有些情况可能是由于转换脚本将 main 分支向后合并导致的，希望这种情况很少发生。如果是这种情况（目前我们还没有找到任何这样的情况），您需要将上述更改为 -m 2 ，以选择正确的父级。只需执行：

```
% git cherry-pick --abort
% git cherry-pick -x -m 2 $HASH
```

要做到这一点。 --abort 将清理失败的第一次尝试。

#### 5.3.6. 重新进行 MFC

如果你做了 MFC，结果出了大问题并且你想重新开始，那么最简单的方法是使用 git reset --hard ，如下所示：

```
% git reset --hard freebsd/stable/12
```

如果你有一些想保留的修订版和一些不想保留的，使用 git rebase -i 更好。

#### 5.3.7. MFC 时的注意事项

当提交源代码到稳定分支和发布工程分支时，我们有以下目标：

- 明确标记直接提交，与从其他分支引入变更的提交区分开。
- 避免在稳定分支和发布工程分支引入已知的破坏性变更。
- 允许开发人员确定哪些更改已经或尚未从一个分支到另一个分支。

对于以上目标，我们使用以下做法：

- 使用 MFC 和 MFS 标签来标记合并自另一个分支的更改。
- 当合并更改时，将 Squashing fixup 提交合并到主提交中。
- 记录合并信息，使 svn mergeinfo --show-revs 起作用。

使用 Git 时，我们将需要使用不同的策略来实现相同的目标。本文旨在定义在使用 Git 合并源提交时实现这些目标的最佳实践。总的来说，我们旨在使用 Git 的本机支持来实现这些目标，而不是强制执行建立在 Subversion 模型之上的实践。

一般注意事项：由于与 Git 的技术差异，我们不会在 stable 或 releng 分支中使用 Git “合并提交”（通过 git merge 创建）。相反，当本文档提到“合并提交”时，是指最初在 main 中进行的提交被复制或“落地”到 stable 分支，或从 stable 分支的提交被以某种 git cherry-pick 变体复制到 releng 分支。

#### 5.3.8. 查找符合 MFC 的哈希值

Git 提供了一些内置支持，通过 git cherry 和 git log --cherry 命令。这些命令比较提交的原始差异（但不包括日志消息等其他元数据）以确定两个提交是否相同。当每个来自 main 的提交作为单个提交落地到 stable 分支时，这个方法效果很好，但如果来自 main 的多个提交被压缩成一个提交落地到 stable 分支时，这个方法就会失效。项目广泛使用 git cherry-pick -x 并保留所有行来解决这些困难，并正在开发自动化工具来利用这一点。

#### 5.3.9. 提交信息标准

##### 5.3.9.1. 标记 MFCs

该项目采纳了以下标记 MFCs 的做法:

- 使用 -x 标志与 git cherry-pick . 这会在合并时向提交消息添加一行，其中包含原始提交的哈希。由于它是由 Git 直接添加的，因此提交者在合并时无需手动编辑提交日志。

在合并多个提交时，请保留所有“cherry picked from”行。

##### 5.3.9.2. 裁剪元数据？

对于 Subversion（甚至是 CVS），文档中没有清楚说明如何为 MFC 提交格式化日志消息中的元数据。应该保留原始提交中的元数据不变，还是应该修改以反映 MFC 提交本身的信息？

历史做法有所不同，尽管某些差异是由领域决定的。例如，与 PR 相关的 MFC 通常在 MFC 中包含 PR 字段，以便将 MFC 提交包括在 bug 跟踪器的审计轨迹中。其他字段则不太明确。例如，Phabricator 显示最后一个标记为审查的提交的差异，因此包括 Phabricator URL 会用已落地的提交替换主要提交。审阅人员列表也不太明确。如果审阅人员已经批准了对 main 的更改，那么是否意味着他们已经批准了 MFC 提交？如果只是相同的代码，或者仅有微小的修改呢？对于更广泛的重写显然不成立。即使对于相同的代码，如果提交没有冲突但引入了 ABI 更改呢？审阅人员可能会因为 ABI 断裂而批准了 main 的提交，但可能不赞同按原样合并相同的提交。在缺乏明确指导的情况下，必须凭借自己的判断力。

对于由 re@ 管理的 MFC，会添加新的元数据字段，例如已批准提交的“Approved by”标签。此类新的元数据将在原始提交经过审查和批准后通过 git commit --amend 或类似方式添加。我们还可能希望在 MFC 提交中保留一些元数据字段，如 Phabricator URL，以供 re@ 未来使用。

保留现有元数据提供了非常简单的工作流程。开发者可以使用 git cherry-pick -x 而无需编辑日志消息。

如果我们选择在 MFC 中调整元数据，开发者将不得不通过使用 git cherry-pick --edit 或 git commit --amend 明确地编辑日志消息。然而，与 svn 相比，至少现有的提交消息可以预先填充，并且可以添加或删除元数据字段，而无需重新输入整个提交消息。

底线是，开发者可能需要为非平凡的 MFC 调整他们的提交消息。

### 5.4. 供应商使用 Git 进行导入

本节详细介绍了使用 Git 进行供应商导入的过程。

#### 5.4.1. 分支命名约定

所有供应商分支和标签都以 vendor/ 开头。这些分支和标签默认可见。

|     | 本章遵循的约定是 freebsd 起源是官方 FreeBSD Git 仓库的起源名称。如果你使用不同的约定，请在下面的示例中将 freebsd 替换为你使用的名称。 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------- |

我们将探讨更新我们树中的 NetBSD 的 mtree 的示例。供应商分支是 vendor/NetBSD/mtree 。

#### 更新旧供应商导入

供应商树通常只包含适用于 FreeBSD 的第三方软件的子集。这些树通常与 FreeBSD 树相比较小。因此，Git 工作树非常小且快速，并且是首选的使用方法。确保在下面选择的任何目录（ ../mtree ）目前不存在。

```
% git worktree add ../mtree vendor/NetBSD/mtree
```

#### 更新供应商分支中的源码

准备供应商源代码的完整、干净树。导入全部内容，但只合并需要的部分。

该示例假设 NetBSD 源代码是从它们在 GitHub 上的镜像处检出的 ~/git/NetBSD 。请注意，“上游”可能已添加或删除文件，因此我们希望确保删除也能传播。 net/rsync 通常已安装，因此我将使用它。

```
% cd ../mtree
% rsync -va --del --exclude=".git" ~/git/NetBSD/usr.sbin/mtree/ .
% git add -A
% git status
...
% git diff --staged
...
% git commit -m "Vendor import of NetBSD's mtree at 2020-12-11"
[vendor/NetBSD/mtree 8e7aa25fcf1] Vendor import of NetBSD's mtree at 2020-12-11
 7 files changed, 114 insertions(+), 82 deletions(-)
% git tag -a vendor/NetBSD/mtree/20201211
```

非常重要的是要验证您导入的源代码来自可信任的来源。许多开源项目使用密码签名来签署代码更改、git 标签和/或源代码压缩包。始终验证这些签名，并使用隔离机制如 jail、chroot，结合一个与您通常使用的不同的专用、非特权用户帐户（有关更多详细信息，请参阅下面的更新 FreeBSD 源代码树部分），直到您确信导入的源代码看起来是安全的。跟踪上游开发并偶尔审查上游代码更改可以极大地帮助改进代码质量并使所有相关方受益。在将它们导入供应商区域之前检查 git diff 结果也是一个好主意。

始终运行 git diff 和 git status 命令，并仔细检查结果。如果有疑问，可以在供应商分支或上游 git 存储库上执行 git annotate 以查看谁以及为什么做出了更改。

在上面的示例中，我们使用 -m 进行说明，但您应该在编辑器中组成一个适当的消息（使用提交消息模板）。

创建一个带有 git tag -a 的注释标签也很重要，否则推送将被拒绝。只有允许推送带注释的标签。注释标签让您有机会输入提交消息。输入您正在导入的版本，以及该版本中的任何显着新功能或修复。

#### 5.4.4. 更新 FreeBSD 副本

此时，您可以将导入推送到我们的存储库中。

```
% git push --follow-tags freebsd vendor/NetBSD/mtree
```

--follow-tags 告诉 git push 也推送与本地提交的修订版本关联的标记。

#### 5.4.5. 更新 FreeBSD 源代码树

现在你需要更新 FreeBSD 中的 mtree。这些源代码存放在 contrib/mtree 中，因为它是上游软件。

有时候，我们可能需要对贡献的代码进行修改，以更好地满足 FreeBSD 的需求。在可能的情况下，请尝试将本地的修改贡献回上游项目，这有助于它们更好地支持 FreeBSD，也能节省未来导入更新时冲突解决的时间。

```
% cd ../src
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree
```

这将生成一个 contrib/mtree 的子树合并提交到本地 vendor/NetBSD/mtree 分支。检查合并结果的差异和上游分支的内容。如果合并将我们的本地更改减少到像空行或缩进变化这样的琐碎差异，请尝试修改本地更改以减少与上游的差异，或者尝试将剩余的更改贡献回上游项目。如果有冲突，您需要在提交前解决它们。在合并提交消息中包含有关合并更改的详细信息。

一些开源软件包含一个 configure 脚本，用于生成定义代码如何构建的文件；通常，这些生成的文件如 config.h 应在导入过程中进行更新。执行此操作时，请始终记住这些脚本是在当前用户的凭据下运行的可执行代码。此过程应始终在一个隔离的环境中运行，理想情况下是在没有网络访问权限的 jail 内，并使用非特权帐户；或者至少使用一个专用帐户，该帐户不同于您通常用于日常工作或推送到 FreeBSD 源代码库的用户帐户。这将最大限度地降低遇到可能导致数据丢失的 bug 或更严重情况下恶意植入代码的风险。使用隔离的 jail 还可以防止配置脚本检测到本地安装的软件包，这可能导致意外结果。

测试更改时，请首先在 chroot 或 jail 环境中运行它们，甚至在虚拟机中运行，尤其是对于内核或库的修改。这种方法有助于防止对您的工作环境产生不利影响。它对许多基本系统组件使用的库的更改特别有益。

#### 5.4.6. 将您的更改重新对齐到最新的 FreeBSD 源代码树

因为当前政策建议不使用合并，如果上游 FreeBSD main 在您有机会推送之前进展，您将不得不重新做合并。

常规 git rebase 或 git pull --rebase 不知道如何将合并提交重新对齐为合并提交，因此您将不得不重新创建提交。

以下步骤应该被采取以便轻松地重新创建合并提交，就好像 git rebase --merge-commits 正常工作一样：

- 切换到仓库顶层目录
- 使用合并树的内容创建一个名为 XXX 的侧边分支。
- 更新此侧分支 XXX ，并确保与 FreeBSD 的 main 分支合并并保持最新状态。

  - 在最坏的情况下，如果有任何冲突，您仍将不得不解决合并冲突，但这应该是非常罕见的。
  - 解决冲突，并将多个提交合并成一个（如果没有冲突，则不需要合并）。

- 结账 main
- 创建分支 YYY （如果出现问题，更容易回退）
- 重新做子树合并
- 与其解决子树合并中的任何冲突，不如将 XXX 的内容检出到其顶部。

  - 尾随的 . 很重要，就像处于存储库顶层一样重要。
  - 与其切换到 XXX 分支，不如将 XXX 的内容直接放在存储库的顶部。

- 将结果提交到先前的提交消息中（本示例假设 XXX 分支只有一个合并）。
- 确保分支相同。
- 根据需要进行审查，包括让其他人查看，如果你认为有必要的话。
- 推送提交，如果您再次“输掉比赛”，只需重新执行这些步骤（请参阅下面的配方）
- 一旦提交到上游，删除分支。它们是一次性的。

按照上面 mtree 的示例，一个人会使用的命令就像这样（ # 开头的注释，以帮助将命令链接到上面的描述）:

```
% cd ../src			# CD to top of tree
% git checkout -b XXX		# create new throw-away XXX branch for merge
% git fetch freebsd		# Get changes from upstream from upstream
% git merge freebsd/main	# Merge the changes and resolve conflicts
% git checkout -b YYY freebsd/main # Create new throw-away YYY branch for redo
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree # Redo subtree merge
% git checkout XXX .		# XXX branch has the conflict resolution
% git commit -c XXX~1		# -c reuses the commit message from commit before rebase
% git diff XXX YYY		# Should be empty
% git show YYY			# Should only have changes you want, and be a merge commit from vendor branch
```

注意：如果提交出现问题，您可以通过重新发出创建它的带有 -B 参数的检出命令来重置 YYY 分支，以重新开始：

```
% git checkout -B YYY freebsd/main # Create new throw-away YYY branch if starting over is just going to be easier
```

#### 5.4.7. 推送更改

一旦您认为您有一组良好的更改，您可以将其推送到 GitHub 或 GitLab 上的分叉，供他人审查。Git 的一个好处是，它允许您发布您工作的草稿版本供他人审查。尽管 Phabricator 适用于内容审查，但发布更新的供应商分支和合并提交可以让其他人检查细节，这些细节最终将出现在仓库中。

审查后，当您确信这是一个好的更改时，您可以将其推送到 FreeBSD 存储库：

```
% git push freebsd YYY:main	# put the commit on upstream's 'main' branch
% git branch -D XXX		# Throw away the throw-a-way branches.
% git branch -D YYY
```

注意：我使用 XXX 和 YYY 来表明它们是糟糕的名称，不应该离开您的机器。如果您将这些名称用于其他工作，那么您需要选择不同的名称，否则会丢失其他工作。这些名称并没有什么特别之处。上游不会允许您推送它们，但请注意上述确切命令。一些命令的语法与典型用法略有不同，不同的行为对于此操作的成功至关重要。

#### 5.4.8. 需要时如何重做事情

如果您尝试在上一节中进行推送，但失败了，那么您应该按照以下步骤进行“重做”操作。这个顺序始终将提交与提交消息保留在 XXX~1，以便更轻松地进行提交。

```
% git checkout -B XXX YYY	# recreate that throw-away-branch XXX and switch to it
% git merge freebsd/main	# Merge the changes and resolve conflicts
% git checkout -B YYY freebsd/main # Recreate new throw-away YYY branch for redo
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree # Redo subtree merge
% git checkout XXX .		# XXX branch has the conflict resolution
% git commit -c XXX~1		# -c reuses the commit message from commit before rebase
```

然后，当准备就绪时，像上面那样检出并推送。

### 5.5. 创建一个新的供应商分支

创建新供应商分支的方法有许多种。推荐的方法是创建一个新的存储库，然后将其与 FreeBSD 合并。如果要将 glorbnitz 导入 FreeBSD 树，则发布 3.1415 版本。为了简单起见，我们不会修剪此版本。这是一个简单的用户命令，将尼茨设备置于不同的神奇格罗布状态，并且足够小，修剪不会节省多少。

#### 5.5.1. 创建存储库

```
% cd /some/where
% mkdir glorbnitz
% cd glorbnitz
% git init
% git checkout -b vendor/glorbnitz
```

此时，您拥有一个新的存储库，所有新的提交都将放在 vendor/glorbnitz 分支上。

Git 专家也可以在他们的 FreeBSD 复刻中使用 git checkout --orphan vendor/glorbnitz 来完成此操作，如果他们更习惯这样做的话。

#### 5.5.2. 复制源代码到

由于这是一个新的导入，您可以直接 cp 源代码，或者使用 tar 甚至 rsync，如上所示。我们将添加所有内容，假设没有点文件。

```
% cp -r ~/glorbnitz/* .
% git add *
```

在这一点上，您应该有一个准备提交的 glorbnitz 的原始副本。

```
% git commit -m "Import GlorbNitz frobnosticator revision 3.1415"
```

如上所述，我为简单起见使用 -m ，但您应该为什么使用 Nitz 获取它以及 Glorb 是什么而创建一个提交消息。并非每个人都会知道，因此在实际提交时，您应该遵循提交日志消息部分，而不是仿效此处使用的简要样式。

#### 5.5.3. 现在将其导入我们的代码库

现在您需要将分支导入我们的代码库。

```
% cd /path/to/freebsd/repo/src
% git remote add glorbnitz /some/where/glorbnitz
% git fetch glorbnitz vendor/glorbnitz
```

注意，vendor/glorbnitz 分支已经在仓库中。此时，如果您愿意，可以删除 /some/where/glorbnitz 。这只是一个手段。

#### 5.5.4. 标记并推送

从这里开始，步骤基本与更新供应商分支的情况相同，只是没有更新供应商分支的步骤。

```
% git worktree add ../glorbnitz vendor/glorbnitz
% cd ../glorbnitz
% git tag --annotate vendor/glorbnitz/3.1415
# Make sure the commit is good with "git show"
% git push --follow-tags freebsd vendor/glorbnitz
```

我们所说的“好”的意思是：

1. 所有正确的文件都存在
2. 没有任何错误的文件存在
3. 供应商分支指向合理的内容
4. 标签看起来很好，并且有注释
5. 标记的提交消息中包含自上次标记以来的新内容的快速总结

#### 5.5.5. 是时候将其最终合并到基本树中

```
% cd ../src
% git subtree add -P contrib/glorbnitz vendor/glorbnitz
# Make sure the commit is good with "git show"
% git commit --amend   # one last sanity check on commit message
% git push freebsd
```

这里的“好”意味着：

1. 所有正确的文件，而且没有错误的文件，都被合并到 contrib/glorbnitz 中。
2. 树上没有其他更改。
3. 提交消息看起来不错。它应该包含自上次合并到 FreeBSD main 分支以来发生的变化的摘要和任何注意事项。
4. 更新应在有值得注意的内容时更新，比如用户可见的变化，重要的升级问题等。

|     | 这还没有将 glorbnitz 连接到构建中。如何做到这一点取决于导入软件的具体情况，超出了本教程的范围。 |
| --- | ----------------------------------------------------------------------------------------------- |

##### 5.5.5.1. 保持当前

所以，时间流逝。现在是时候更新树以获取最新的上游更改了。当你签出 main 时，确保没有差异。在进行以下操作之前，将这些提交到一个分支（或使用 git stash ）会容易得多。

如果你习惯于 git pull ，我们强烈建议使用 --ff-only 选项，并进一步将其设置为默认选项。或者，如果你在 main 分支中有暂存的更改， git pull --rebase 会很有用。

```
% git config --global pull.ff only
```

如果你希望此设置仅应用于这个存储库，可能需要省略 --global。

```
% cd freebsd-src
% git checkout main
% git pull (--ff-only|--rebase)
```

存在一个常见的陷阱，即组合命令 git pull 尝试执行合并，有时会创建一个之前不存在的合并提交。这样做可能更难恢复。

推荐使用更长的形式。

```
% cd freebsd-src
% git checkout main
% git fetch freebsd
% git merge --ff-only freebsd/main
```

这些命令将您的树重置为 main 分支，然后从最初拉取树的位置更新它。在执行此操作之前切换到 main 是很重要的，这样可以使变更向前移动。现在，是时候将变更推进了：

```
% git rebase -i main working
```

这将带出一个交互式屏幕，用于更改默认设置。现在，只需退出编辑器。一切都应该生效。如果没有，那么您将需要解决差异。这份 github 文档可以帮助您浏览此过程。

##### 推送变更到上游的时间到了

首先，确保推送 URL 已正确配置为上游存储库。

```
% git remote set-url --push freebsd ssh://git@gitrepo.freebsd.org/src.git
```

然后，验证用户名和电子邮件是否正确配置。 我们要求它们与 FreeBSD 集群中的密码条目完全匹配。

使用

```
freefall% gen-gitconfig.sh
```

在 freefall.freebsd.org 上获取一个配方，您可以直接使用，假设/usr/local/bin 在 PATH 中。

下面的命令将 working 分支合并到上游 main 分支中。在执行此操作之前，确保您已经调整好更改，使其与您在 FreeBSD 源代码仓库中想要的一样。此语法将 working 分支推送到 main ，使 main 分支向前推进。只有在这导致对 main 进行线性更改（例如，没有合并）时，您才能执行此操作。

```
% git push freebsd working:main
```

如果您的推送由于丢失提交竞赛而被拒绝，请在再次尝试之前重新设置您的分支：

```
% git checkout working
% git fetch freebsd
% git rebase freebsd/main
% git push freebsd working:main
```

##### 5.5.5.3. 推送更改到上游的时间（备选）

有些人发现在推送到远程存储库之前，将他们的更改合并到本地 main 更容易。此外， git arc stage 在您需要执行分支的子集时，将更改从一个分支移动到本地 main 。指令与前一部分类似：

```
% git checkout main
% git merge --ff-only `working`
% git push freebsd
```

如果你输了比赛，那就再试一次

```
% git pull --rebase
% git push freebsd
```

这些命令将获取最新的 freebsd/main ，然后在其上重新定义本地 main 的更改，这正是你在失去提交比赛时想要的。注意：使用此技术无法合并供应商分支提交。

##### 5.5.5.4. 查找 Subversion 修订版

您需要确保已获取笔记（请参阅详细的日常使用说明）。一旦获取了这些笔记，笔记将像这样显示在 git log 命令中：

```
% git log
```

如果您有特定版本，请使用此结构：

```
% git log --grep revision=XXXX
```

找到特定的修订版。'commit' 后面的十六进制数字是您可以用来引用此提交的哈希值。

### 5.6. Git 常见问题解答

本节为用户和开发人员经常遇到的问题提供了一些有针对性的答案。

|     | 我们通常使用将 FreeBSD 仓库的远程源命名为'freebsd'，而不是默认的'origin'，以便人们可以用于他们自己的开发，并最大程度地减少错误地推送到错误的仓库。 |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------- |

#### 5.6.1. 用户

##### 5.6.1.1. 我如何只用一个仓库跟踪-current 和-stable 版本？

问：虽然磁盘空间不是一个大问题，但使用一个副本的仓库更有效率。使用 SVN 镜像时，我可以从同一个仓库检出多个树。使用 Git 怎么做？

答：你可以使用 Git worktrees。有很多方法可以做到这一点，但最简单的方法是使用一个 clone 来跟踪-current，并使用一个 worktree 来跟踪稳定版本。虽然使用“裸仓库”作为一种应对方式被提出，但它更复杂，这里不做文档记录。

首先，你需要克隆 FreeBSD 仓库，这里展示了克隆到 freebsd-current 以减少混淆。$URL 是任何对你最合适的镜像：

```
% git clone -o freebsd --config remote.freebsd.fetch='+refs/notes/*:refs/notes/*' $URL freebsd-current
```

然后一旦克隆完成，您可以简单地从中创建一个工作树：

```
% cd freebsd-current
% git worktree add ../freebsd-stable-12 stable/12
```

这将会将 stable/12 检出到一个名为 freebsd-stable-12 的目录中，与 freebsd-current 目录处于同一级。一旦创建，更新方式与您预期的非常相似：

```
% cd freebsd-current
% git checkout main
% git pull --ff-only
# changes from upstream now local and current tree updated
% cd ../freebsd-stable-12
% git merge --ff-only freebsd/stable/12
# now your stable/12 is up to date too
```

我建议使用 --ff-only ，因为这样更安全，可以避免意外陷入“合并噩梦”，在您的树中多出一个变更，迫使进行复杂的合并而不是简单的合并。

这是一个更详细的好写作。

#### 5.6.2. 开发人员

##### 5.6.2.1. 糟糕！我提交到 main ，而不是另一个分支。

Q: 时不时我会搞砸，错误地提交到 main 分支。我该怎么办？

A: 首先，不要慌。

其次，不要推送。事实上，如果还没有推送，几乎可以修复任何问题。本节中的所有答案都假设尚未推送。

假定您已经提交了 main ，并且想要创建一个名为 issue 的分支：

```
% git branch issue                # Create the 'issue' branch
% git reset --hard freebsd/main   # Reset 'main' back to the official tip
% git checkout issue              # Back to where you were
```

##### 5.6.2.2. 哎呀！我把东西提交到了错误的分支！

问：我正在 wilma 分支上开发功能，但不小心在'wilma'中提交了与 fred 分支相关的更改。我该怎么办？

A：答案与之前的相似，但是要挑出优势。这假设 wilma 上只有一个提交，但可以推广到更复杂的情况。它还假设这是 wilma 上的最后一个提交（因此在 git cherry-pick 命令中使用 wilma），但这也可以概括。

```
# We're on branch wilma
% git checkout fred		# move to fred branch
% git cherry-pick wilma		# copy the misplaced commit
% git checkout wilma		# go back to wilma branch
% git reset --hard HEAD^	# move what wilma refers to back 1 commit
```

Git 专家们会先将 wilma 分支倒退 1 次提交，切换到 fred，然后使用 git reflog 来查看删除的那一个提交是什么，然后挑选出来。

Q：但是如果我想提交一些更改到 main ，但出于某些原因保留其余更改在 wilma 中怎么办？

答：如果你想在 main 中“放置”你正在处理的分支的部分内容，而在分支的其他部分尚未准备好之前（比如你发现了一个无关的错别字，或者修复了一个偶然的 bug），上述相同的技巧也适用。你可以将这些更改 cherry pick 到 main ，然后推送到父仓库。一旦你完成了，清理就再简单不过了：只需 git rebase -i 。Git 会注意到你已经做了这个，并自动跳过共同的更改（即使你不得不更改提交信息或稍微调整提交）。不需要切回到 wilma 进行调整：只需 rebase！

问：我想将分支 wilma 的一些更改拆分到分支 fred 中

答：更一般的答案与之前相同。你会检出/创建 fred 分支，从 wilma 中逐一 cherry pick 你想要的更改，然后 rebase wilma 以删除你 cherry pick 的那些更改。 git rebase -i main wilma 会将你带入一个编辑器，并删除与提交到 fred 的提交对应的 pick 行。如果一切顺利，没有冲突，你就完成了。如果有，你需要在进行中解决冲突。

另一种做法是检出 wilma ，然后创建指向树中相同节点的分支 fred 。然后你可以合并这两个分支，在编辑器中保留需要的更改 fred 或者 wilma ，删除其余的。有些人在开始之前会创建一个叫做 pre-split 的标签/分支，以防出现问题。你可以使用以下顺序撤消操作：

```
% git checkout pre-split	# Go back
% git branch -D fred		# delete the fred branch
% git checkout -B wilma		# reset the wilma branch
% git branch -d pre-split	# Pretend it didn't happen
```

最后一步是可选的。如果你打算再次分开，可以省略这一步。

问：但我是边阅读边操作的，并没有看到你建议在最后创建分支，现在 fred 和 wilma 都搞砸了。如何找到我开始前的 wilma 是什么。我不知道自己移动了多少次。

A：一切并非绝望。只要时间不长，提交次数不多（数百次），你可以弄清楚这些。

因此，我创建了一个 wilma 分支，并向其提交了一些内容，然后决定将其拆分为 fred 和 wilma。当我这样做时，没有发生任何奇怪的事情，但假设发生了。查看您所做的事情的方法是通过 git reflog ：

```
% git reflog
6ff9c25 (HEAD -> wilma) HEAD@{0}: rebase -i (finish): returning to refs/heads/wilma
6ff9c25 (HEAD -> wilma) HEAD@{1}: rebase -i (start): checkout main
869cbd3 HEAD@{2}: rebase -i (start): checkout wilma
a6a5094 (fred) HEAD@{3}: rebase -i (finish): returning to refs/heads/fred
a6a5094 (fred) HEAD@{4}: rebase -i (pick): Encourage contributions
1ccd109 (freebsd/main, main) HEAD@{5}: rebase -i (start): checkout main
869cbd3 HEAD@{6}: rebase -i (start): checkout fred
869cbd3 HEAD@{7}: checkout: moving from wilma to fred
869cbd3 HEAD@{8}: commit: Encourage contributions
...
%
```

在这里，我们可以看到我所做的更改。您可以使用它来找出问题出在哪里。我只是在这里指出几件事。第一件事是 HEAD@{X}是一个“提交”相关的东西，所以您可以将其用作命令的参数。尽管如果该命令向存储库提交任何内容，则 X 数字会更改。您也可以使用哈希（第一列）。

接下来，'鼓励贡献' 是我在 wilma 上做出的最后一次提交，然后我决定拆分事务。当我创建 fred 分支来完成这个过程时，你也可以看到相同的哈希值。我从重新设置 fred 开始，你可以看到每一步的 '开始' 和 '完成'。虽然我们这里不需要它，但你可以准确地了解到发生了什么。幸运的是，为了修复这个问题，你可以按照之前答案的步骤进行操作，但要用哈希 869cbd3 替换 pre-split 。虽然这看起来有点啰嗦，但你一次只做一件事情，很容易记住。你也可以堆叠：

```
% git checkout -B wilma 869cbd3
% git branch -D fred
```

然后你就可以准备再试一次。使用带有哈希的 checkout -B 结合检出和为其创建分支。而使用 -B 而不是 -b 强制移动一个现有的分支。无论哪种方式都可以，这就是 Git 优缺点所在。我倾向于使用 git checkout -B xxxx hash 而不是检出哈希，然后创建/移动分支，纯粹是为了避免关于分离 HEAD 的稍微令人不安的消息：

```
% git checkout 869cbd3
M	faq.md
Note: checking out '869cbd3'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 869cbd3 Encourage contributions
% git checkout -B wilma
```

这产生了相同的效果，但我不得不读更多的东西，而被切断的 HEAD 不是我喜欢思考的图像。

##### 5.6.2.3. 哎呀！我做了一个 git pull ，导致了一个合并提交，我该怎么办呢？

问：我在自动驾驶模式下对我的开发树做了一个 git pull ，结果在 main 上创建了一个合并提交。我该如何恢复？

答：当您使用您的开发分支检出时，这种情况可能会发生。

拉取之后，你将会检出新的合并提交。Git 支持使用 HEAD^# 语法来检查合并提交的父提交：

```
git log --oneline HEAD^1   # Look at the first parent's commits
git log --oneline HEAD^2   # Look at the second parent's commits
```

从这些日志中，你可以轻松识别哪个提交是你的开发工作。然后你只需将分支重置到相应的 HEAD^# :

```
git reset --hard HEAD^2
```

问：但我还需要修复我的 main 分支。我该怎么做？

Git 将远程存储库分支跟踪在一个 freebsd/ 命名空间中。要修复你的 main 分支，只需使其指向远程的 main ：

```
git branch -f main freebsd/main
```

Git 中的分支并没有什么神奇之处：它们只是图上的标签，通过进行提交自动向前移动。因此，上面的操作有效是因为你只是在移动一个标签。由于这一点，不需要保留关于分支的任何元数据。

##### 5.6.2.4. 混合和匹配分支

问题：我有两个分支 worker 和 async ，想要将它们合并为一个名为 feature 的分支，同时保留两个分支中的提交记录。

答：这就是樱桃拣选的工作。

```
% git checkout worker
% git checkout -b feature	# create a new branch
% git cherry-pick main..async	# bring in the changes
```

您现在有一个名为 feature 的新分支。这个分支合并了两个分支的提交。您可以进一步使用 git rebase 进行精选。

Q: 我有一个名为 driver 的分支，我想将其拆分为 kernel 和 userland ，这样我可以分别演变它们，并在准备好时提交每个分支。

A: 这需要一些准备工作，但 git rebase 会在这里承担主要工作。

```
% git checkout driver		# Checkout the driver
% git checkout -b kernel	# Create kernel branch
% git checkout -b userland	# Create userland branch
```

现在你有两个相同的分支。所以，现在是将提交分离出来的时候了。我们假设所有 driver 中的提交都将进入 kernel 或 userland 分支，但不会同时进入两者。

```
% git rebase -i main kernel
```

并只包括您想要的更改（使用“p”或“pick”行），并删除您不需要的提交（这听起来很可怕，但如果情况变得更糟，您可以将所有内容清除并从 driver 分支重新开始，因为您尚未移动它）。

```
% git rebase -i main userland
```

并做与 kernel 分支相同的事情。

问：哦，太棒了！我遵循上述步骤，但在 kernel 分支中忘记了一个提交。我该如何恢复？

A：您可以使用 driver 分支找到缺失的提交哈希并挑选它。

```
% git checkout kernel
% git log driver
% git cherry-pick $HASH
```

Q：好的。我遇到了与上述相同的情况，但我的提交都混在一起了。我需要将一些提交的部分放在一个分支上，其余的放在另一个分支上。事实上，我有好几个这样的情况。您的变基方法听起来很棘手。

A：在这种情况下，最好是精心整理原始分支以分离提交，然后使用上述方法拆分分支。

所以，让我们假设只有一个具有干净树的提交。您可以使用 git rebase 和 edit 命令行，或者您可以在提交的顶部使用此命令。无论哪种方式，步骤都是一样的。我们需要做的第一件事是备份一个提交，同时保留树中未提交的更改。

```
% git reset HEAD^
```

注意：请不要在此处添加 --hard ，因为这也会从您的树中删除更改。

现在，如果你幸运的话，需要拆分的更改完全沿着文件行进行。在这种情况下，您只需为每个组中的文件执行常规 git add ，然后执行 git commit 。注意：当您执行此操作时，当您执行重置时，将丢失提交消息，因此，如果出于某种原因您需要它，应该保存一份副本（尽管 git log $HASH 可以恢复它）。

如果你不幸运的话，你需要分开文件。还有另外一个工具可以逐个文件应用。

```
git add -i foo/bar.c
```

将逐步检查差异，提示您是否包含或排除每个补丁。完成后， git commit ，剩余的部分将保留在您的树中。您也可以多次运行它，甚至可以跨多个文件操作（虽然我发现逐个文件操作并使用 git rebase -i 将相关提交合并在一起更容易）。

#### 5.6.3. 克隆和镜像

问：我想镜像整个 Git 仓库，我该怎么做？

答：如果你只是想镜像，那么

```
% git clone --mirror $URL
```

可以解决问题。不过，如果你想用它做镜像以外的其他事情，有两个缺点，你需要重新克隆。

首先，这是一个'裸仓库'，其中包含仓库数据库，但没有检出的工作树。这对于镜像是很好的，但日常工作则不适用。有多种方法可以解决这个问题 git worktree ：

```
% git clone --mirror https://git.freebsd.org/ports.git ports.git
% cd ports.git
% git worktree add ../ports main
% git worktree add ../quarterly branches/2020Q4
% cd ../ports
```

但是，如果您不打算将镜像用于进一步的本地克隆，则不太合适。

第二个缺点是，Git 通常会重写来自上游的引用（分支名称、标签等），以便您的本地引用可以独立于上游发展。这意味着，如果您不是在私有项目分支上提交到此仓库，则会丢失更改。

Q: 那么我可以做什么？

A: 嗯，您可以将所有上游仓库的引用存储到本地仓库的私有命名空间中。Git 通过 'refspec' 克隆所有内容，而默认的 refspec 是：

```
        fetch = +refs/heads/*:refs/remotes/freebsd/*
```

它表示只获取分支引用。

然而，FreeBSD 仓库中还有许多其他内容。要查看这些内容，您可以为每个 ref 命名空间添加显式 refspec，或者您可以获取所有内容。要设置您的仓库以执行此操作：

```
git config --add remote.freebsd.fetch '+refs/*:refs/freebsd/*'
```

这将把上游仓库中的所有内容放入您的本地仓库的 refs/freebsd/ 命名空间。请注意，这也会抓取所有未转换的供应商分支，并且与它们相关的 ref 数量相当大。

您需要用它们的全名来引用这些 'refs'，因为它们不在 Git 的常规命名空间内。

```
git log refs/freebsd/vendor/zlib/1.2.10
```

查看 zlib 供应商分支版本从 1.2.10 开始的日志。

### 5.7. 与他人合作

在像 FreeBSD 这样的大型项目上进行良好的软件开发的关键之一是在将更改推送到树之前与他人合作的能力。FreeBSD 项目的 Git 存储库目前不允许将用户创建的分支推送到存储库，因此，如果您希望与他人共享更改，您必须使用另一种机制，例如托管的 GitLab 或 GitHub，以在用户生成的分支中共享更改。

以下说明显示如何设置一个基于 FreeBSD main 分支的用户生成分支，并将其推送到 GitHub。

在开始之前，请确保您的本地 Git 存储库是最新的，并设置了正确的源，如上所示。

```

% git remote -v
freebsd https://git.freebsd.org/src.git (fetch)
freebsd ssh://git@gitrepo.freebsd.org/src.git (push)
```

第一步是根据以下准则在 GitHub 上创建一个 FreeBSD 的分支。分叉的目的地应该是您自己的个人 GitHub 帐户（在我的情况下是 gvnn3）。

现在在本地系统上添加一个指向您的分支的远程:

```
% git remote add github git@github.com:gvnn3/freebsd-src.git
% git remote -v
github	git@github.com:gvnn3/freebsd-src.git (fetch)
github	git@github.com:gvnn3/freebsd-src.git (push)
freebsd	https://git.freebsd.org/src.git (fetch)
freebsd	ssh://git@gitrepo.freebsd.org/src.git (push)
```

有了这个设置，你可以像上面展示的那样创建一个分支。

```
% git checkout -b gnn-pr2001-fix
```

在您的分支中进行任何修改。构建、测试，一旦您准备好与他人合作，就可以将更改推送到托管的分支。在你推送之前，你将需要设置适当的上游，因为 Git 会在你第一次尝试推送到你的 github 远程时告诉你这一点。

```
% git push github
fatal: The current branch gnn-pr2001-fix has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream github gnn-pr2001-fix
```

将推送设置为 git 建议的方式可以使其成功:

```
% git push --set-upstream github gnn-feature
Enumerating objects: 20486, done.
Counting objects: 100% (20486/20486), done.
Delta compression using up to 8 threads
Compressing objects: 100% (12202/12202), done.
Writing objects: 100% (20180/20180), 56.25 MiB | 13.15 MiB/s, done.
Total 20180 (delta 11316), reused 12972 (delta 7770), pack-reused 0
remote: Resolving deltas: 100% (11316/11316), completed with 247 local objects.
remote:
remote: Create a pull request for 'gnn-feature' on GitHub by visiting:
remote:      https://github.com/gvnn3/freebsd-src/pull/new/gnn-feature
remote:
To github.com:gvnn3/freebsd-src.git
 * [new branch]                gnn-feature -> gnn-feature
Branch 'gnn-feature' set up to track remote branch 'gnn-feature' from 'github'.
```

对同一分支进行后续更改将默认正确推送:

```
% git push
Enumerating objects: 4, done.
Counting objects: 100% (4/4), done.
Delta compression using up to 8 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 314 bytes | 1024 bytes/s, done.
Total 3 (delta 1), reused 1 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To github.com:gvnn3/freebsd-src.git
   9e5243d7b659..cf6aeb8d7dda  gnn-feature -> gnn-feature
```

此时，您的工作现在在 GitHub 上的分支中，您可以与其他协作者共享该链接。

### 5.8. 陆地一个 github 拉取请求

本节记录了如何将针对 FreeBSD Git 镜像在 GitHub 提交的 GitHub 拉取请求落地。虽然这目前不是官方提交补丁的方式，但有时会有好的修复以这种方式传入，最简单的方法是将其带入提交者的树，并从那里将其推送到 FreeBSD 的树中。类似的步骤可以用来从其他存储库拉取分支并将其落地。在提交他人的拉取请求时，应特别注意检查所有更改，以确保它们确实如所表示。

在开始之前，请确保本地 Git 存储库是最新的，并且已设置了正确的源，如上所示。此外，请确保具有以下源：

```
% git remote -v
freebsd https://git.freebsd.org/src.git (fetch)
freebsd ssh://git@gitrepo.freebsd.org/src.git (push)
github https://github.com/freebsd/freebsd-src (fetch)
github https://github.com/freebsd/freebsd-src (fetch)
```

通常拉请求很简单：只包含一个提交的请求。在这种情况下，可以使用简化的方法，尽管在前一节中的方法也适用。在这里，创建一个分支，挑选改动，调整提交消息，并在推送之前进行检查。本示例中使用分支 staging ，但可以是任何名称。该技术适用于拉请求中的任何数量的提交，特别是当改动可以干净地应用到 FreeBSD 树时。然而，当有多个提交时，尤其是当需要进行小的调整时， git rebase -i 比 git cherry-pick 更好。简要地说，这些命令创建一个分支；从拉请求中挑选改动；测试它；调整提交消息；然后将其快进合并回 main 。下面是 PR 编号 $PR 。在调整消息时，添加 Pull Request:<span> </span><a href="https://github.com/freebsd-src/pull/$PR" class="bare">https://github.com/freebsd-src/pull/$PR</a> 。提交到 FreeBSD 存储库的所有拉请求都应该由至少一个人审查。这个人不一定是提交者，但在这种情况下，提交者应该信任其他审阅人员有能力审查提交。在将拉请求推送到存储库之前对拉请求进行代码审查的提交者应该在提交中添加 Reviewed by: 行，因为在这种情况下不是隐含的。也应将在 github 上审查并批准提交的任何人添加到 Reviewed by: 。如常，应该小心确保更改执行了其预定的功能，并且没有恶意代码存在。

```
Author:     github-user <38923459+github-user@users.noreply.github.com>
```

应该向作者礼貌地请求更好的姓名和/或电子邮件。应该特别注意确保不引入样式问题或恶意代码。

```
% git fetch github pull/$PR/head:staging
% git rebase -i main staging	# to move the staging branch forward, adjust commit message here
<do testing here, as needed>
% git checkout main
% git pull --ff-only		# to get the latest if time has passed
% git checkout main
% git merge --ff-only staging
<test again if needed>
% git push freebsd --push-option=confirm-author
```

对于有多个带冲突提交的复杂拉请求，请按照以下大纲操作。

1. 检查拉取请求 git checkout github/pull/XXX
2. 创建一个分支来重新基于 git checkout -b staging
3. 将 staging 分支重新基于最新的 main ，使用 git rebase -i main staging
4. 解决冲突并进行必要的测试
5. 将 staging 分支快进到上述的 main
6. 对更改进行最终的合理性检查，确保一切正常
7. 推送到 FreeBSD 的 Git 仓库。

当将其他地方开发的分支合并到本地树以进行提交时，这也适用。

完成拉取请求后，使用 GitHub 的网页界面关闭它。 值得注意的是，如果您的 github 原点使用 https:// ，您唯一需要 GitHub 帐户的步骤就是关闭拉取请求。

## 版本控制历史

该项目已迁移到 git。

FreeBSD 源代码库于 2008 年 5 月 31 日从 CVS 转换为 Subversion。第一个真正的 SVN 提交是 r179447。源代码库于 2020 年 12 月 23 日从 Subversion 转换为 Git。最后一个真正的 svn 提交是 r368820。第一个真正的 git 提交哈希值为 5ef5f51d2bef80b0ede9b10ad5b0e9440b60518c.

FreeBSD doc/www 存储库于 2012 年 5 月 19 日从 CVS 切换到 Subversion。第一个真正的 SVN 提交是 r38821。文档存储库于 2020 年 12 月 8 日从 Subversion 切换到 Git。最后的 SVN 提交是 r54737。第一个真正的 git 提交哈希是 3be01a475855e7511ad755b2defd2e0da5d58bbe。

FreeBSD ports 存储库于 2012 年 7 月 14 日从 CVS 切换到 Subversion。第一个真正的 SVN 提交是 r300894。ports 存储库于 2021 年 4 月 6 日从 Subversion 切换到 Git。最后的 SVN 提交是 r569609。第一个真正的 git 提交哈希是 ed8d3eda309dd863fb66e04bccaa513eee255cbf。

## 7. 设置、惯例和传统

作为一名新开发者，有许多事情要做。第一组步骤只适用于提交者。这些步骤必须由导师为那些不是提交者的人完成。

### 7.1. 对于新提交者

那些被授予对 FreeBSD 代码库的提交权限的人必须遵循这些步骤。

- 在提交每个更改之前，请获得导师的批准！
- 所有的源代码提交都必须首先到达 FreeBSD-CURRENT 分支，然后再合并到 FreeBSD-STABLE 分支。FreeBSD-STABLE 分支必须与早期版本保持 ABI 和 API 兼容性。请勿合并会破坏此兼容性的更改。

** 新提交者的步骤**

1. 将作者实体添加到 doc/shared/authors.adoc - 添加作者实体。后续步骤依赖该实体，缺少此步骤将导致构建失败。这是一个相对简单的任务，但仍然是版本控制技能的良好第一次测试。
2. 更新开发人员和贡献者列表 doc/shared/contrib-committers.adoc - 添加一个条目，然后将出现在“开发人员”部分的贡献者列表中。条目按姓氏排序。
   doc/shared/contrib-additional.adoc - 删除该条目。条目按名字排序。
3. 在 doc/website/data/en/news/news.toml 中添加一个新闻条目 - 添加一条条目。查找其他宣布新提交者的条目，按照格式进行操作。使用批准邮件中的日期。
4. 添加一个 PGP 密钥 Dag-Erling Smørgrav <<a href="mailto:des@FreeBSD.org">des@FreeBSD.org</a>> 编写了一个 shell 脚本（doc/documentation/tools/addkey.sh），以便更轻松地进行此操作。有关更多信息，请参阅 README 文件。 

   使用 doc/documentation/tools/checkkey.sh 来验证密钥是否符合最低的最佳实践标准。 
   
   
   添加和检查密钥后，将更新后的文件添加到源代码控制，然后提交它们。此文件中的条目按姓氏排序。 
   
   在版本库中非常重要有一个当前的 PGP/GnuPG 密钥。该密钥可能需要用于提交者的积极身份验证。例如， FreeBSD Administrators <<a href="mailto:admins@FreeBSD.org">admins@FreeBSD.org</a>> 可能需要它来进行账户恢复。可从 https://docs.FreeBSD.org/pgpkeys/pgpkeys.txt 下载完整的 FreeBSD.org 用户密钥环。 
   
5. 更新导师和学员信息 src/share/misc/committers-.dot - 在当前提交者部分添加一个条目，其中仓库是 doc ， ports 或 src ，具体取决于授予的提交特权。
   在底部部分为每个额外的导师/受导者关系添加一个条目。
6. 生成 Kerberos 密码 参见 FreeBSD 集群的 Kerberos 和 LDAP 网络密码以生成或设置 Kerberos 账户，以便与其他 FreeBSD 服务（如 bug 跟踪数据库）一起使用（在该步骤中，您将获得一个 bug 跟踪账户）。
7. 可选：启用 Wiki 账户 FreeBSD Wiki 账户 - Wiki 账户允许共享项目和想法。尚未拥有账户的人可以按照 Wiki/About 页面上的说明获取账户。如果需要 Wiki 账户帮助，请联系 wiki-admin@FreeBSD.org。
8. 可选：更新维基信息维基信息 - 在获取对维基的访问权限后，一些人会向“我们是如何到达这里”、“IRC 昵称”、“FreeBSD 的狗”和/或“FreeBSD 的猫”页面添加条目。
9. 可选：使用个人信息更新 Ports ports/astro/xearth/files/freebsd.committers.markers 和 src/usr.bin/calendar/calendars/calendar.freebsd - 一些人会向这些文件添加自己的条目，以显示他们的位置或生日日期。
10. 可选：防止重复邮件将订阅者提交消息到所有 doc 存储库分支的提交消息、所有 ports 存储库分支的提交消息或所有 src 存储库分支的提交消息可能希望取消订阅，以避免收到提交消息和跟进的重复副本。

### 7.2. 对于每个人

1. 向其他开发者介绍自己，否则没有人会知道你是谁，也不知道你在 FreeBSD 上正在做什么。介绍不需要很详细，只需写一两段关于你是谁，作为 FreeBSD 开发者计划做什么，以及谁会是你的导师的内容。将此电子邮件发送到 FreeBSD 开发者邮件列表，你就可以开始了！
2. 登录 freefall.FreeBSD.org ，创建一个 /var/forward/user 文件（其中 user 是你的用户名），其中包含你希望邮件转发到 yourusername@FreeBSD.org 的电子邮件地址。这包括所有提交消息以及发送到 FreeBSD 提交者邮件列表和 FreeBSD 开发者邮件列表的所有其他邮件。如果空间需要释放，已经在 freefall 上占据永久居住的非常大的邮箱可能会被截断而没有警告，因此请转发或将其保存到其他地方。 

   如果你的电子邮件系统使用严格规则的 SPF，你应该将 mx2.FreeBSD.org 排除在 SPF 检查之外。 
   由于处理垃圾邮件给处理邮件列表的中央邮件服务器带来了严重负担，前端服务器会进行一些基本检查，并会根据这些检查丢弃一些消息。目前，对连接主机进行正确的 DNS 信息检查是唯一的检查，但这可能会改变。有些人将这些检查归咎于弹回有效电子邮件。要关闭你电子邮件的这些检查，请在 freefall.FreeBSD.org 上创建一个名为~/.spam_lover 的文件。

>那些是开发人员但不是提交者的人将不会被订阅到提交者或开发人员的邮件列表。订阅是根据访问权限派生的。 

#### 7.2.1. SMTP 访问设置

对于那些愿意通过 FreeBSD.org 基础设施发送电子邮件消息的人，请按照以下说明操作：

1. 将您的邮件客户端指向 smtp.FreeBSD.org:587 。
2. 启用 STARTTLS。
3. 确保您的 From: 地址设置为 <em>yourusername</em>@FreeBSD.org 。
4. 对于认证，您可以使用您的 FreeBSD Kerberos 用户名和密码（请参阅 FreeBSD 集群的 Kerberos 和 LDAP 网密码）。首选 <em>yourusername</em>/mail 主体，因为它仅适用于对邮件资源的认证。 
在输入用户名时，请勿包含 @FreeBSD.org 。 

>附加说明<br />*仅接受来自 <em>yourusername</em>@FreeBSD.org 的邮件。如果您以一个用户的身份进行验证，则不允许从另一个用户发送邮件。*将添加一个标头，显示 SASL 用户名：( Authenticated sender:<span> </span><em>username</em> )。* 主机设有各种速率限制，以减少暴力攻击的尝试。 

##### 7.2.1.1. 使用本地 MTA 将电子邮件转发到 FreeBSD.org SMTP 服务

也可以使用本地 MTA 将本地发送的电子邮件转发到 FreeBSD.org SMTP 服务器。

示例 1. 使用 Postfix

要告诉本地的 Postfix 实例，任何来自 <em>yourusername</em>@FreeBSD.org 的内容都应该转发到 FreeBSD.org 服务器，请将以下内容添加到您的 main.cf 文件中：

```
sender_dependent_relayhost_maps = hash:/usr/local/etc/postfix/relayhost_maps
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/usr/local/etc/postfix/sasl_passwd
smtp_use_tls = yes
```

创建 /usr/local/etc/postfix/relayhost_maps 文件，并添加以下内容：

```
yourusername@FreeBSD.org  [smtp.freebsd.org]:587
```

创建 /usr/local/etc/postfix/sasl_passwd 文件，并添加以下内容：

```
[smtp.freebsd.org]:587          yourusername:yourpassword
```

如果电子邮件服务器被其他人使用，您可能希望阻止他们以您的地址发送电子邮件。为此，请将以下内容添加到您的 main.cf 文件中：

```
smtpd_sender_login_maps = hash:/usr/local/etc/postfix/sender_login_maps
smtpd_sender_restrictions = reject_known_sender_login_mismatch
```

创建 /usr/local/etc/postfix/sender_login_maps 文件，并添加以下内容：

```
yourusername@FreeBSD.org yourlocalusername
```

其中 yourlocalusername 是用于连接到 Postfix 本地实例的 SASL 用户名。

示例 2. 使用 OpenSMTPD

要告诉本地的 OpenSMTPD 实例，任何来自 <em>yourusername</em>@FreeBSD.org 的邮件都应转发到 FreeBSD.org 服务器，请将以下内容添加到您的 smtpd.conf 文件中：

```
action "freebsd" relay host smtp+tls://freebsd@smtp.freebsd.org:587 auth <secrets>
match from any auth yourlocalusername mail-from "_yourusername_@freebsd.org" for any action "freebsd"
```

在这里，yourlocalusername 是用于连接到本地 OpenSMTPD 实例的 SASL 用户名。

创建 /usr/local/etc/mail/secrets，并添加以下内容：

```
freebsd	yourusername:yourpassword
```

示例 3. 使用 Exim

要指示本地 Exim 实例将所有来自 <em>example</em>@FreeBSD.org 的邮件转发到 FreeBSD.org 服务器，请将以下内容添加到 Exim 配置中：

```
Routers section: (at the top of the list):
freebsd_send:
   driver = manualroute
   domains = !+local_domains
   transport = freebsd_smtp
   route_data = ${lookup {${lc:$sender_address}} lsearch {/usr/local/etc/exim/freebsd_send}}

Transport Section:
freebsd_smtp:
        driver = smtp
  tls_certificate=<local certificate>
  tls_privatekey=<local certificate private key>
  tls_require_ciphers = EECDH+ECDSA+AESGCM:EECDH+aRSA+AESGCM:EECDH+ECDSA+SHA384:EECDH+ECDSA+SHA256:EECDH+aRSA+SHA384:EECDH+aRSA+SHA256:EECDH+AESGCM:EECDH:EDH+AESGCM:EDH+aRSA:HIGH:!MEDIUM:!LOW:!aNULL:!eNULL:!LOW:!RC4:!MD5:!EXP:!PSK:!SRP:!DSS
  dkim_domain = <local DKIM domain>
  dkim_selector = <local DKIM selector>
  dkim_private_key= <local DKIM private key>
  dnssec_request_domains = *
  hosts_require_auth = smtp.freebsd.org

Authenticators:
freebsd_plain:
  driver = plaintext
  public_name = PLAIN
  client_send = ^example/mail^examplePassword
  client_condition = ${if eq{$host}{smtp.freebsd.org}}
```

创建 /usr/local/etc/exim/freebsd_send，内容如下：

```
example@freebsd.org:smtp.freebsd.org::587
```

### 7.3. 导师

所有新开发人员在最初几个月内都会被分配一位导师。导师负责教导受 mentee 之约束并指导他们融入开发者社区。导师在这个初始时期也会对 mentee 的行为负个人责任。

对于提交者：在获得导师批准之前，请勿提交任何内容。在提交消息中使用 Approved by: 行记录该批准。

当导师认为学员已经掌握了相关知识并准备好独立提交时，导师会通过提交给导师的方式宣布。这文件在每个仓库的 admin 孤儿分支中。如何访问这些分支的详细信息可以在"admin"分支中找到。

## 8. 提交前审核

代码审查是提高软件质量的一种方式。以下准则适用于提交到 main (-CURRENT) 分支的 src 仓库。其他分支、 ports 和 docs 树有其自己的审查政策，但这些准则通常适用于需要审查的提交：

- 所有非微不足道的更改在提交到仓库之前应进行审查。
- 审查可以通过电子邮件、Bugzilla、Phabricator 或其他机制进行。在可能的情况下，审查应该是公开的。
- 负责代码更改的开发人员也负责进行所有必要的审查相关更改。
- 代码审查可以是一个迭代过程，直到补丁准备好提交为止。具体来说，一旦提交补丁进行审查，应该在提交前收到明确的“看起来不错”的反馈。只要是明确的，这可以采用任何对审查方法有意义的形式。
- 超时并不代替审查。

有时候代码审查会花费比你期望的更长的时间，特别是对于更大的功能。加快审查你的补丁的方式包括：

- 审查其他人的补丁。如果你帮忙，每个人都会更愿意为你做同样的事情；善意是我们的货币。
- 快速处理补丁。如果很紧急，说明为什么对你来说很重要让这个补丁上线，并在每隔几天就快速处理它。如果不是很紧急，一周发一次普通的询问是礼貌。记住你正在向其他专业开发人员索要宝贵的时间。
- 在邮件列表、IRC 等地方寻求帮助。其他人可能可以直接帮助您，或者建议一个审阅者。
- 将您的补丁拆分成多个较小的补丁，这些补丁之间互相建立。您的补丁越小，别人就越有可能快速查看。在进行大规模更改时，从努力开始即牢记此点是有帮助的，因为在完成后将大型更改拆分为较小的更改通常很困难。

开发人员应参与代码审查，既可以担任审查者，也可以担任被审阅者。如果有人愿意审查您的代码，您应该回报别人同样的帮助。请注意，尽管任何人都可以审查并就补丁提供反馈，但只有适当的主题专家才能批准更改。这通常是与相关代码定期工作的提交者。

在某些情况下，可能没有专业的主题专家可用。在这种情况下，由经验丰富的开发人员审查，再加上适当的测试，就足够了。

## 9. 提交日志消息

本节包含有关如何格式化提交日志的一些建议和传统。

### 9.1. 为什么提交信息很重要？

当你在 Git、Subversion 或其他版本控制系统 (VCS) 中提交更改时，会提示你编写一些描述提交的文字——提交信息。这个提交信息有多重要？你应该花费一些精力去编写它吗？如果你只是简单地写 fixed a bug ，这真的重要吗？

大多数项目都有不止一个开发者，并且持续一段时间。提交信息是与其他开发者沟通的一个非常重要的方法，无论是在现在还是将来。

FreeBSD 有数百名活跃开发人员和数十万次跨越几十年历史的提交。开发人员社区在这段时间里学到了好的提交消息有多么宝贵；有时这些是艰难学来的教训。

提交消息至少具有三个目的：

- 与其他开发人员交流 FreeBSD 提交会向各种邮件列表发送电子邮件。这些包括提交消息以及补丁本身的副本。提交消息也可以通过像 git log 这样的命令查看。这有助于让其他开发人员意识到正在进行的更改；其他开发人员可能希望测试更改，可能对该主题感兴趣并希望更详细地审查，或者可能正在进行他们自己的项目，这些项目将受益于互动。
- 在一个具有悠久历史的大型项目中，当调查问题或行为变化时，可能很难找到感兴趣的变化。冗长详细的提交消息允许搜索可能相关的变化。例如， git log --since 1year --grep 'USB timeout' 。
- 提供历史文档提交消息用于记录变化，供将来的开发人员查阅，也许是数年甚至数十年后的开发人员。今天显而易见的变化，将来可能并非如此。

git blame 命令使用变更（哈希和主题行）标注源文件的每一行。

确定了重要性之后，下面是一个良好的 FreeBSD 提交消息的要素：

### 9.2. 从一个主题开始

提交消息应以一个单行主题开始，简要总结更改。主题本身应该使读者能够快速确定更改是否感兴趣。

### 9.3. 保持主题行短小

主题行应尽可能短，同时保留必要信息。这样可以提高浏览 Git 日志的效率，使得 git log --oneline 可以在单个 80 列行上显示短哈希和主题。一个好的经验法则是保持在 63 个字符以下，如果可能的话，目标是约 50 个字符或更少。

### 9.4. 如适用，主题行前缀应以组件为前缀

如果更改涉及特定组件，则主题行可以以该组件名称为前缀，并加上冒号(:)。

✓ `foo: Add -k option to keep temporary data`

在上述建议的 63 个字符限制中包含前缀，以便 git log --oneline 避免换行。

### 9.5. 主题的第一个字母大写

句子的首字母要大写。如果有前缀，不需要大写，除非必要（例如， USB: 要大写）。

### 9.6. 不要在标题行末尾使用标点符号

不要以句号或其他标点符号结尾。在这方面，标题行类似于报纸标题。

### 9.7. 用空行分隔主题和正文

用空行将正文与主题分开。

一些微不足道的提交不需要正文，只有主题。

✓ `ls: Fix typo in usage text`

### 9.8. 将消息限制在 72 列

git log 和 git format-patch 缩进提交消息四个空格。将消息限制在 72 列不仅提供右侧边缘的匹配边界，还保持提交消息在格式化补丁中低于 RFC 2822 建议的电子邮件行长度限制 78 个字符。此限制与可能呈现提交消息的各种工具兼容；长行长度可能导致行换行不一致。

### 9.9. 使用现在时，祈使语气

这有助于提供简短的主题行，并确保一致性，包括与自动生成的提交消息（例如由 git revert 生成的消息）一致。在阅读提交主题列表时，这一点非常重要。把主题看作是完成这句话“应用后，此更改将…”。

✓ `foo: Implement the -k (keep) option` ✗ `foo: Implemented the -k option` ✗ `This change implements the -k option in foo` ✗ `-k option added`

### 9.10. 专注于什么和为什么，而不是如何

解释更改完成的内容以及为什么要这样做，而不是如何做。

不要假设读者熟悉这个问题。解释更改的背景和动机。如果有基准数据，请包含在内。

如果更改有局限性或不完整的方面，请在提交信息中描述它们。

### 9.11. 考虑提交信息的部分是否可以改为代码注释

有时在写提交信息时，您可能会发现自己写了一两句来解释更改中一些棘手或令人困惑的方面。当发生这种情况时，请考虑将该解释作为代码本身的注释是否有价值。

### 9.12. 为将来的自己编写提交消息

写提交消息时，您可以将所有上下文考虑在内 - 造成更改的原因，考虑和拒绝的备选方案，更改的限制等等。想象一下，未来一两年再次访问更改，并编写提交消息，以提供必要的上下文。

### 9.13. 提交消息应该能够单独存在

您可以包含对邮件列表文章、基准结果网站或代码审查链接的引用。然而，提交消息应包含所有相关信息，以防这些引用将来不再可用。

同样，提交可能会参考先前的提交，例如在修复错误或回滚的情况下。除了提交标识符（修订版或哈希）外，还应包括来自所引用提交的主题行（或其他合适的简要引用）。随着每次版本控制系统的迁移（从 CVS 到 Subversion 到 Git），来自先前系统的修订标识符可能变得难以跟踪。

### 9.14. 在页脚中包含适当的元数据

除了在每次提交中包含信息性消息外，可能还需要一些额外的信息。

此信息包含一个或多个行，其中包含关键字或短语，一个冒号，用于格式化的制表符，然后是附加信息。

关键字或短语是：

| `PR:`                          | 受此提交影响的问题报告（如果有的话）。可以在一行上指定多个受影响的问题报告（通常通过关闭）。多个 PR 可以用逗号或空格分隔。                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Reported by:`                 | 报告问题的人的姓名和电子邮件地址；对于开发者，只需在 FreeBSD 集群上使用的用户名。通常在没有 PR 时使用，例如如果问题是在邮件列表上报告的。                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `Submitted by:`<br /> (已废弃) | 这已被 git 弃用；提交的补丁应该通过使用 git commit --author 设置作者，作者应使用全名和有效电子邮件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `Reviewed by:`                 | 更改审阅者的姓名和电子邮件地址；对于开发者来说，只需提供在 FreeBSD 集群上的用户名。如果补丁提交到邮件列表进行审查，并且审查结果是积极的，那么只需包括列表名称。如果审阅者不是项目成员，请提供姓名、电子邮件地址，以及如果 ports 是外部角色如维护者：<br /><br /> 由开发者审阅：<br /><br />`Reviewed by: username`<br /><br />由不是开发者的 ports 维护者审阅：<br /><br />`Reviewed by: Full Name <valid@email> (maintainer)`                                                                                                                                                           |
| `Tested by:`                   | 测试更改的人员姓名和电子邮件地址；对于开发人员，只需在 FreeBSD 集群上使用用户名。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `Approved by:`                 | 批准更改的人员姓名和电子邮件地址；对于开发人员，只需在 FreeBSD 集群上使用用户名。<br /><br />有几种情况下是习惯性批准的：<br />_ 在新提交者在指导下 _ 提交到由 LOCKS 文件（src）覆盖的树区域 _ 在发布周期期间 _ 提交到您没有提交位的仓库（例如，src 提交者提交到 docs） * 提交到由其他人维护的 port<br /><br />在指导下，请获得导师的批准再进行提交。在此字段中输入导师的用户名，并注明他们是导师：<br /><br />`Approved by: username-of-mentor (mentor)`<br /><br />如果团队批准了这些提交，则在括号中包括团队名称，后跟批准者的用户名。例如：<br /><br />`Approved by: re (username)` |
| `Obtained from:`               | 从获得代码的项目的名称（如果有的话）。不要使用此行来表示个人的姓名。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `Fixes:`                       | 由此更改修复的提交的 Git 短哈希和标题行，由 git log -n 1 --oneline GIT-COMMIT-HASH 返回。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `MFC after:`                   | 要在以后收到一封电子邮件提醒进行 MFC，请指定计划进行 MFC 之后的天数、周数或月数。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `MFC to:`                      | 如果提交应合并到一组稳定分支中，则指定分支名称。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `MFH:`                         | 如果提交要合并到 ports 季度分支名称，请指定季度分支。例如 2021Q2 。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `Relnotes:`                    | 如果更改是下一个发布版本的发行说明的候选项，请设置为 yes 。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `Security:`                    | 如果更改涉及安全漏洞或安全风险，请添加一个或多个参考文献或问题描述。如果可能，请包括一个 VuXML URL 或 CVE ID。                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `Event:`                       | 此次提交的事件描述。如果这是一个经常性事件，可以添加年份甚至月份。例如，这可能是 FooBSDcon 2019 。这行的想法是要承认会议、聚会和其他类型的见面活动，并展示这些是有用的。请不要使用 Sponsored by: 这一行，因为那是为支持某些功能或开发人员的组织而设计的。                                                                                                                                                                                                                                                                                                                                |
| `Sponsored by:`                | 此更改的赞助组织，如果有的话。用逗号分隔多个组织。如果只有一部分工作是有赞助的，或者不同的作者获得了不同数量的赞助，请在每个赞助名称后用括号适当表示。例如， Example.com (alice, code refactoring), Wormulon (bob), Momcorp (cindy) 显示 Alice 获得 Example.com 赞助进行代码重构，而 Wormulon 赞助 Bob 的工作，Momcorp 赞助 Cindy 的工作。其他作者要么没有得到赞助，要么选择不列出赞助。                                                                                                                                                                                                 |
| `Pull Request:`                | 此更改是作为针对 FreeBSD 公共只读 Git 存储库之一的拉取请求或合并请求提交的。它应该包括拉取请求的完整 URL，因为这些通常充当代码审查。例如: https://github.com/freebsd/freebsd-src/pull/745                                                                                                                                                                                                                                                                                                                                                                                                |
| `Co-authored-by:`              | 提交的其他作者的姓名和电子邮件地址。GitHub 在 https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors 中有关于 Co-authored-by 托架的详细描述。                                                                                                                                                                                                                                                                                                                                                  |
| `Signed-off-by:`               | ID 证明符合 https://developercertificate.org/                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `Differential Revision:`       | Phabricator 审核的完整 URL。这一行必须是最后一行。例如： https://reviews.freebsd.org/D1708 。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

例 4.基于 PR 的提交的提交日志

该提交基于 John Smith 提交的 PR 的补丁。提交消息"PR"字段已填写。

```
...

PR:		12345
```

提交者用 git commit --author "John Smith <<a href="mailto:John.Smith@example.com">John.Smith@example.com</a>>" 设置了补丁的作者。

示例 5. 需要审查的提交日志

虚拟内存系统正在被修改。在向适当的邮件列表（在本例中是 freebsd-arch ）发送补丁并且修改得到批准后。

```
...

Reviewed by:	-arch
```

示例 6. 需要批准的提交日志

在与列出的 MAINTAINER 一起工作后，提交了 port，他表示可以继续提交。

```
...

Approved by:	abc (maintainer)
```

其中 abc 是批准人的账号名。

示例 7。提交日志，包含从 OpenBSD 引入代码的提交

基于 OpenBSD 项目工作的一些代码提交

```
...

Obtained from:	OpenBSD
```

示例 8。对 FreeBSD-CURRENT 进行更改的提交日志，计划稍后提交到 FreeBSD-STABLE

将一些代码提交，将在两周后从 FreeBSD-CURRENT 合并到 FreeBSD-STABLE 分支中。

```
...

MFC after:	2 weeks
```

其中 2 是计划在多少天、周或月之后进行 MFC 的数量。周选项可以是 day ， days ， week ， weeks ， month ， months 。

经常需要组合这些。

考虑这样一种情况，用户提交了一个包含来自 NetBSD 项目的代码的 PR。查看 PR 时，开发人员发现这不是他们通常工作的树的领域，因此他们通过 arch 邮件列表审查了更改。由于更改比较复杂，开发人员选择在一个月后进行 MFC，以便充分测试。

要包含在提交中的额外信息应该如下

示例 9。示例合并提交日志

```
PR:		54321
Reviewed by:	-arch
Obtained from:	NetBSD
MFC after:	1 month
Relnotes:	yes
```

## 10. 新文件的首选许可证

FreeBSD 项目的完整许可政策可以在 https://www.FreeBSD.org/internal/software-license 找到。本节的其余部分旨在帮助您开始。通常情况下，当有疑问时，请询问。给予建议比修复源代码要容易得多。

FreeBSD 项目建议并使用以下文本作为首选许可方案：

```
/*-
 * SPDX-License-Identifier: BSD-2-Clause
 *
 * Copyright (c) [year] [your name]
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 *
 * THIS SOFTWARE IS PROVIDED BY THE AUTHOR AND CONTRIBUTORS ``AS IS'' AND
 * ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED.  IN NO EVENT SHALL THE AUTHOR OR CONTRIBUTORS BE LIABLE
 * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
 * OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
 * HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
 * LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
 * OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
 * SUCH DAMAGE.
 *
 * [id for your version control system, if any]
 */
```

FreeBSD 项目强烈反对在新代码中使用所谓的“广告条款”。由于 FreeBSD 项目有大量贡献者，许多商业供应商难以遵守此条款。如果您的代码树中包含广告条款，请考虑将其删除。事实上，请考虑为您的代码使用上述许可证。

FreeBSD 项目反对完全新的许可证和标准许可证的变体。新的许可证需要获得 core@FreeBSD.org 的批准才能存在于 src 仓库中。使用的许可证种类越多，给希望利用此代码的人带来的问题就越多，通常是由于措辞不当的许可证带来的意外后果。

项目政策规定，一些非 BSD 许可证下的代码必须仅放置在存储库的特定部分，并且在某些情况下，编译必须是有条件的，甚至默认情况下被禁用。例如，GENERIC 内核必须仅在与 BSD 许可证相同或实质上相似的许可证下编译。GPL、APSL、CDDL 等许可证的软件不得编译到 GENERIC 中。

开发人员被提醒，在开源中，正确理解“开放”与正确理解“源码”同样重要，因为对知识产权的不当处理会带来严重后果。如有任何问题或关注，请立即向核心团队反映。

## 11. 跟踪授予 FreeBSD 项目的许可证

各种软件或数据存在于存储库中，FreeBSD 项目已被授予特殊许可以使用它们。一个例子是 Terminus 字体，用于 vt(4)。在这里，作者 Dimitar Zhekov 允许我们在 2 条款的 BSD 许可证下使用“Terminus BSD 控制台”字体，而不是他通常使用的开放字体许可证。

明智的做法是记录所有此类许可授予的情况。因此，core@FreeBSD.org 已决定对其进行归档。每当 FreeBSD 项目被授予特殊许可时，我们要求通知 core@FreeBSD.org。参与安排此类许可授予的任何开发人员，请将详细信息发送至 core@FreeBSD.org，包括：

- 授予特殊许可的个人或组织的联系方式。
- 仓库中受许可授予涵盖的文件、目录等，包括提交特许材料的修订版本号。
- 许可证生效日期。除非另有协议，否则这将是软件作者发布许可证的日期。
- 许可证文本。
- 有关适用于 FreeBSD 使用许可材料的任何限制、限制或例外的说明。
- 任何其他相关信息。

待 core@FreeBSD.org 确信所有必要的细节都已收集并且正确，秘书将发送一封包含许可详情的 PGP 签名确认收据。此收据将被持久存档，并作为我们对许可授予的永久记录。

许可存档应仅包含许可授予的详细信息；这不是讨论许可或其他主题的地方。许可存档中的数据访问将根据对 core@FreeBSD.org 的要求进行。

## 12. 树中的 SPDX 标记

该项目在我们的源代码库中使用 SPDX 标记。目前，这些标记被设计为帮助自动化工具机械地重建许可要求。树中所有的 SPDX-License-Identifier 标记都应被视为信息性的。树中带有这些标记的所有文件也都有统治该文件使用的许可的副本。在出现不一致的情况下，逐字许可是控制性的。该项目尝试遵循 SPDX 规范，版本 2.2。如何标记源文件和有效的代数表达式可在附录 IV 和附录 V 中找到。该项目从 SPDX 的有效短许可标识符列表中提取标识符。该项目仅使用 SPDX-License-Identifier 标记。

截至 2021 年 3 月，树中的约 25,000 个文件中的 90,000 个已经被标记。

## 13. 开发者关系

当直接在自己的代码上工作或在已经被确认为自己责任的代码上工作时，可能不需要在提交之前与其他贡献者进行确认。当在系统中某个明显被忽视的 bug 区域工作时（我们很羞愧，有一些这样的区域），同样适用。当修改维护的系统部分时，无论是正式维护还是非正式维护，都考虑像开发者在成为贡献者之前那样请求审查。要在 Makefile 中联系列出的 MAINTAINER 中查找 ports。

要确定一个区域是否被维护，请检查树的根目录下的 MAINTAINERS 文件。如果没有人被列出，请扫描修订历史以查看过去提交更改的人。要列出最近 2 年内给定文件的所有提交作者的姓名和电子邮件地址，以及每个作者提交的次数，并按提交次数降序排序，请使用:

```
% git -C /path/to/repo shortlog -sne --since="2 years" -- relative/path/to/file
```

如果查询无人回答，或者提交者表示对受影响区域不感兴趣，请继续提交。

> 避免给维护者发送私人邮件。其他人可能对对话感兴趣，而不仅仅是最终结果。 

如果对提交有任何疑问，请在提交前进行审查。最好在那时就解决问题，而不是在它成为代码库的一部分时。如果提交引发争议，建议在问题解决之前考虑撤回更改。请记住，使用版本控制系统我们可以随时更改回来。

不要质疑他人的意图。如果他们对问题有不同的解决方案，甚至是不同的问题，这可能不是因为他们愚蠢，家庭背景可疑，或者因为他们试图破坏辛勤劳动、个人形象或 FreeBSD，而基本上是因为他们对世界有不同的观点。不同是好的。

坦诚地不同意。从事物的优点阐述你的立场，诚实地谈论它可能存在的缺点，并愿意以开放的心态看待他们的解决方案，甚至是他们对问题的看法。

接受纠正。我们都是易犯错误的。当你犯了错误时，请道歉并继续生活。不要责备自己，当然也不要因为你的错误责备他人。不要浪费时间在尴尬或指责上，只需修复问题然后继续前行。

寻求帮助。寻找（并提供）同行评审。开源软件应该擅长于应用于它的眼球数量；如果没有人审查代码，这一点就不适用了。

## 14. 如果有疑问……

当对某事不确定时，无论是技术问题还是项目惯例，请务必询问。如果保持沉默，你永远不会取得进展。

如果涉及技术问题，请在公共邮件列表中询问。避免直接给知道答案的个人发送邮件。这样每个人都可以从问题和答案中学习。

对于项目特定或管理问题，请按以下顺序询问：

- 你的导师或前导师。
- 在 IRC、电子邮件等上有经验的提交者。
- 有“帽子”的任何团队，因为他们可以给你一个明确的答案。
- 如果仍然不确定，请在 FreeBSD 开发者邮件列表上询问。

一旦您的问题得到解答，如果没有人指引您找到明确解答您问题的文档，请记录下来，因为其他人会有同样的问题。

## Bugzilla 15

FreeBSD 项目利用 Bugzilla 来跟踪漏洞和变更请求。如果您提交了在 PR 数据库中找到的修复或建议，请确保关闭 PR。如果您花时间关闭与您的提交相关的其他 PR，那也会被认为是礼貌的行为。

有非 FreeBSD.org Bugzilla 账户的提交者可以通过以下步骤将旧账户与 FreeBSD.org 账户合并：

1. 使用您的旧账户登录。
2. 打开新的 bug。选择 Services 作为产品， Bug Tracker 作为组件。在 bug 描述中列出您希望合并的账户。
3. 使用 FreeBSD.org 账户登录并发布评论到新打开的 bug 以确认所有权。有关如何为 FreeBSD Cluster 生成或设置 FreeBSD.org 账户密码的更多详细信息，请参见 Kerberos 和 LDAP 网络密码。
4. 如果有两个以上的账户需要合并，请从每个账户发布评论。

你可以在以下网址了解更多关于 Bugzilla 的信息：

- [FreeBSD 问题报告处理指南](https://docs.freebsd.org/en/articles/pr-guidelines/)
- [https://www.FreeBSD.org/support](https://www.freebsd.org/support/)

## 16. Phabricator

FreeBSD 项目利用 Phabricator 进行代码审查请求。有关详细信息，请参阅 Phabricator 维基页面。

FreeBSD.org 非 Phabricator 账户的参与者可以按照以下步骤将旧账户重命名为 FreeBSD.org 账户：

1. 将您的 Phabricator 账户电子邮件更改为您的 FreeBSD.org 电子邮件。
2. 使用您的 FreeBSD.org 帐户在我们的错误跟踪器上打开新 bug，请参阅 Bugzilla 了解更多信息。选择 Services 作为产品， Code Review 作为组件。在 bug 描述中请求重命名您的 Phabricator 帐户，并提供指向您的 Phabricator 用户的链接。例如， https://reviews.freebsd.org/p/bob_example.com/

> Fabrik 帐户无法合并，请勿开设新帐户。

## 17. Who 是谁

除了存储库管理员外，还有其他 FreeBSD 项目成员和团队，您在作为提交者的角色中可能会认识。简言之，并非所有成员，主要有以下这些：

Documentation Engineering Team <<a href="mailto:doceng@FreeBSD.org">doceng@FreeBSD.org</a>> doceng 是负责文档构建基础设施、批准新的文档提交者，并确保 FreeBSD 网站和 FTP 站点上的文档与 Subversion 树同步更新的团队。它不是冲突解决机构。绝大多数与文档相关的讨论都在 FreeBSD 文档项目邮件列表上进行。有关 doceng 团队的更多详细信息可以在其章程中找到。有意向为文档贡献的提交者应熟悉《文档项目入门》。

Konstantin Belousov <<a href="mailto:kib@FreeBSD.org">kib@FreeBSD.org</a>>, Dave Cottlehuber <<a href="mailto:dch@FreeBSD.org">dch@FreeBSD.org</a>>, Marc Fonvieille <<a href="mailto:blackend@FreeBSD.org">blackend@FreeBSD.org</a>>, John Hixson <<a href="mailto:jhixson@FreeBSD.org">jhixson@FreeBSD.org</a>>, Xin Li <<a href="mailto:delphij@FreeBSD.org">delphij@FreeBSD.org</a>>, Ed Maste <<a href="mailto:emaste@FreeBSD.org">emaste@FreeBSD.org</a>>, Mahdi Mokhtari <<a href="mailto:mmokhi@FreeBSD.org">mmokhi@FreeBSD.org</a>>, Colin Percival <<a href="mailto:cperciva@FreeBSD.org">cperciva@FreeBSD.org</a>>, Doug Rabson <<a href="mailto:dfr@FreeBSD.org">dfr@FreeBSD.org</a>>, Muhammad Moinur Rahman <<a href="mailto:bofh@FreeBSD.org">bofh@FreeBSD.org</a>> 这些是 Release Engineering Team <<a href="mailto:re@FreeBSD.org">re@FreeBSD.org</a>> 的成员。该团队负责设置发布截止日期并控制发布流程。在代码冻结期间，发布工程师对即将发布状态的任何分支的所有系统更改有最终权威。如果您希望从 FreeBSD-CURRENT 合并到 FreeBSD-STABLE（无论这些值在任何给定时间是什么），这些人是您需要联系的人。

Gordon Tetlow <<a href="mailto:gordon@FreeBSD.org">gordon@FreeBSD.org</a>>``Gordon Tetlow 是 FreeBSD 安全官员，负责监督 Security Officer Team <<a href="mailto:security-officer@FreeBSD.org">security-officer@FreeBSD.org</a>> 。

FreeBSD 提交者邮件列表{dev-src-all}、{dev-ports-all}和{dev-doc-all}是版本控制系统用于发送提交消息的邮件列表。永远不要直接发送电子邮件到这些列表。只有当回复是简短的并且直接与提交相关时，才发送回复到该列表。

FreeBSD 开发人员邮件列表所有的提交者都订阅了-developers。该列表旨在成为提交者“社区”问题的讨论区。例如，核心投票、公告等。

FreeBSD 开发人员邮件列表专供 FreeBSD 提交者使用。为了开发 FreeBSD，提交者必须能够就将在公开宣布之前解决的事项进行公开讨论。有关进行中工作的坦诚讨论不适合公开发布，可能会损害 FreeBSD。

所有 FreeBSD 提交者都不应在未经所有作者许可的情况下发布或转发 FreeBSD 开发者邮件列表之外的消息。违反者将被从 FreeBSD 开发者邮件列表中移除，并导致提交权限被暂时中止。重复或严重违反可能会导致永久撤销提交权限。

该列表不打算用作代码审查或任何技术讨论的场所。事实上，将其用作此类用途会损害 FreeBSD 项目，因为它给人一种闭门羹的感觉，所有影响整个 FreeBSD 使用社区的一般决策都在没有“公开”的情况下作出。最后但同样重要的是，永远不要在发送电子邮件到 FreeBSD 开发者邮件列表时抄送/密送另一个 FreeBSD 列表。永远不要在发送电子邮件到另一个 FreeBSD 邮件列表时抄送/密送 FreeBSD 开发者邮件列表。这样做会极大地减少该列表的好处。

## 18. SSH 快速入门指南

1. 如果您不希望每次使用 ssh(1) 时都输入密码，并且您使用密钥进行认证，那么 ssh-agent(1) 会为您提供便利。如果您想使用 ssh-agent(1)，请确保在运行其他应用程序之前先运行它。X 用户，例如，通常在他们的 .xsession 或 .xinitrc 中执行此操作。有关详细信息，请参见 ssh-agent(1)。
2. | 使用 ssh-keygen(1) 生成密钥对。密钥对将位于您的 $HOME/.ssh/ 目录中。 |     | 仅支持 ECDSA、Ed25519 或 RSA 密钥。 |
   | -------------------------------------------------------------------- | --- | ----------------------------------- |
3. 将您的公钥（$HOME/.ssh/id_ecdsa.pub、$HOME/.ssh/id_ed25519.pub 或 $HOME/.ssh/id_rsa.pub）发送给设置您为提交者的人，以便将其放入/etc/ssh-keys/中的 yourlogin freefall 中。

现在可以使用 ssh-add(1)进行身份验证，每个会话只需一次。 它会提示输入私钥的密码，然后将其存储在认证代理（ssh-agent(1)）中。 使用 ssh-add -d 来移除存储在代理中的密钥。

用一个简单的远程命令进行测试: ssh freefall.FreeBSD.org ls /usr 。

有关更多信息，请参阅 security/openssh-portable, ssh(1), ssh-add(1), ssh-agent(1), ssh-keygen(1)和 scp(1)。

有关添加、更改或移除 ssh(1)密钥的信息，请参阅本文。

## 19. Coverity® 为 FreeBSD 贡献者提供的可用性

所有 FreeBSD 开发人员都可以获得所有 FreeBSD 项目软件的 Coverity 分析结果访问权限。所有有兴趣获取自动 Coverity 运行分析结果的人，可以在 Coverity Scan 注册。

FreeBSD 维基包括了一个迷你指南，供那些有兴趣使用 Coverity® 分析报告的开发人员参考：https://wiki.freebsd.org/CoverityPrevent。请注意，这个迷你指南只能被 FreeBSD 开发人员阅读，如果您无法访问该页面，则需要请求某人将您加入适当的维基访问列表。

最后，所有将使用 Coverity® 的 FreeBSD 开发人员都被鼓励通过向 FreeBSD 开发人员的邮件列表发布任何问题来获取更多详情和使用信息。

## 20. FreeBSD 联合开发者的规则大全

FreeBSD 项目的所有参与者都应遵守自 https://www.FreeBSD.org/internal/code-of-conduct 可获取的行为准则。作为开发者，您代表着项目的公众形象，您的行为对公众对项目的看法具有至关重要的影响。本指南扩展了适用于开发者的行为准则的内容。

1. 尊重其他开发者。
2. 尊重其他贡献者。
3. 在提交之前讨论任何重要更改。
4. 尊重现有维护者（如果列在 Makefile 的 MAINTAINER 字段中或在顶层目录中列在 MAINTAINER 中）。
5. 如果请求，任何有争议的更改必须在争议解决之前由维护者撤销。安全相关的更改可能会在安全官员的裁量下覆盖维护者的意愿。
6. 更改应该先进入 FreeBSD-CURRENT，然后才进入 FreeBSD-STABLE，除非由发布工程师特别许可或者这些更改不适用于 FreeBSD-CURRENT。任何适用的非微不足道或非紧急的更改也应被允许在 FreeBSD-CURRENT 中停留至少 3 天以进行充分测试后再合并。发布工程师在 FreeBSD-STABLE 分支上享有与规则＃5 中维护者概述的同样权限。
7. 不要在公共场合与其他提交者争论；这看起来很不好。
8. 尊重所有的代码冻结，并及时阅读 committers 和 developers 邮件列表，以便了解何时实施代码冻结。
9. 在任何程序上存在疑问时，请先询问！
10. 在提交更改之前，请先进行测试。
11. 不得未经各自维护者明确批准就提交贡献软件。

如上所述，违反这些规则的行为可能导致暂时停权或在重复违规的情况下永久取消提交特权。核心成员有权暂时暂停提交特权，直到整个核心团队有机会审查问题为止。在“紧急情况”下（提交者对存储库造成破坏），存储库管理员也可以暂时暂停。只有核心团队的三分之二以上成员有权暂停提交特权超过一周或永久取消特权。这条规则的目的不是要让核心团队成为一群可以像处理空的苏打罐一样轻率处理提交者的残暴独裁者，而是为了给项目一个类似保险丝的安全装置。如果有人失去控制，重要的是能够立即处理，而不是被辩论所瘫痪。在所有情况下，被暂停或取消特权的提交者有权要求核心团队“听证会”，暂停的总持续时间将在那时确定。被暂停特权的提交者还可以在 30 天后以及以后的每 30 天要求审查（除非总暂停期少于 30 天）。完全取消特权的提交者可以在过去 6 个月后要求审查。此审查政策严格非正式，在所有情况下，核心团队保留根据自己认为的原决定采取或忽略审查请求的权利。

在项目运作的其他方面，核心是提交者的一个子集，并受相同的规则约束。只是因为某人属于核心团队，这并不意味着他们有特殊豁免权来跨越这里所确定的任何界限；核心团队的"特殊权力"只有在作为一个团体时才会发挥作用，而不是个人行为。作为个体，核心团队成员首先是提交者，其次才是核心团队成员。

### 20.1. 详情

1. 尊重其他提交者。这意味着您需要把其他提交者当作他们所是的同行开发者来对待。尽管我们偶尔试图证明相反，但一个人并不会因为愚蠢而成为提交者，而且没有什么比被同行以愚蠢对待更令人恼火的事情了。无论我们是否总是彼此尊重（每个人都有糟糕的日子），我们仍然必须始终尊重其他提交者，在公共论坛和私人电子邮件中。
   能够长期共同合作是这个项目最大的资产，远比任何代码更改重要，将关于代码的争论转化为影响我们长期能够和谐共事的问题，无论如何想象，这种交换都不值得。
   为了遵守这条规则，不要在生气或以其他可能被他人视为不必要对抗的方式行事时发送电子邮件。首先冷静下来，然后考虑如何以最有效的方式沟通，以说服其他人你论点的正确性，不要只是发泄一下情绪，以便在短期内感觉好一些，而以牺牲长期的火药战为代价。这不仅仅是非常糟糕的“能量经济”，而且反复展示公开攻击性的行为会严重影响我们良好合作的能力，将受到项目领导的严厉处理，并可能导致暂停或终止您的提交权限。项目领导将考虑提交的公共和私人通信。它不会寻求披露私人通信，但如果涉及投诉的提交者自愿提供，它将加以考虑。
   项目领导绝无仅有地不喜欢这一切选择，但团结至上。任何数量的代码或好建议都不值得牺牲这一点。
2. 尊重其他贡献者。您并非始终是提交者。曾经您也是一名贡献者。始终牢记这一点。记得曾经寻求帮助和关注时的感受。不要忘记作为贡献者，您的工作对您非常重要。记得那时的感受。不要打击、贬低或羞辱贡献者。以尊重对待他们。他们是我们未来的提交者。他们对项目的贡献与提交者一样重要。他们的贡献与您自己的一样有效和重要。毕竟，在成为提交者之前，您也做出了许多贡献。永远牢记这一点。
   将尊重其他提交者的要点考虑到对贡献者身上。
3. 在提交更改之前讨论任何重要的变动。仓库不是初始提交更正或讨论正确性的地方，这些应该首先通过邮件列表或使用 Phabricator 服务完成。提交只有在达成类似共识后才会发生。这并不意味着在纠正每个明显的语法错误或手册拼写错误之前需要征得许可，只是当建议的更改不再是显而易见的时候，获得一些反馈是很有好处的。如果结果比以前明显更好，人们真的不会介意大刀阔斧的改变，只是不喜欢被这些变化惊讶到。确保事情朝着正确的方向发展的最佳方式是由一个或多个其他提交者审查代码。
   当有疑问时，请索要审查！
4. 如果列出了现有的维护者，请尊重他们。FreeBSD 的许多部分并非以某个特定个人会在你提交更改到“他们”的领域时跳起来大声叫的意义上“拥有”，但仍然最好先核实一下。我们使用的一个惯例是在由一个或多个人积极维护的任何软件包或子树的 Makefile 中放置一个维护者行；请参阅“源代码树指导方针和政策”中的文档。当代码部分有数个维护者时，被一个维护者提交的受影响区域的更改需要至少由另一个维护者审查。在某些情况下，某事物的“维护人员”身份不明确时，请查看与相关文件有关的仓库日志，看看是否最近或主要在该领域工作的人。
5. 如果请求，任何有争议的更改必须在解决争端之前被有维护者支持撤销。安全相关更改可能在安全官员的裁量下覆盖维护者的意愿。在冲突时（当双方都确信自己是正确的时候，当然）这可能很难接受，但版本控制系统使得在有争议的更改需要持续争议时不必进行持续的争端，而是更容易地撤销有争议的更改，让每个人再次镇定下来然后尝试弄清最好的解决方法是什么。如果更改最终被证明是最好的事情，那么很容易将其重新引入。如果事实证明不是，那么用户在忙于辩论其利弊时并没有必须忍受树上的错误更改。人们很少在代码库中要求撤销，因为讨论通常在提交之前就暴露出坏或有争议的更改，但在这种罕见情况下，应该毫无争议地执行撤销，以便我们能立即转入确定它是否是错误或不是的话题。
6. 更改在 FreeBSD-STABLE 之前需经过 FreeBSD-CURRENT，除非释放工程师特别允许或者它们不适用于 FreeBSD-CURRENT。任何适用的非微不足道或者非紧急更改也应当允许在 FreeBSD-CURRENT 保持至少 3 天，以便能够进行充分测试。释放工程师在 FreeBSD-STABLE 分支上拥有与规则＃5 中概述的相同权威。这是另一个“不要就此争论”的问题，因为最终负责（并被批评）的是释放工程师，如果一个更改结果不佳。请尊重这一点，当涉及到 FreeBSD-STABLE 分支时，请给予释放工程师您的全力配合。对于碰巧观察者来说，FreeBSD-STABLE 的管理可能经常看起来过于保守，但也要记住保守主义应当是 FreeBSD-STABLE 的特色，那里适用不同的规则。如果更改立即合并到 FreeBSD-STABLE，那么让 FreeBSD-CURRENT 成为测试场地实际上是没有意义的。更改需要一些时间由 FreeBSD-CURRENT 开发人员进行测试，因此在合并之前要允许一些时间过去，除非 FreeBSD-STABLE 修复至关重要，时间紧迫或者如此明显以至于不需要进一步测试（手册页面的拼写修正，明显的错误/拼写错误修正等）。换句话说，请运用常识。
   安全分支的更改（例如， releng/9.3 ）必须得到 Security Officer Team <<a href="mailto:security-officer@FreeBSD.org">security-officer@FreeBSD.org</a>> 的成员批准，或在某些情况下，必须得到 Release Engineering Team <<a href="mailto:re@FreeBSD.org">re@FreeBSD.org</a>> 的成员批准。
7. 请勿在公共场合与其他提交者争吵；这会给人留下不好的印象。这个项目有公共形象需要维护，而这个形象对我们所有人都非常重要，特别是如果我们继续吸引新成员的话。尽管大家都尽力控制自己的情绪，有时候还是会失去耐心，说出愤怒的言辞。在这种情况下，最好的做法是尽量减少影响，直到大家都冷静下来。不要在公共场合发表愤怒的言论，也不要将私人通信或其他私人沟通转发到公共邮件列表、邮件别名、即时通讯频道或社交媒体网站。人们一对一说的话通常没有公开场合那么圆滑，这样的沟通在这里是不合适的，只会加剧本已糟糕的局面。如果发出挑衅性言辞的人至少有点风度私下发出，那么你也请保持私密性。如果你觉得受到其他开发者的不公正对待，并且这让你感到痛苦，请向核心小组提出问题，而不是公开讨论。核心小组将尽力调解，使事情恢复正常。在涉及到代码库更改并且参与者似乎无法达成和解协议的情况下，核心小组可能会指定一个双方同意的第三方来解决争议。所有参与方必须同意受这个第三方达成的决定约束。
8. 尊重所有的代码冻结期，并及时阅读 committers 和 developers 邮件列表，以便了解何时生效代码冻结期。在代码冻结期间提交未经批准的更改是一个非常大的错误，提交者需要在跳过长时间之后进行更新之前保持最新的消息。滥用这一点的人将被暂停其提交特权，直到他们从我们在格陵兰运行的 FreeBSD 快乐再教育营回来。
9. 如果对任何流程存在疑问，请先询问！许多错误都是因为有人匆忙行事，仅仅假设他们知道正确的做法。如果你以前没有做过这件事，那么很有可能你实际上不知道我们是如何做事的，真的需要先询问，否则你将会在公共场合彻底让自己难堪。询问“我该怎么做这件事？”并没有任何羞耻感。我们已经知道你是一个聪明的人；否则，你不会成为提交者。
10. 在提交更改之前测试它们。如果你的更改涉及内核，请确保能够编译 GENERIC 和 LINT。如果你的更改发生在其他任何地方，请确保能够制作完整的世界。如果你的更改涉及到一个分支，请确保你的测试是在运行该代码的机器上进行的。如果你有一个可能会影响其他架构的更改，请务必在所有支持的架构上进行测试。请确保你的更改适用于支持的工具链。请参考 FreeBSD 内部页面，查看可用资源列表。随着其他架构被添加到 FreeBSD 支持的平台列表中，将会提供相应的共享测试资源。
11. 未经相关维护人员明确批准，不要提交到贡献软件。贡献软件是指 src/contrib、src/crypto 或 src/sys/contrib 目录下的任何内容。
    上述目录用于通常导入到供应商分支的贡献软件。在那里提交内容可能会在导入更新版本的软件时引起不必要的麻烦。一般来说，考虑将补丁发送到上游供应商。经维护人员许可，补丁可以先提交到 FreeBSD。
    修改上游软件的原因包括对紧密耦合的依赖项进行严格控制或其代码在规范库的分发中缺乏可移植性。不论原因如何，尽量减少复刻的维护负担对维护人员有帮助。避免对文件进行琐碎或外观上的更改，因为这会使以后每次合并更加困难：每次导入都需要手动重新验证此类补丁。
    如果某个特定软件缺乏维护者，鼓励您接管所有权。如果您对当前的维护人员信息不确定，请发送电子邮件至 FreeBSD 架构和设计邮件列表咨询。

### 20.2. 多架构政策

FreeBSD 在最近的几个发行周期中添加了几个新的架构 ports，现在真正不再是一个 i386™ 中心的操作系统。为了使 FreeBSD 跨我们支持的平台更容易保持可移植性，核心团队制定了这一规定：

我们的 32 位参考平台是 i386，我们的 64 位参考平台是 amd64。主要的设计工作（包括主要的 API 和 ABI 更改）在至少一个 32 位和至少一个 64 位平台上必须经过验证，最好是主要的参考平台，在提交到源代码树之前。

开发者还应该了解我们的硬件架构长期支持的层级政策。这里的规则旨在在开发过程中提供指导，并且与在该部分列出的特性和架构的要求不同。发布时架构特性支持的层级规则比开发过程中的更为严格。

### 20.3. 多编译器政策

FreeBSD 通过 Clang 和 GCC 进行构建。项目以谨慎和可控的方式进行此额外工作，以最大化收益，同时将额外工作量降到最低。支持 Clang 和 GCC 提高了用户的灵活性。这些编译器各有优缺点，支持两者允许用户选择最适合其需求的编译器。Clang 和 GCC 支持类似的 C 和 C++ 方言，仅需相对少量的条件代码。项目通过使用两种编译器的特性，增加了代码覆盖率并提高了代码质量。通过支持这些编译器，项目能够在更多用户环境中构建，并利用更多 CI 环境，增加了用户的便利性并为其提供了更多测试工具。通过谨慎限制支持的版本范围到这些编译器的现代版本，项目避免了不必要的测试矩阵增加。较旧和不常见的编译器以及较旧的语言方言仅提供非常有限的支持，允许用户程序使用它们构建，但不限制基础系统使用它们构建。为了确保额外工作的收益大于其负担，具体的平衡继续演变。项目曾经支持非常旧的 Intel 编译器或旧版本的 GCC，但我们用精心选择的一系列现代编译器取代了这些过时的编译器。本节文档记录了我们使用不同编译器的情况及其预期。

FreeBSD 项目提供了一个内置的 Clang 编译器。由于在树中，这个编译器是支持最全面的编译器。所有更改在提交之前必须使用它进行编译。对于更改的完整测试，应适当使用此编译器进行。

在任何时刻，FreeBSD 项目还支持一个或多个树外编译器。目前，这是 GCC 12.x。理想情况下，提交者应该使用这个编译器测试编译，特别是针对大型或风险较大的更改。这个编译器可作为 ${TARGET_ARCH}-gcc${VERSION} 包使用，例如 aarch64-gcc12 或 riscv64-gcc12。项目运行自动化 CI 任务来使用这些编译器构建所有内容。提交者应该修复因其更改而破坏的任务。提交者可能需要在必要时使用，例如 CROSS_TOOLCHAIN=aarch64-gcc12 或 CROSS_TOOLCHAIN=llvm15 进行测试构建。

FreeBSD 项目在 github 上也有一些 CI 流水线。对于 github 上的拉取请求和推送到 github 分支的一些分支，会运行一些交叉编译任务。这些任务使用一个版本的 Clang 进行 FreeBSD 构建，该版本有时会滞后于树内编译器一段时间的主要版本。

FreeBSD 项目也在升级编译器。Clang 和 GCC 都是快速变化的目标。有些更改树内的事项，例如删除旧式的 K&R 函数声明和定义，会在编译器进入树之前完成。提交者应该尽量注意这一点，并愿意查看与新编译器有关的代码或更改的问题。此外，在新编译器版本进入树后不久，如果存在怀疑未检测到的回归，人们可能需要用旧版本编译一些内容。

除了编译器外，LLVM 的 LLD 和 GNU 的 binutils 间接被编译器使用。提交者应注意汇编语法的变化和链接器的特性，确保两种变体都能正常工作。这些组件将作为 FreeBSD 的 CI 作业的一部分，用于测试 Clang 或 GCC。

FreeBSD 项目提供头文件和库，允许使用其他编译器来构建不在基础系统中的软件。这些头文件支持使环境尽可能严格到标准，支持从 ANSI-C 的先前方言回溯到 C89，以及我们大约 ports 收集中发现的其他边缘情况。此支持限制了在头文件等地方淘汰较旧的标准，但不限制将基础系统更新到更新的方言。也不要求整个基础系统使用这些较旧的标准来编译。破坏此支持将导致 ports 收集中的软件包失败，因此在可能的情况下应避免，并在易于修复时立即修复。

FreeBSD 构建系统目前适应这些不同的环境。随着编译器添加新的警告，项目尝试修复它们。然而，有时这些警告需要大量重做，因此通过使用根据编译器版本评估正确结果的 make 变量来某种方式地抑制它们。开发者应注意这一点，并确保任何特定于编译器的标志都被适当地条件化。

#### 20.3.1. 当前编译器版本

当前，在树内编译器当前为 Clang 15.x。目前，在 github 和项目的 CI jenkins 作业中已测试 GCC 12 和 Clang 12、13、14 和 15。正在进行工作，以使树能够适用于 Clang 16。支持的最旧项目分支使用的是 Clang 12，因此构建的引导部分必须适用于 Clang 主要版本 12 到 15。

### 20.4. 其他建议

在提交文档更改时，请在提交之前使用拼写检查器。对于所有的 XML 文档，请通过运行 make lint 和文本处理/ igor 来验证格式指令是否正确。

对于手册页，请运行 sysutils/manck 和文本处理/ igor，以验证手册页上的所有交叉引用和文件引用是否正确，并且手册页具有所有适当的 MLINKS 已安装。

不要将样式修复与新功能混合在一起。样式修复是不修改代码功能的任何更改。混合更改会在请求修订之间的差异时使功能更改变得模糊，从而可能隐藏任何新的错误。在提交给 doc/ 的提交中不要将空白更改与内容更改混在一起。差异中的额外混乱会让翻译人员的工作变得更加困难。相反，对任何样式或空格更改进行清晰标记的单独提交在提交消息中。

### 20.5. 弃用功能

当需要从软件基础系统中移除功能时，请尽可能遵循以下准则：

1. 在可能的情况下，手册页面和可能的发行说明中提及该选项、实用程序或接口已被弃用。使用弃用功能会生成警告。
2. 该选项、实用程序或接口将保留到下一个主要（点零）发布。
3. 该选项、实用程序或接口将被移除，不再列入文档。它现在已经过时。通常最好在发布说明中记录其移除。

### 20.6. 隐私和机密

1. 大多数 FreeBSD 的业务是公开进行的。FreeBSD 是一个开放的项目。这意味着不仅任何人都可以使用源代码，而且大部分开发过程对公众开放进行审核。
2. 某些敏感事务必须保持私密或受限。遗憾的是无法完全透明。作为 FreeBSD 开发者，您将对信息拥有一定的特权访问权。因此，您需要遵守某些保密要求。有时，保密需求来自外部合作者或具有特定时间限制。但主要是不发布私人通信。
3. 安全官员独自控制安全咨询的发布。在涉及影响多个操作系统的安全问题时，FreeBSD 经常依赖早期访问以准备协调发布的咨询。除非 FreeBSD 开发者可以信任以维护安全性，否则不会提供这种早期访问。安全官员负责控制有关漏洞信息的预发布访问，并定时发布所有咨询。他可以在需要时要求任何具有相关知识的开发者以保密条件提供帮助准备安全修复。
4. 核心的通信在必要时保密。最初，对核心的通信将视为机密。然而，大部分核心业务最终将总结在每月或每季度的核心报告中。我们将谨慎处理，避免公开任何敏感细节。一些特别敏感的主题记录可能根本不会报告，并且仅保留在核心的私人档案中。
5. 访问某些商业敏感数据可能需要签署保密协议。只有在签署了保密协议后才能访问某些商业敏感数据。在签订任何约束性协议之前，必须咨询 FreeBSD 基金会法务人员。
6. 未经许可，私人通信不得公开。除上述具体要求外，还有一个普遍的期望，即未经涉及各方同意，不要发布开发人员之间的私人通信。在将消息转发到公共邮件列表、或发布到其他可以被非原始通信者访问的论坛或网站之前，请先征求许可。
7. 在项目专用或受限访问频道上的通信必须保持私密性。与个人通信类似，某些内部通信渠道，包括仅限 FreeBSD 提交者的邮件列表和受限访问 IRC 频道被视为私密通信。需获得许可才能发布这些来源的材料。
8. 核心团队可以批准发布。如果因为通信者众多或者无理由拒绝发布而无法获得许可，核心团队可以批准发布那些值得更广泛发表的私密事项。

## 21. 支持多种架构

FreeBSD 是一个设计用于在许多不同类型的硬件架构上运行的高度可移植的操作系统。保持机器相关（MD）和机器无关（MI）代码的清晰分离，以及尽量减少 MD 代码，是我们保持对当前硬件趋势的敏捷性的重要策略的一部分。每一个 FreeBSD 支持的新硬件架构都会大大增加代码维护、工具链支持和发布工程的成本。这也大大增加了对内核变更有效测试成本。因此，有很强的动机区分对各种架构的支持的类别，同时在几个被视为 FreeBSD“目标受众”的关键架构上保持强大。

### 21.1. 总体意图说明

FreeBSD 项目针对“商业现成生产质量（COTS）工作站、服务器和高端嵌入式系统”。通过专注于这些环境中感兴趣的一小组体系结构，FreeBSD 项目能够保持高水平的质量、稳定性和性能，同时最大限度地减少对项目中各种支持团队的负担，如 ports 团队、文档团队、安全官员和发布工程团队。硬件支持的多样性通过提供新功能和使用机会为 FreeBSD 消费者提供了更多选项，但这些好处必须始终在考虑与额外平台支持相关的现实维护成本方面慎重考虑。

FreeBSD 项目将平台目标区分为四个层次。每个层次包括消费者可以依赖的保证列表，以及项目和开发人员履行这些保证的义务。这些列表定义了每个层次的最低保证。项目和开发人员可能会在给定层次的稳定分支上提供超出最低保证的额外支持，但此类额外支持不被保证。每个平台目标都被分配到每个稳定分支的特定层次。因此，一个平台目标可能在并行的稳定分支上被分配到不同的层次。

### 21.2. 平台目标

对硬件平台的支持包括两个组成部分：内核支持和用户空间应用二进制接口（ABI）。内核平台支持包括运行 FreeBSD 内核所需的内容，例如机器相关的虚拟内存管理和设备驱动程序。用户空间 ABI 指定了用户进程与 FreeBSD 内核和基本系统库交互的接口。用户空间 ABI 包括系统调用接口、公共数据结构的布局和语义，以及传递给子例程的参数的布局和语义。ABI 的某些组成部分可能由规范定义，例如 C++ 异常对象的布局或 C 函数的调用约定。

FreeBSD 内核还使用 ABI（有时称为内核二进制接口（KBI）），其中包括公共数据结构的语义和布局，以及内核本身中公共函数的参数布局和语义。

FreeBSD 内核可能支持多个用户态 ABI。例如，FreeBSD 的 amd64 内核支持 FreeBSD amd64 和 i386 用户态 ABI 以及 Linux x86_64 和 i386 用户态 ABI。FreeBSD 内核应支持“本机”ABI 作为默认 ABI。本机“ABI”通常与内核 ABI 共享某些属性，如 C 调用约定，基本类型的大小等。

为内核和用户态 ABI 定义了层。在一般情况下，平台的内核和 FreeBSD ABI 被分配到同一层级。

#### 21.2.1. 层次 1: 完全支持的架构

层次 1 平台是最成熟的 FreeBSD 平台。它们由安全官员、发布工程团队和 Ports 管理团队支持。层次 1 架构在 FreeBSD 操作系统的所有方面，包括安装和开发环境，都应达到生产质量水平。

FreeBSD 项目向层次 1 平台的消费者提供以下保证：

- 官方 FreeBSD 发行镜像将由发布工程团队提供。
- 支持的版本将提供安全公告和勘误通知的二进制更新和源代码补丁。
- 支持的分支将提供安全公告的源代码补丁。
- 二进制更新和跨平台安全通知的源补丁通常会在公告发布时提供。
- 用户空间 ABI 的更改通常会包括兼容性层以确保对任何稳定分支编译的二进制文件的正确运行，其中平台为第一等级。这些兼容性层可能未在默认安装中启用。如果未针对 ABI 更改提供兼容性层，则会在发行说明中清楚地记录缺少兼容性层的情况。
- 内核 ABI 的某些部分的更改将包括兼容性层，以确保对在分支上编译的最旧支持版本的内核模块的正确操作。请注意，并非所有内核 ABI 的部分都受到保护。
- 官方二进制软件包将由 ports 团队提供。对于嵌入式架构，这些软件包可能是从不同架构交叉构建的。
- 大多数相关的 ports 应该要么构建，要么具有适当的过滤器以防止不适当的软件包构建。
- 不本质上特定于平台的新功能将在所有一级架构上完全功能化。
- 由较旧稳定分支编译的二进制文件使用的功能和兼容性填充可能会在较新的主要版本中删除。这些删除将在发行说明中清楚记录。
- 第 1 层平台应有完整的文档记录。基本操作将记录在 FreeBSD 手册中。
- 第 1 层平台将包含在源代码树中。
- 第一层平台应通过内部工具链或外部工具链自行托管。如果需要外部工具链，将提供外部工具链的官方二进制软件包。

为了保持第一层平台的成熟性，FreeBSD 项目将维护以下资源以支持开发：

- 在 FreeBSD.org 集群或其他方便所有开发人员访问的位置提供构建和测试自动化支持。嵌入式平台可以在 FreeBSD.org 集群中用模拟器替代实际硬件。
- 在 make universe 和 make tinderbox 目标中的包含。
- 在 FreeBSD 集群中的专用硬件，用于软件包构建（可以通过本地方式或通过 qemu-user）。

开发者需共同提供以下内容，以维持平台的一级支持状态：

- 更改源代码树不应故意破坏一级平台的构建。
- 一级架构必须拥有成熟健康的用户生态和活跃的开发者。
- 开发者应能够在常见的非嵌入式一级系统上构建软件包。这可能意味着如果针对该平台的非嵌入式系统普遍可用，则可以进行本地构建，或者可以在其他一级架构上进行交叉构建。
- 更改不能破坏用户空间 ABI。如果需要 ABI 更改，则应通过符号版本化或共享库版本提升来确保现有二进制文件的 ABI 兼容性。
- 合并到稳定分支的更改不能破坏内核 ABI 的受保护部分。如果需要内核 ABI 更改，则应修改更改以保留现有内核模块的功能。

#### 21.2.2. 第二层：开发和特定体系结构

第二层平台功能正常，但 FreeBSD 平台尚不成熟。它们未得到安全官员、发布工程和 Ports 管理团队的支持。

第二层平台可能是仍在积极开发中的第一层平台候选。达到生命周期结束的架构也可能从第一层状态转移到第二层状态，因为维持系统处于生产质量状态的资源可用性减少。良好支持的小众架构也可能是第二层平台。

FreeBSD 项目为第二层平台的消费者提供以下保证：

- 基础 ports 基础结构应包括足够支持构建 ports 和软件包的二级架构基础设施。这包括对 ports-mgmt/pkg 等基本软件包的支持，但不能保证能够构建或运行任意 ports。
- 如果不是特定于平台的新功能，应在所有二级架构上可行，如果没有实现。
- 二级平台将包含在源代码树中。
- Tier 2 平台应通过树内工具链或外部工具链自托管。如果需要外部工具链，则将提供官方二进制包。
- 即使未提供官方发行版，Tier 2 平台也应提供功能性内核和用户空间。

为了维护 Tier 2 平台的成熟度，FreeBSD 项目将维护以下资源以支持开发：

- make universe 和 make tinderbox 目标中的包含。

总的来说，开发人员需要提供以下内容以维持平台的二级状态：

- 对源代码树的更改不应故意破坏二级平台的构建。
- Tier 2 架构必须有活跃的用户和开发者社区。
- 虽然允许更改破坏用户空间 ABI，但不应该毫无必要地破坏 ABI。重要的用户空间 ABI 变更应该限制在主要版本中进行。
- 在尚未在 Tier 2 架构上实现的新功能中，应该提供一种在这些架构上禁用它们的方式。

#### 21.2.3. 第三层: 实验性架构

第三层平台至少具有部分的 FreeBSD 支持。它们不受到安全官员、发布工程以及 Ports 管理团队的支持。

第三层平台是处于早期开发阶段的架构，适用于非主流硬件平台，或被视为不太可能被广泛使用的传统系统。对于第三层平台的初始支持可能存在于一个独立的仓库中，而不是主要的源代码仓库。

FreeBSD 项目不对第三层平台的使用者提供任何保证，并且不承诺维护资源来支持开发。第三层平台可能无法始终构建，内核或用户空间的 ABI 也不被视为稳定。

#### 21.2.4. 不支持的架构

项目不以任何形式支持其他平台。项目以前将这些称为第四层系统。

当一个平台过渡到不受支持状态时，所有对该平台的支持将从源代码、ports 和文档树中移除。请注意，只要该平台在 ports 支持的分支中受到支持，ports 支持应继续保留。

### 21.3. 关于更改架构级别的政策

系统只能通过 FreeBSD 核心团队的批准从一个级别移动到另一个级别，核心团队应与安全官、发布工程和 ports 管理团队协作做出决定。为了将平台提升到更高的级别，任何缺失的支持保证必须在提升完成之前得到满足。

## 22. Ports 特定常见问题解答

### 22.1. 添加一个新的 Port

#### 22.1.1. 我如何添加一个新的 port？

往树中添加 port 相对简单。待 port 准备好添加，如下文所述，你需要在分类的 Makefile 中添加 port 的目录条目。在这个 Makefile 中，ports 按字母顺序列出并添加到 SUBDIR 变量中，如下所示：

```
	SUBDIR += newport
```

待 port 及其分类的 Makefile 准备就绪，新 port 就可以提交了：

```
% git add category/Makefile category/newport
% git commit
% git push
```

|     | 不要忘记按照这里的说明为 ports 树设置 git 钩子；已开发出特定钩子来验证分类的 Makefile。 |
| --- | --------------------------------------------------------------------------------------- |

#### 22.1.2. 当我添加一个新的 port 时，还有其他需要我知道的事情吗？

检查 port，最好确保它能正确编译和打包。

Porter 手册的测试章节包含更详细的说明。请参阅 Portclippy / Portfmt 和 poudriere 部分。

您不一定需要消除所有警告，但请确保您已经修复了简单的警告。

如果 port 来自一个之前没有为项目做出贡献的提交者，请将该人的姓名添加到 FreeBSD 贡献者列表的附加贡献者部分。

如果 port 作为 PR 提交。要关闭 PR，请将状态更改为 Issue Resolved ，并将解决方案更改为 Fixed 。

```
# make install
# make package
# make deinstall
# pkg add package you built above
# make deinstall
# make reinstall
# make package
```

注意，poudriere 是包构建的参考，如果 port 在 poudriere 中构建失败，将被移除。

### 22.2. 移除现有 Port

#### 22.2.1. 如何移除现有 port？

首先，请阅读关于存储库副本的部分。在移除 port 之前，您必须验证没有其他依赖于它的 ports。

- 确保 ports 中没有对 port 的依赖关系：

  - 在最近的 INDEX 文件中，port 的 PKGNAME 只出现在一行中。
  - | 没有其他 1001 包含在其 Makefiles 中任何对 1002 的目录或 PKGNAME 的引用。 |     |     |     | 使用 Git 时，考虑使用 git-grep(1)，它比 0 快得多。 |
    | ------------------------------------------------------------------------ | --- | --- | --- | -------------------------------------------------- |

- 然后，删除 1001:

  - 使用 git rm 删除 port 的文件和目录。
  - 在父目录 Makefile 中移除 port 的 SUBDIR 列表。
  - 在 ports/MOVED 中添加条目。
  - 从 port/ports 中删除 port（如果存在）。

或者，您可以使用 rmport 脚本，位于 ports/Tools/scripts。此脚本由 Vasil Dimov < vd@FreeBSD.org>编写。在向 FreeBSDports 邮件列表发送有关此脚本的问题时，请同时抄送当前维护者 Chris Rees < crees@FreeBSD.org>。

### 22.3. 如何将 port 移动到新位置？

1. 对 ports 进行彻底检查，查找旧 port 位置/名称的任何依赖项，并进行更新。仅在 INDEX 上运行 grep 是不够的，因为一些 ports 已启用编译时选项的依赖项。建议对 ports 进行完整的 git-grep(1) 检查。
2. 从旧类别 Makefile 中删除 SUBDIR 条目，并在新类别 Makefile 中添加 SUBDIR 条目。
3. 向 ports/MOVED 添加一个条目。
4. 在 ports/security/vuxml 中搜索 xml 文件中的条目，并相应地调整它们。特别检查具有新名称的先前软件包，其版本可能包含新的 port。
5. 移动 port 与 git mv 。
6. 提交更改。

### 如何将 port 复制到新位置？

1. 使用 cp -R old-cat/old-port new-cat/new-port 复制 port。
2. 将新 port 添加到 new-cat/Makefile。
3. 在 new-cat/new-port 中更改内容。
4. 提交更改。

### 22.5. Ports 冻结

#### 22.5.1. 什么是“ports 冻结”？

“ports 冻结”是在发布之前 ports 树处于的受限状态。它用于确保发布的软件包质量更高。通常持续几周。在此期间，修复构建问题并生成发布软件包。这一做法已不再使用，因为发布的软件包是从当前稳定的季度分支构建的。

要了解有关如何向季度分支合并提交的授权请求的更多信息，请参阅“提交合并请求到季度分支的程序是什么？”。

### 22.6. 季度分支

#### 22.6.1. 请求将提交合并到季度分支的授权程序是什么？

截至 2020 年 11 月 30 日，无需寻求明确批准即可提交到季度分支。

#### 22.6.2. 合并提交到季度分支的过程是什么？

将提交合并到季度分支（我们出于历史原因称之为 MFH 的过程）与在 src 仓库中进行 MFC 的提交非常相似，基本上：

```
% git checkout 2021Q2
% git cherry-pick -x $HASH
(verify everything is OK, for example by doing a build test)
% git push
```

其中 $HASH 是您要复制到季度分支的提交的哈希值。 -x 参数确保包含 main 分支的哈希 $HASH 在季度分支的新提交消息中。

### 创建一个新类别

#### 创建新类别的过程是什么？

请参阅《波特手册》中关于提议新类别的内容。一旦按照该过程操作，并且将 PR 分配给 Ports 管理团队 <portmgr@FreeBSD.org>，他们将决定是否批准。如果批准，他们将负责：

1. 执行任何必要的移动。（这仅适用于实体类别。）
2. 在 ports/Mk/bsd.port.mk 中更新 VALID_CATEGORIES 的定义。
3. 将 PR 重新分配给您。

#### 22.7.2. 实施新物理类别需要做什么？

1. 升级每个移动的 port 的 Makefile。暂时不连接新类别到构建。要做到这一点，您需要：

   1. 更改 port 的 CATEGORIES （这是练习的重点，记住吗？）新类别被列在第一位。这将有助于确保 PKGORIGIN 正确。
   2. 运行 make describe 。由于你将在几步后运行的顶级 make index 是对整个 ports 层次结构的 make describe ，在此处捕获任何错误将节省你以后不得不重新运行该步骤的时间。
   3. 如果你想非常彻底，现在可能是运行 portlint(1) 的好时机。

2. 检查 PKGORIGIN 是否正确。ports 系统使用每个 port 的 CATEGORIES 条目创建其 PKGORIGIN ，该 PKGORIGIN 用于将已安装的软件包连接到它们构建的 port 目录。如果此条目错误，常见的 port 工具如 pkg-version(8) 和 portupgrade(1) 将失败。为此，请使用 chkorigin.sh 工具： env PORTSDIR=/path/to/ports sh -e /path/to/ports/Tools/scripts/chkorigin.sh 。这将检查 ports 树中的每个 port，即使是那些未连接到构建的，所以你可以在移动操作后直接运行它。提示：不要忘记查看你刚刚移动的 ports 的任何从属 ports 的 PKGORIGIN ！
3. 在您自己的本地系统上测试所建议的更改：首先，在旧的 ports 的'categories' Makefiles 中注释掉 SUBDIR 条目；然后在 ports/Makefile 中启用构建新类别。在受影响的类别目录中运行 make checksubdirs 以检查 SUBDIR 条目。接下来，在 ports/目录中运行 make index。即使在现代系统上，这可能需要超过 40 分钟；但这是一个必要的步骤，以防止其他人出现问题。
4. 完成此操作后，您可以提交更新的 ports/Makefile 以将新类别连接到构建，并提交旧类别或类别的 Makefile 更改。
5. 向 ports/MOVED 添加适当的条目。
6. 更新文档，修改：

   - Porter's Handbook 中的类别列表

7. 只有在上述所有事项完成并且没有人再报告新 ports 有问题之后，才应从仓库中先前位置删除旧的 ports。

#### 22.7.3. 实施新虚拟类别需要做什么？

这比实体类别简单得多。只需要进行少量修改：

- 在 Porter's Handbook 中的类别列表

### 22.8. 杂项问题

#### 22.8.1. 是否有更改可以在不征求维护者批准的情况下提交？

大多数 ports 的全面批准适用于这些类型的修复：

- 对 port 进行大多数基础设施更改（即现代化，但不更改功能）。例如，覆盖范围包括转换为新的 USES 宏，启用详细生成，并切换到新的 ports 系统语法。
- 琐碎且经过测试的构建和运行时修复。
- 文档或元数据的更改 ports，例如 pkg-descr 或 COMMENT 。

|     | 例外情况包括由 Ports 管理团队 <portmgr@FreeBSD.org>或安全官团队 <security-officer@FreeBSD.org> 维护的任何内容。这些团队维护的 ports 不得进行未授权的提交。 |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |

#### 22.8.2. 我如何知道我的 port 是否构建正确？

软件包每周构建多次。如果 port 构建失败，维护者将收到来自 pkg-fallout@FreeBSD.org 的电子邮件。

所有软件包构建（官方的，实验性的，和非回归）的报告都聚合在 pkg-status.FreeBSD.org。

#### 22.8.3. 我添加了一个新的 port。我需要将它添加到 INDEX 中吗？

不需要。该文件可以通过运行 make index 生成，或者可以通过 make fetchindex 下载预生成的版本。

#### 22.8.4. 我还有其他不允许触碰的文件吗？

直接位于 ports/ 下的任何文件，或者以大写字母开头的子目录中的任何文件（如 Mk/、Tools/ 等）。特别是，Ports 管理团队 < portmgr@FreeBSD.org> 对 ports/Mk/bsd.port*.mk 文件非常保护，所以不要提交对这些文件的更改，除非你想面对他们的愤怒。

#### 22.8.5. 当 port distfile 的校验和在文件变化时没有版本更改时，更新的正确过程是什么？

当由于作者在不更改 port 修订版的情况下更新文件而更新分发文件的校验和时，提交消息会包含原始文件和新分发文件之间相关差异的摘要，以确保分发文件没有被损坏或恶意更改。如果当前版本的 port 在 ports 树上已经存在一段时间，旧分发文件通常会在 ftp 服务器上可用；否则，应联系作者或维护者以了解为什么分发文件发生了变化。

#### 22.8.6.如何请求 ports 树的实验性测试构建（exp-run）？

在提交具有重大 ports 影响的补丁之前，必须完成一次实验运行（exp-run）。该补丁可以针对 ports 树或基本系统。

完整软件包构建将使用提交者提供的补丁，并要求提交者在提交前修复检测到的问题（故障）。

1. 转到 Bugzilla 的新 PR 页面。
2. 选择您的补丁涉及的产品。
3. 像往常一样填写 bug 报告。记得附上补丁。
4. 如果顶部显示“显示高级字段”，请点击它。它现在会显示“隐藏高级字段”。许多新字段将会可用。如果它已经显示“隐藏高级字段”，则不需要做任何操作。
5. 在“Flags”部分，将“exp-run”设置为 ? 。至于所有其他字段，将鼠标悬停在任何字段上会显示更多详细信息。
6. 提交。等待构建运行。
7. Ports 管理团队 <portmgr@FreeBSD.org> 将回复可能的后果。
8. 根据后果的情况：

   - 如果没有后果，程序就到此为止，变更可以提交，等待任何其他所需的批准。

     1. 如果有后果，必须加以修复，可以直接在 ports 树中修复 ports，或者将其添加到提交的补丁中。
     2. 当这样做时，回到第 6 步，表示后果已经修复，并等待再次运行 exp-run。只要存在损坏的 ports，就重复此过程。

## 23. 开发人员不是提交者的特定问题

在 FreeBSD 机器上有一些人没有提交权限。本文档几乎适用于这些开发人员（除了与提交和随之而来的邮件列表成员资格相关的内容）。特别是，我们建议您阅读：

- [ 行政细节](https://docs.freebsd.org/en/articles/committers-guide/#admin)
- | 为每个人 |     | 请您的导师将您添加到“额外贡献者”（doc/shared/contrib-additional.adoc），如果您尚未在其中列出。 |
  | -------- | --- | ---------------------------------------------------------------------------------------------- |
- [ 开发者关系](https://docs.freebsd.org/en/articles/committers-guide/#developer.relations)
- [ SSH 快速入门指南](https://docs.freebsd.org/en/articles/committers-guide/#ssh.guide)
- [FreeBSD 提交者的规则大列表](https://docs.freebsd.org/en/articles/committers-guide/#rules)

## 24. 关于 Google Analytics 的信息

截至 2012 年 12 月 12 日，FreeBSD 项目网站启用了 Google Analytics 以收集有关网站使用情况的匿名统计数据。

|     | 截至 2022 年 3 月 3 日，FreeBSD 项目已移除 Google Analytics。 |
| --- | ------------------------------------------------------------- |

## 25. 杂项问题

### 如何访问 people.FreeBSD.org 以发布个人或项目信息？

people.FreeBSD.org 和 freefall.FreeBSD.org 一样。只需创建一个 public_html 目录。您放置在该目录中的任何内容将自动显示在 https://people.FreeBSD.org/下。

### 邮件列表存档存储在哪里？

邮件列表存档在 freefall.FreeBSD.org 的/local/mail 下。

### 25.3. 我想指导新的 committer。我需要遵循什么流程？

查看内部页面上的新帐户创建流程文档。

## 26. FreeBSD 贡献者的福利和津贴

### 26.1. 认可

作为一名胜任的软件工程师的认可是最持久的价值。此外，有机会与一些每个工程师都梦想见到的最优秀的人一起工作是一个巨大的额外福利！

### 26.2. FreeBSD 商城

FreeBSD 提交者可以在会议上从 FreeBSD 商城免费获取包含四张 CD 或 DVD 的套装。

### 26.3. `Gandi.net`

Gandi 提供网站托管、云计算、域名注册和 X.509 证书服务。

Gandi 为所有 FreeBSD 开发者提供 E-rate 折扣。为了简化获得折扣的过程，首先设置 Gandi 账户，填写账单信息并选择货币。然后使用您的 @freebsd.org 邮件地址发送邮件至 non-profit@gandi.net，并指明您的 Gandi 账号。

### 26.4. `rsync.net`

rsync.net 为 UNIX 用户提供优化的云存储服务，完全基于 FreeBSD 和 ZFS 运行。

rsync.net 为 FreeBSD 开发者提供永久免费的 500 GB 账户。只需使用您的 @freebsd.org 地址在 https://www.rsync.net/freebsd.html 注册即可获得此免费账户。
