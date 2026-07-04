# 提交者指南

- 原文地址：[Committer's Guide](https://docs.freebsd.org/en/articles/committers-guide/)

## 摘要

本文档为 FreeBSD 提交者社区提供信息。所有新的提交者在开始之前应该阅读本文档，现有的提交者也强烈建议定期回顾。

几乎所有的 FreeBSD 开发者都有一个或多个代码库的提交权限。然而，也有一些开发者没有权限，文中某些内容同样适用于他们。（例如，有些人只有权限操作问题报告数据库。）有关更多信息，请参见[非提交者特定问题](https://docs.freebsd.org/en/articles/committers-guide/#non-committers)。

本文档也可能对希望了解 FreeBSD 项目运作方式的社区成员有所帮助。



## 1. 管理详细信息

| *登录方式*              | [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html)，仅支持协议 2                                                                                                                                                                                                                                         |
| ------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| *主要 Shell 主机*       | `freefall.FreeBSD.org`                                                                                                                                                                                                                                                                                                        |
| *参考机器*              | `ref*.FreeBSD.org`，`universe*.freeBSD.org`（请参见 [FreeBSD 项目主机](https://www.freebsd.org/internal/machines/)）                                                                                                                                                                                                                    |
| *SMTP 主机*           | `smtp.FreeBSD.org:587`（请参见 [SMTP 访问设置](https://docs.freebsd.org/en/articles/committers-guide/#smtp-setup)）                                                                                                                                                                                                                    |
| `src/` Git 仓库       | `ssh://git@gitrepo.FreeBSD.org/src.git`                                                                                                                                                                                                                                                                                       |
| `doc/` Git 仓库       | `ssh://git@gitrepo.FreeBSD.org/doc.git`                                                                                                                                                                                                                                                                                       |
| `ports/` Git 仓库     | `ssh://git@gitrepo.FreeBSD.org/ports.git`                                                                                                                                                                                                                                                                                     |
| *内部邮件列表*            | developers（技术上称为 all-developers），doc-developers，doc-committers，ports-developers，ports-committers，src-developers，src-committers。（每个项目的仓库都有自己的 -developers 和 -committers 邮件列表。可以在 `freefall.FreeBSD.org` 的 **/local/mail/repository-name-developers-archive** 和 **/local/mail/repository-name-committers-archive** 中找到这些列表的归档。） |
| *核心团队月度报告*          | `FreeBSD.org` 集群上的 **/home/core/public/reports**                                                                                                                                                                                                                                                                              |
| *Ports 管理团队月度报告*    | `FreeBSD.org` 集群上的 **/home/portmgr/public/monthly-reports**                                                                                                                                                                                                                                                                   |
| *重要的 `src/` Git 分支* | `stable/n`（`n`-STABLE），`main`（-CURRENT）                                                                                                                                                                                                                                                                                       |

连接到项目主机需要使用 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html)。有关更多信息，请参见 [SSH 快速入门指南](https://docs.freebsd.org/en/articles/committers-guide/#ssh.guide)。

有用的链接：

- [FreeBSD 项目内部页面](https://www.freebsd.org/internal/)
- [FreeBSD 项目主机](https://www.freebsd.org/internal/machines/)
- [FreeBSD 项目管理组](https://www.freebsd.org/administration/)

## 2. FreeBSD 的 OpenPGP 密钥

FreeBSD 项目使用符合 OpenPGP（*Pretty Good Privacy*）标准的加密密钥来验证提交者的身份。携带重要信息（如公钥）的消息可以使用 OpenPGP 密钥进行签名，以证明它们确实来自提交者。有关更多信息，请参见 [Michael Lucas 的《PGP 和 GPG：实用偏执的电子邮件》](https://nostarch.com/releases/pgp_release.pdf) 和 [维基百科上的 PGP 介绍](https://en.wikipedia.org/wiki/Pretty_Good_Privacy)。


### 2.1. 创建密钥

可以使用现有的密钥，但应先通过 **documentation/tools/checkkey.sh** 检查。此时，确保密钥具有 FreeBSD 用户 ID。

对于那些还没有 OpenPGP 密钥，或者需要新的密钥来符合 FreeBSD 安全要求的人，以下是如何生成一个密钥的步骤。

1. 安装 **security/gnupg**。在 **~/.gnupg/gpg.conf** 中输入以下内容，以设置签名和新密钥的最低默认首选项（有关更多详细信息，请参见 [GnuPG 选项文档](https://www.gnupg.org/documentation/manuals/gnupg/GPG-Options.html)）：

   ```sh
   # 按优先顺序列出用于签名的首选算法（从强到弱）
   personal-digest-preferences SHA512 SHA384 SHA256 SHA224
   # 新密钥的默认首选项
   default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 CAMELLIA256 AES192 CAMELLIA192 AES CAMELLIA128 CAST5 BZIP2 ZLIB ZIP Uncompressed
   ```

2. 生成密钥：

   ```sh
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
   What keysize do you want? (2048) 2048 ①
   Requested keysize is 2048 bits
   Please specify how long the key should be valid.
   	 0 = key does not expire
         <n>  = key expires in n days
         <n>w = key expires in n weeks
         <n>m = key expires in n months
         <n>y = key expires in n years
   Key is valid for? (0) 3y ②
   Key expires at Wed Nov  4 17:20:20 2015 MST
   Is this correct? (y/N) y
   GnuPG needs to construct a user ID to identify your key.

   Real name: Chucky Daemon ③
   Email address: notreal@example.com
   Comment:
   You selected this USER-ID:
   "Chucky Daemon <notreal@example.com>"

   Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
   You need a Passphrase to protect your secret key.
   ```

- ① 2048 位密钥，三年有效期目前提供足够的保护（2022-10）。
- ② 三年的密钥有效期足够短，以使得计算能力增强后不再使用过时的密钥，但又足够长，以减少密钥管理问题。
- ③ 在此处使用真实姓名，最好与政府签发的 ID 上显示的姓名匹配，以便其他人更容易验证你的身份。在 `Comment` 部分可以输入一些帮助别人识别你的文字。

输入电子邮件地址后，会请求设置密码短语。创建安全密码短语的方法存在争议。为了避免建议一种单一的方式，以下是一些描述不同方法的链接：[https://world.std.com/~reinhold/diceware.html](https://world.std.com/~reinhold/diceware.html)，[https://www.iusmentis.com/security/passphrasefaq/](https://www.iusmentis.com/security/passphrasefaq/)，[https://xkcd.com/936/](https://xkcd.com/936/)，[https://en.wikipedia.org/wiki/Passphrase](https://en.wikipedia.org/wiki/Passphrase)。

保护好私钥和密码短语。如果私钥或密码短语可能已被泄露或披露，请立即通知 [accounts@FreeBSD.org](mailto:accounts@FreeBSD.org) 并撤销密钥。

新密钥的提交步骤请参见[新提交者步骤](https://docs.freebsd.org/en/articles/committers-guide/#commit-steps)。

## 3. Kerberos 和 LDAP Web 密码用于 FreeBSD 集群

FreeBSD 集群需要 Kerberos 密码来访问某些服务。由于 LDAP 在集群中代理 Kerberos，Kerberos 密码也作为 LDAP Web 密码。一些需要此密码的服务包括：

- [Bugzilla](https://bugs.freebsd.org/bugzilla)

要在 FreeBSD 集群中创建一个新的 Kerberos 账户，或使用随机密码生成器重置现有账户的 Kerberos 密码：

```sh
% ssh kpasswd.freebsd.org
```

>**注意**
>
>必须从 FreeBSD.org 集群外部的机器执行此操作。

也可以通过登录 `freefall.FreeBSD.org` 并运行以下命令手动设置 Kerberos 密码：

```sh
% kpasswd
```

>**注意**
>
> 如果之前未使用过 FreeBSD.org 集群的 Kerberos 认证服务，则会显示 `Client unknown` 错误。此错误意味着必须先使用上面显示的 `ssh kpasswd.freebsd.org` 方法初始化 Kerberos 账户。

## 4. 提交权限类型

FreeBSD 仓库包含多个组件，这些组件合并后支持基本操作系统源代码、文档、第三方应用 Port 基础设施和各种维护的工具。当分配 FreeBSD 提交权限时，会指定可以使用该权限的树区域。通常，权限所在的区域反映了谁授权分配该提交权限。以后可能会添加额外的权限区域：当这种情况发生时，提交者应遵循该区域的正常提交权限分配程序，向相关实体寻求批准，并可能在该区域获得一段时间的导师支持。

| *提交者类型* | *负责*     | *树区域组件*             |
| ------- | -------- | ------------------- |
| src     | srcmgr@  | src/                |
| doc     | doceng@  | doc/，ports/，src/ 文档 |
| ports   | portmgr@ | ports/              |

在开发区域权限概念之前分配的提交权限可能适用于树的许多部分。然而，常识表明，提交者如果未曾在某个区域工作过，应在提交前寻求审查，向适当的责任方寻求批准，并/或与导师合作。由于不同区域的代码维护规则不同，这既有助于提交者在不熟悉的区域工作，也有助于树上的其他人。

鼓励提交者在开发过程中寻求审查，无论工作发生在哪个区域。

### 4.1. 提交者在其他树区域活动的政策

- 所有提交者都可以修改 **src/share/misc/committers-*.dot**，**src/usr.bin/calendar/calendars/calendar.freebsd** 和 **ports/astro/xearth/files**。
- doc 提交者可以在没有 src 提交者批准的情况下提交文档更改，例如手册页、README 文件、fortune 数据库、日历文件以及注释修复，前提是提交时遵循正常的审查和管理程序。
- 所有提交者都可以在拥有适当权限的非导师提交者批准的情况下对一切其他区域进行更改。导师提交者可以提供 `Reviewed by` 但不能提供 `Approved by`。
- 提交者可以通过通常的流程获得附加的权限，寻找导师并由导师向 core、doceng 或 portmgr 提交提案，获批后将被加入到 `access` 并进入正常的导师期，在此期间会持续提供 `Approved by`。

#### 4.1.1. 文档隐式（普遍）批准

某些类型的修复被文档工程团队（[doceng@FreeBSD.org](mailto:doceng@FreeBSD.org)）授予了“普遍批准”，允许所有提交者修复这些类别的问题，而不需要文档提交者的批准或审查。

普遍批准适用于以下类型的修复：

- 错别字
- 微小修复
  诸如标点、URL、日期、路径和文件名的错误信息等常见错误，这些错误可能使读者困惑。

多年来，某些隐式批准已经在文档树中获得。这些最常见的情况包括：

- 更改 **documentation/content/en/books/porters-handbook/versions/_index.adoc**
  [__FreeBSD_version 值（Porter's Handbook）](https://docs.freebsd.org/en/books/porters-handbook/versions/)，主要用于 src 提交者。
- 更改 **doc/shared/contrib-additional.adoc**
  [Additional FreeBSD Contributors](https://docs.freebsd.org/en/articles/contributors/#contrib-additional) 维护。
- 所有 [新提交者步骤](https://docs.freebsd.org/en/articles/committers-guide/#commit-steps)，文档相关。
- 安全通告；错误通告；发布；
  由安全官团队 [security-officer@FreeBSD.org](mailto:security-officer@FreeBSD.org) 和发布工程团队 [re@FreeBSD.org](mailto:re@FreeBSD.org) 使用。
- 更改 **website/content/en/donations/donors.adoc**
  用于捐赠联络办公室 [donations@FreeBSD.org](mailto:donations@FreeBSD.org)。

在提交前，必须进行构建测试；有关更多信息，请参见 [FreeBSD 文档项目初学者指南](https://docs.freebsd.org/en/books/fdp-primer/) 中的“概述”和“FreeBSD 文档构建过程”部分。

## 5. Git 入门

### 5.1. Git 基础

当搜索 “Git Primer” 时，会出现很多优秀的资源。Daniel Miessler 的 [A git primer](https://danielmiessler.com/study/git/) 和 Willie Willus 的 [Git - Quick Primer](https://gist.github.com/williewillus/068e9a8543de3a7ef80adb2938657b6b) 都是很好的概述。Git 书籍虽然更长一些，但也非常完整：[https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2)。如果你需要关于 Git 的常见问题和陷阱的指导，可以参考这个网站 [https://dangitgit.com/](https://dangitgit.com/)。最后，一篇面向计算机科学家的 [Git 介绍](https://eagain.net/articles/git-for-computer-scientists/) 也对一些人解释 Git 的世界观有所帮助。

本文档假设你已经阅读过这些内容，并尽量不重复基础知识（尽管会简要介绍）。

### 5.2. Git 简明入门

这份入门指南的范围比旧的 Subversion 入门指南小，但应该涵盖基本的操作。

#### 5.2.1. 范围

如果你想下载 FreeBSD，编译源代码，并通过这种方式保持更新，那么这份指南适合你。它涵盖了获取源代码、更新源代码、二分搜索，并简要讲解如何处理一些本地更改。它主要覆盖基础内容，并在需要时提供深入学习的好指引。当读者觉得基础内容不足时，其他部分的指南会涉及更高级的贡献主题。

本节的目标是突出 Git 中跟踪源代码所需的功能。假设你对 Git 有基本了解。互联网上有许多 Git 入门资料，但 [Git 书籍](https://git-scm.com/book/en/v2) 提供了较为全面的介绍。

#### 5.2.2. 开发者入门

本节介绍了提交者如何推送开发者或贡献者的提交。

##### 5.2.2.1. 日常使用

>**注意**
>
>以下示例中，替换 `${repo}` 为所需的 FreeBSD 仓库名称：`doc`、`ports` 或 `src`。

- 克隆仓库：

  ```sh
  % git clone -o freebsd --config remote.freebsd.fetch='+refs/notes/*:refs/notes/*' https://git.freebsd.org/${repo}.git
  ```

  然后你应该会看到官方镜像作为远程源：

  ```sh
  % git remote -v
  freebsd  https://git.freebsd.org/${repo}.git (fetch)
  freebsd  https://git.freebsd.org/${repo}.git (push)
  ```

- 配置 FreeBSD 提交者数据：
  在 `repo.freebsd.org` 上的提交钩子会检查 `Commit` 字段是否与 FreeBSD.org 上的提交者信息匹配。最简单的方式是通过执行 **/usr/local/bin/gen-gitconfig.sh** 脚本获取推荐的配置：

  ```sh
  % gen-gitconfig.sh
  [...]
  % git config user.name (your name in gecos)
  % git config user.email (your login)@FreeBSD.org
  ```

- 设置推送 URL：

  ```sh
  % git remote set-url --push freebsd git@gitrepo.freebsd.org:${repo}.git
  ```

  然后你应该会看到分开的 fetch 和 push URL，这是最有效的配置：

  ```sh
  % git remote -v
  freebsd  https://git.freebsd.org/${repo}.git (fetch)
  freebsd  git@gitrepo.freebsd.org:${repo}.git (push)
  ```

  再次注意，`gitrepo.freebsd.org` 已被规范化为 `repo.freebsd.org`。

- 安装提交消息模板钩子：
  对于 doc 仓库：

  ```sh
  % cd .git/hooks
  % ln -s ../../.hooks/prepare-commit-msg
  ```

  对于 ports 仓库：

  ```sh
  % git config --add core.hooksPath .hooks
  ```

  对于 src 仓库：

  ```sh
  % cd .git/hooks
  % ln -s ../../tools/tools/git/hooks/prepare-commit-msg
  ```

##### 5.2.2.2. `admin` 分支

`access` 和 `mentors` 文件存储在每个仓库的孤立分支 `internal/admin` 中。

以下是如何将 `internal/admin` 分支检出到本地名为 `admin` 的分支的示例：

```sh
% git config --add remote.freebsd.fetch '+refs/internal/*:refs/internal/*'
% git fetch
% git checkout -b admin internal/admin
```

或者，你可以为 `admin` 分支添加一个工作树：

```sh
git worktree add -b admin ../${repo}-admin internal/admin
```

要在 Web 上浏览 `internal/admin` 分支：[https://cgit.freebsd.org/${repo}/log/?h=internal/admin](https://cgit.freebsd.org/${repo}/log/?h=internal/admin)

推送时，指定完整的 refspec：

```sh
git push freebsd HEAD:refs/internal/admin
```

#### 5.2.3. 保持与 FreeBSD src 树同步

第一步：克隆树。此操作将下载整个树。下载有两种方式。大多数人会选择对仓库进行深度克隆。但是，有时你可能希望进行浅克隆。

##### 5.2.3.1. 分支名称

FreeBSD-CURRENT 使用 `main` 分支。

`main` 是默认分支。

对于 FreeBSD-STABLE，分支名称包括 `stable/12` 和 `stable/13`。

对于 FreeBSD-RELEASE，发布工程分支名称包括 `releng/12.4` 和 `releng/13.2`。

[https://www.freebsd.org/releng/](https://www.freebsd.org/releng/) 上显示：

- `main` 和 `stable/⋯` 分支是开放的
- `releng/⋯` 分支在发布标签时会被冻结。

示例：

- 在 [releng/13.1](https://cgit.freebsd.org/src/log/?h=releng/13.1) 分支上标记 [release/13.1.0](https://cgit.freebsd.org/src/tag/?h=release/13.1.0)
- 在 [releng/13.2](https://cgit.freebsd.org/src/log/?h=releng/13.2) 分支上标记 [release/13.2.0](https://cgit.freebsd.org/src/tag/?h=release/13.2.0)。

##### 5.2.3.2. 仓库

请参见 [管理详情](https://docs.freebsd.org/en/articles/committers-guide/#admin) 获取最新的 FreeBSD 源代码获取地址。`$URL` 可以从该页面获取。

注意：该项目不使用子模块，因为子模块与我们的工作流程和开发模型不太契合。如何跟踪第三方应用程序的变化在其他地方有讨论，通常对普通用户来说并不重要。

##### 5.2.3.3. 深度克隆

深度克隆会拉取整个树，以及所有历史和分支。这是最容易操作的方式。它还允许你使用 Git 的 worktree 功能，将所有活跃分支签出到不同的目录，但只有一个仓库副本。

```sh
% git clone -o freebsd $URL -b branch [<directory>]
```

— 这将创建一个深度克隆。`branch` 应该是前面列出的分支之一。如果没有指定 `branch`，将使用默认分支 (`main`)。如果没有指定 `<directory>`，新目录的名称将与仓库的名称（**doc**、**ports** 或 **src**）匹配。

如果你对历史感兴趣，计划进行本地更改，或者计划在多个分支上工作，那么你会需要一个深度克隆。它也是最容易保持更新的方式。如果你对历史有兴趣，但只在一个分支上工作并且空间有限，你还可以使用 `--single-branch` 仅下载一个分支（尽管一些合并提交将不会引用已合并的分支，这对某些关注详细历史的用户可能很重要）。

##### 5.2.3.4. 浅克隆

浅克隆仅复制当前的代码，而不会下载历史。这在你需要构建特定版本的 FreeBSD 时非常有用，或者当你刚开始并计划全面跟踪树时。你还可以使用它来限制历史版本的数量。然而，下面会提到这种方法的一个重要限制。

```sh
% git clone -o freebsd -b branch --depth 1 $URL [dir]
```

这将克隆仓库，但只包含最新版本，其余历史不会下载。如果你之后改变主意，可以使用 `git fetch --unshallow` 来获取旧的历史。

>**警告**
>
>当你进行浅克隆时，`uname` 输出中将不会显示提交计数。这可能使得在发布安全公告时，判断系统是否需要更新变得更困难。

##### 5.2.3.5. 构建

下载完成后，构建过程按照手册中的描述进行，例如：

```sh
% cd src
% make buildworld
% make buildkernel
% make installkernel
% make installworld
```

因此，这里不再深入讨论。

如果你想构建自定义内核，FreeBSD 手册中的 [内核配置部分](https://docs.freebsd.org/en/books/handbook/#kernelconfig) 建议在 **sys/${ARCH}/conf** 下创建一个文件 MYKERNEL，针对 GENERIC 进行修改。为了让 Git 忽略 MYKERNEL，可以将它添加到 `.git/info/exclude`。

##### 5.2.3.6. 更新

更新两种类型的树使用相同的命令。这会拉取自上次更新以来的所有修改。

```sh
% git pull --ff-only
```

这将更新树。在 Git 中，"fast forward" 合并是指仅需要设置新的分支指针，而不需要重新创建提交。通过始终执行快速前进合并/拉取，你将确保拥有 FreeBSD 树的准确副本。如果你希望维护本地补丁，这一点非常重要。

见下文如何管理本地更改。最简单的方法是使用 `--autostash` 选项与 `git pull` 命令一起使用，但也有更复杂的选项可用。

#### 5.2.4. 选择特定版本

在 Git 中，`git checkout` 用于检出分支和特定版本。Git 的版本是长哈希值，而不是顺序编号。

当你检出特定版本时，只需在命令行中指定你想要的哈希值（`git log` 命令可以帮助你决定要选择哪个哈希值）：

```
% git checkout 08b8197a74
```

然后你就会检出该版本。你将看到类似如下的消息：

```sh
Note: checking out '08b8197a742a96964d2924391bf9fdfeb788865d'.

You are in a 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 08b8197a742a hook gpiokeys.4 to the build
```

其中最后一行是由你检出的哈希值生成的，提交消息的第一行来自该版本的提交。哈希值可以缩写为最短的唯一长度。Git 本身对于显示的数字长度并不一致。

#### 5.2.5. 二分查找（Bisect）

有时，事情出了问题。最后一个版本工作正常，但你刚刚更新的版本不行。开发人员可能会要求你通过二分查找来追踪是哪个提交导致了回归问题。

Git 通过强大的 `git bisect` 命令使得二分查找变得容易。以下是如何使用它的简要概述。你可以查看 [https://www.metaltoad.com/blog/beginners-guide-git-bisect-process-elimination](https://www.metaltoad.com/blog/beginners-guide-git-bisect-process-elimination) 或 [https://git-scm.com/docs/git-bisect](https://git-scm.com/docs/git-bisect) 获取更多详细信息。`git bisect` 手册页面非常适合描述可能出错的情况，如何处理无法构建的版本，何时使用除 'good' 和 'bad' 之外的术语等，这里不再涉及。

`git bisect start --first-parent` 将开始二分查找过程。接下来，你需要告诉 Git 要遍历的版本范围。`git bisect good XXXXXX` 会告诉它工作版本，`git bisect bad XXXXX` 会告诉它坏版本。坏版本通常是 HEAD（即你当前检出的版本）。好的版本是你上次检出的版本。`--first-parent` 参数是必需的，以确保后续的 `git bisect` 命令不会试图检出缺少完整 FreeBSD 源代码树的供应商分支。

>**技巧**
>
> 如果你想知道上次检出的版本，可以使用
>
>```sh
>5ef0bd68b515 (HEAD -> main, freebsd/main, freebsd/HEAD) HEAD@{0}: pull --ff-only: Fast-forward
>a8163e165c5b (upstream/main) HEAD@{1}: checkout: moving from b6fb97efb682994f59b21fe4efb3fcfc0e5b9eeb to main
>...
>```
>
>显示我将工作树移动到 `main` 分支（a816…），然后从上游更新（到 5ef0…）。在这种情况下，坏版本是 HEAD（或 5ef0bd68b515），好版本是 a8163e165c5b。正如你从输出中看到的，`HEAD@{1}` 通常也有效，但如果在更新后做了其他操作再发现需要进行二分查找时，它并不总是可靠的。

首先设置“好”版本，然后设置“坏”版本（尽管顺序无关紧要）。当你设置坏版本时，它会提供有关过程的统计信息：

```sh
% git bisect start --first-parent
% git bisect good a8163e165c5b
% git bisect bad HEAD
Bisecting: 1722 revisions left to test after this (roughly 11 steps)
[c427b3158fd8225f6afc09e7e6f62326f9e4de7e] Fixup r361997 by balancing parens.  Duh.
```

然后，你需要构建并安装该版本。如果它是好的，输入 `git bisect good`；如果它是坏的，输入 `git bisect bad`。如果该版本无法编译，输入 `git bisect skip`。每一步之后你都会看到类似的消息。完成后，将坏版本报告给开发人员（或自己修复 bug 并提交补丁）。`git bisect reset` 会结束该过程，并将你带回到你开始的地方（通常是 `main` 的顶端）。同样，`git bisect` 手册（如上所述）是处理错误或特殊情况时的良好资源。

#### 5.2.6. 使用 GnuPG 签名提交、标签和推送

Git 知道如何签名提交、标签和推送。当你签名 Git 提交或标签时，你可以证明提交的代码来自你，并且在传输过程中没有被更改。你还可以证明是你提交了代码，而不是别人。

关于签名提交和标签的更详细文档可以在 [Git Tools - Signing Your Work](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work) 章节中找到。

签名推送的原因可以在 [引入此功能的提交](https://github.com/git/git/commit/a85b377d0419a9dfaca8af2320cc33b051cbed04) 中找到。

最好的方法是告诉 Git，你总是希望签名提交、标签和推送。你可以通过设置以下几个配置变量来实现：

```sh
% git config --add user.signingKey LONG-KEY-ID
% git config --add commit.gpgSign true
% git config --add tag.gpgSign true
% git config --add push.gpgSign if-asked
```

>**注意**
>
> 为了避免可能的冲突，确保给 Git 提供一个长密钥 ID。你可以通过以下命令获得长 ID：`gpg --list-secret-keys --keyid-format LONG`。

>**技巧**
>
> 如果要使用特定的子密钥，而不让 GnuPG 解析为主密钥，可以在密钥后加上 `!`。例如，要使用子密钥 `DEADBEEF` 进行加密，使用 `DEADBEEF!`。

#### 5.2.6.1. 验证签名

提交签名可以通过运行 `git verify-commit <commit hash>` 或 `git log --show-signature` 来验证。

标签签名可以通过 `git verify-tag <tag name>` 或 `git tag -v <tag name>` 来验证。

#### 5.2.7. Ports 考虑事项

ports 树的操作方式相同，分支名称不同，且仓库的位置也不同。

用于浏览器的 cgit 仓库 Web 界面位于 [https://cgit.FreeBSD.org/ports/](https://cgit.FreeBSD.org/ports/)。生产 Git 仓库位于 [https://git.FreeBSD.org/ports.git](https://git.FreeBSD.org/ports.git) 和 ssh://anongit@git.FreeBSD.org/ports.git（或 `anongit@git.FreeBSD.org:ports.git`）。

GitHub 上也有一个镜像，查看 [外部镜像](https://docs.freebsd.org/en/books/handbook/mirrors/#mirrors) 以了解概况。*最新*分支是 `main`。*季度*分支命名为 `yyyyQn`，其中 `yyyy` 为年份，`n` 为季度。

##### 5.2.7.1. 提交信息格式

在 ports 仓库中，有一个钩子帮助你编写提交信息，位于 [.hooks/prepare-commit-message](https://cgit.freebsd.org/ports/tree/.hooks/prepare-commit-msg)。你可以通过运行 `git config --add core.hooksPath .hooks` 来启用它。

提交信息应格式化如下：

```sh
category/port: 总结。

描述你为什么做成这些修改。

PR:      12345
```

>**重要**
>
> 第一行是提交的主题，包含了更改的 port 以及提交的总结。它应该包含 50 个字符或更少。
>
>提交信息的其余部分应该在 72 个字符的边界处换行。
>
>应该有一个空行将它与提交信息的其余部分分开。
>
>如果有任何元数据字段，它们应该与提交信息之间有另一个空行，以便容易区分。

#### 5.2.8. 管理本地更改

本节讨论跟踪本地更改。如果你没有本地更改，可以跳过本节。

重要的一点是：所有的更改在推送之前都是本地的。与 Subversion 不同，Git 使用的是分布式模型。对于用户而言，大多数操作几乎没有区别。然而，如果你有本地更改，你可以使用相同的工具来管理它们，就像你使用相同的工具来拉取 FreeBSD 的更改一样。所有你尚未推送的更改都是本地的，可以轻松修改（如 `git rebase`，稍后会讨论）。

##### 5.2.8.1. 保留本地更改

保留本地更改（尤其是微小更改）最简单的方法是使用 `git stash`。在最简单的形式下，你可以使用 `git stash` 来记录更改（这会将它们推送到暂存栈）。大多数人使用它来在更新树之前保存更改。然后，他们使用 `git stash apply` 将它们重新应用到树上。stash 是一个更改栈，可以通过 `git stash list` 查看。更多细节可以参考 `git-stash` 手册页 ([https://git-scm.com/docs/git-stash](https://git-scm.com/docs/git-stash))。

这种方法适用于你对树所做的小改动。当你有一些不那么微小的更改时，最好还是保留一个本地分支并进行 rebase。`git stash` 也与 `git pull` 命令集成：只需在命令行上添加 `--autostash` 即可。

##### 5.2.8.2. 保留本地分支

与 Subversion 不同，Git 保留本地分支更为简便。在 Subversion 中，你需要合并提交并解决冲突。虽然这可以管理，但可能导致历史记录变得复杂，在上游提交时可能困难重重，或者如果你需要重新创建该过程时也很麻烦。Git 也允许合并，存在相同的问题。这是管理分支的一种方式，但它的灵活性最差。

除了合并，Git 还支持“变基”的概念，这可以避免这些问题。`git rebase` 命令会将一个分支的所有提交重新播放到父分支的新位置。我们将覆盖使用它时最常见的场景。

###### 5.2.8.2.1. 创建分支

假设你想对 FreeBSD 的 `ls` 命令做一个更改，让它永远不显示颜色。做这个更改有很多原因，但我们用这个作为示例。FreeBSD 的 `ls` 命令会时不时地更新，你需要应对这些变化。幸运的是，使用 Git rebase 通常是自动完成的。

```sh
% cd src
% git checkout main
% git checkout -b no-color-ls
% cd bin/ls
% vi ls.c     # 进行更改
% git diff    # 查看更改
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
% # 这些更改看起来不错，进行提交...
% git commit ls.c
```

提交会让你进入编辑器描述你所做的更改。完成后，你就有了一个 **本地** 分支。按照手册中的说明，像往常一样构建并安装它。Git 与其他版本控制系统的不同之处在于，你必须显式告诉它哪些文件需要提交。我在提交命令行上选择了文件，但你也可以使用 `git add` 来进行提交，许多更深入的教程会涉及到这一点。

###### 5.2.8.2.2. 更新时

当需要引入新版本时，操作几乎与没有分支时一样。你会像上面那样更新，但在更新之前会有一个额外的命令，在更新后也有一个额外的命令。以下假设你从未修改过树。开始 rebase 操作时，必须确保树是干净的（Git 要求如此）。

```sh
% git checkout main
% git pull --ff-only
% git rebase -i main no-color-ls
```

这会打开一个编辑器，列出所有提交。对于这个示例，不要做任何更改。这通常是在更新基线时所做的（尽管你也可以使用 Git rebase 命令来整理分支中的提交）。

完成上述操作后，你需要将 ls.c 文件中的提交从旧版本的 FreeBSD 移动到新版本中。

有时会出现合并冲突。这是正常的，不用恐慌。你只需像处理任何其他合并冲突一样处理它们。为了简化起见，我将描述一个可能出现的常见问题。完整的处理方式可以在本节末尾找到。

假设包括文件发生了变化，且进行了一个彻底的转换，改用了 terminfo，并且选项名称发生了变化。当你更新时，可能会看到如下内容：

```sh
Auto-merging bin/ls/ls.c
CONFLICT (content): Merge conflict in bin/ls/ls.c
error: could not apply 646e0f9cda11... no color ls
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 646e0f9cda11... no color ls
```

这看起来可能很吓人。如果你打开编辑器，你会看到这是一个典型的三方合并冲突解决方法，你可能已经熟悉了来自其他源代码管理系统的冲突处理（其余的 ls.c 被省略）：

```sh
<<<<<<< HEAD
 #ifdef COLORLS_NEW
 #include <terminfo.h>
 =======
 #undef COLORLS
 #ifdef COLORLS
 #include <termcap.h>
 >>>>>>> 646e0f9cda11... no color ls
....
```

新代码在前，你的代码在后。

正确的修复方法是：在 `#ifdef` 之前添加 `#undef COLORLS_NEW`，然后删除旧的更改：



```sh
#undef COLORLS_NEW
#ifdef COLORLS_NEW
#include <terminfo.h>
....
```

保存文件后，rebase 操作被中断，因此你需要完成它：



```sh
% git add ls.c
% git rebase --continue
....
```

这告诉 Git `ls.c` 已经修复，继续进行 rebase 操作。如果有冲突，你会被带入编辑器来更新提交信息。如果提交信息仍然准确，只需退出编辑器。

如果你在 rebase 时遇到困难，不用恐慌。运行 `git rebase --abort` 会将你带回到干净的状态。重要的是，开始时树必须是未修改的。顺便说一句，前面提到的 `git reflog` 在这里非常有用，因为它会列出所有（中间）提交，你可以查看或检查它们，或进行 cherry-pick。

有关此主题的更多信息，参考 [https://www.freecodecamp.org/news/the-ultimate-guide-to-git-merge-and-git-rebase/](https://www.freecodecamp.org/news/the-ultimate-guide-to-git-merge-and-git-rebase/)，它提供了相当全面的介绍。对于偶尔出现但本指南中未涉及的问题，它是一个很好的资源。

##### 5.2.8.3. 切换到另一个 FreeBSD 分支

如果你希望从 `stable/12` 切换到当前分支。如果你有一个深度克隆，以下命令即可：

```sh
% git checkout main
% # 在此构建并安装...
```

但是，如果你有本地分支，则有一两个注意事项。首先，`rebase` 会重写历史，因此你可能需要采取措施保存它。其次，切换分支通常会导致更多的冲突。如果我们假设上述示例相对于 `stable/12`，然后切换到 `main`，我建议执行以下操作：

```sh
% git checkout no-color-ls
% git checkout -b no-color-ls-stable-12   # 为该分支创建另一个名称
% git rebase -i stable/12 no-color-ls --onto main
```

上述操作会执行以下步骤：首先检出 `no-color-ls` 分支。然后创建一个新名称（`no-color-ls-stable-12`），以防你需要返回它。接着，你将进行 `rebase`，并将其应用到 `main` 分支上。这会查找所有当前 `no-color-ls` 分支中的提交（直到与 `stable/12` 分支合并的地方），然后将它们重新应用到 `main` 分支上，从而在 `main` 分支上创建一个新的 `no-color-ls` 分支（这就是为什么我让你创建一个占位名称）。

### 5.3. MFC（从当前合并）流程

#### 5.3.1. 概述

MFC 工作流可以总结为 `git cherry-pick -x` 加 `git commit --amend` 来调整提交信息。对于多个提交，使用 `git rebase -i` 将它们压缩到一起并编辑提交信息。

#### 5.3.2. 单个提交 MFC

```sh
% git checkout stable/X
% git cherry-pick -x $HASH --edit
```

对于 MFC 提交，例如供应商导入，你需要为 cherry-pick 操作指定一个父提交。通常情况下，这将是你从中进行 cherry-pick 的分支的“第一个父”：

```sh
% git checkout stable/X
% git cherry-pick -x $HASH -m 1 --edit
```

如果出现问题，你需要通过 `git cherry-pick --abort` 中止 cherry-pick 操作，或者修复问题后执行 `git cherry-pick --continue`。

完成 cherry-pick 后，使用 `git push` 推送。如果由于提交竞争失败而导致错误，使用 `git pull --rebase` 并重新尝试推送。

#### 5.3.3. MFC 到 RELENG 分支

向需要批准的分支执行 MFC 时需要更加小心。无论是典型的合并操作还是特殊的直接提交，过程相同：

- 首先将合并或直接提交到适当的 `stable/X` 分支，然后再合并到 `releng/X.Y` 分支。
- 使用 `stable/X` 分支中的哈希值执行 MFC 到 `releng/X.Y` 分支。
- 在提交信息中保留两个 "cherry picked from" 行。
- 在编辑器中确保添加 `Approved by:` 行。

```sh
% git checkout releng/13.0
% git cherry-pick -x $HASH --edit
```

如果忘记添加 `Approved by:` 行，可以通过 `git commit --amend` 编辑提交信息，然后再推送更改。

#### 5.3.4. 多个提交 MFC

```sh
% git checkout -b tmp-branch stable/X
% for h in $HASH_LIST; do git cherry-pick -x $h; done
% git rebase -i stable/X
# 将每个提交（除了第一个）标记为 'squash'
# 如果需要，更新提交信息以反映所有提交的元素。
# 确保保留 "cherry picked from" 行。
% git push freebsd HEAD:stable/X
```

如果由于提交竞争失败而导致推送失败，执行 rebase 并重新尝试：

```sh
% git checkout stable/X
% git pull
% git checkout tmp-branch
% git rebase stable/X
% git push freebsd HEAD:stable/X
```

MFC 完成后，你可以删除临时分支：

```sh
% git checkout stable/X
% git branch -d tmp-branch
```

#### 5.3.5. MFC 供应商导入

供应商导入是唯一会在 `main` 分支中创建合并提交的操作。将合并提交 cherry-pick 到 `stable/XX` 时会遇到额外的困难，因为合并提交有两个父提交。通常，你会希望选择第一个父提交的差异，因为它是与 `main` 的差异（尽管也可能有一些例外）。

```sh
% git cherry-pick -x -m 1 $HASH
```

通常这就是你需要的命令。这将告诉 cherry-pick 应用正确的差异。

有一些情况（希望是少数）可能是 `main` 分支被转换脚本反向合并了。如果是这种情况（虽然我们还没有遇到过这种情况），你应该将上述命令中的 `-m 1` 改为 `-m 2`，以获取正确的父提交。操作如下：

```sh
% git cherry-pick --abort
% git cherry-pick -x -m 2 $HASH
```

`--abort` 会清理第一次失败的尝试。

#### 5.3.6. 重新执行 MFC

如果执行了 MFC 操作并且结果非常糟糕，你希望重新开始，最简单的方式是使用 `git reset --hard`，如下所示：

```sh
% git reset --hard freebsd/stable/12
```

不过，如果你希望保留某些提交，而舍弃其他提交，使用 `git rebase -i` 更为合适。

#### 5.3.7. MFC 时的注意事项

在将源代码提交到 stable 和 releng 分支时，我们有以下目标：

- 清晰地区分直接提交与从其他分支合并的提交。
- 避免在 stable 和 releng 分支中引入已知的破坏性更改。
- 允许开发者确定哪些更改已经从一个分支合并到另一个分支，哪些没有。

在 Subversion 中，我们使用以下做法来实现这些目标：

- 使用 `MFC` 和 `MFS` 标签来标记从其他分支合并的提交。
- 在合并更改时，将修复提交压缩到主要提交中。
- 记录合并信息，以便 `svn mergeinfo --show-revs` 可以工作。

在 Git 中，我们需要采用不同的策略来实现相同的目标。本文件旨在定义在使用 Git 合并源代码提交时的最佳实践，以实现这些目标。一般来说，我们更倾向于使用 Git 的原生支持来实现这些目标，而不是强制执行基于 Subversion 模型的做法。

一项通用说明：由于 Git 与 Subversion 存在技术差异，我们将不会在 stable 或 releng 分支中使用 Git“合并提交”（通过 `git merge` 创建的提交）。相反，当本文件提到“合并提交”时，它指的是最初在 `main` 分支上创建的提交，并通过某种方式使用 `git cherry-pick` 被复制或“落地”到 stable 分支，或者从 stable 分支复制到 releng 分支的提交。

#### 5.3.8. 查找合适的哈希以进行 MFC

Git 提供了内建支持，使用 `git cherry` 和 `git log --cherry` 命令。这些命令比较提交的原始差异（但不比较其他元数据，如日志消息），以确定两个提交是否相同。当每个提交从 `main` 分支单独合并到稳定分支时，这些命令非常有效，但如果从 `main` 分支合并多个提交并将它们合并为一个提交到稳定分支时，这种方法就会失效。为了绕过这些困难，项目广泛使用 `git cherry-pick -x` 保留所有行，并且正在开发自动化工具以利用这一点。

#### 5.3.9. 提交消息标准

##### 5.3.9.1. 标记 MFC

项目采用了以下标记 MFC 的做法：

- 使用 `git cherry-pick` 的 `-x` 标志。这会在提交消息中添加一行，包含合并时原始提交的哈希。由于这是 Git 直接添加的，提交者在合并时不需要手动编辑提交日志。

当合并多个提交时，保留所有的 `cherry picked from` 行。

##### 5.3.9.2. 是否修剪元数据？

在 Subversion（甚至 CVS）中，并未清晰记录如何为 MFC 提交格式化日志消息中的元数据。应该保留原始提交的元数据不变，还是应该修改以反映 MFC 提交本身的信息？

历史做法有所不同，尽管某些差异与字段有关。例如，涉及 PR 的 MFC 通常会在 MFC 中包含 PR 字段，以便将 MFC 提交包含在 bug 跟踪器的审核历史中。其他字段则不太明确。例如，Phabricator 显示的是最后一个提交与评审标签相关的 diff，因此如果在 MFC 中包含 Phabricator URL，会替换主提交为已落地的提交。审查人列表也不明确。如果审查人批准了对 `main` 的更改，这是否意味着他们也批准了 MFC 提交？如果是完全相同的代码或者仅做了微小修改，那么答案可能是肯定的。但对于更复杂的修改来说，这显然不成立。即使代码完全相同，若提交没有冲突但引入了 ABI 更改，审查人可能会因为 ABI 改变而批准 `main` 提交，但可能不会批准相同的提交。对此，必须根据具体情况做出判断，直到能够就清晰的指南达成一致。

对于由 re@ 管理的 MFC，增加了新的元数据字段，如 `Approved by` 标签，用于批准的提交。这些新元数据必须通过 `git commit --amend` 或类似命令添加，在原始提交经过审核和批准后添加。我们也可能希望将一些元数据字段保留在 MFC 提交中，比如 Phabricator URL，以便将来由 re@ 使用。

保留现有元数据提供了一种非常简单的工作流。开发者使用 `git cherry-pick -x`，无需编辑日志消息。

如果我们选择调整 MFC 中的元数据，开发者将需要显式地通过使用 `git cherry-pick --edit` 或 `git commit --amend` 来编辑日志消息。然而，与 SVN 相比，至少现有的提交消息可以预先填充，并且可以在不重新输入整个提交消息的情况下添加或删除元数据字段。

总的来说，开发者可能需要为非平凡的 MFC 提交整理其提交消息。

### 5.4. 使用 Git 进行供应商导入

本节详细介绍了使用 Git 进行供应商导入的过程。

#### 5.4.1. 分支命名约定

所有供应商分支和标签都以 `vendor/` 开头。这些分支和标签默认是可见的。

>**注意**
>
> 本章遵循的约定是，`freebsd` 为官方 FreeBSD Git 仓库的源名称。如果你使用不同的约定，请在以下示例中将 `freebsd` 替换为你使用的名称。

我们将以更新 NetBSD 的 mtree 为例，说明其在我们树中的供应商分支。该供应商分支为 `vendor/NetBSD/mtree`。

#### 5.4.2. 更新旧的供应商导入

供应商树通常只包含适合 FreeBSD 的第三方软件子集。这些树通常比 FreeBSD 树要小得多。因此，Git 工作树非常小且快速，且是推荐使用的方法。确保所选目录（例如 `../mtree`）尚不存在。

```sh
% git worktree add ../mtree vendor/NetBSD/mtree
```

#### 5.4.3. 更新供应商分支中的源代码

准备一个完整、干净的供应商源代码树。导入所有内容，但只合并所需的部分。

以下示例假设 NetBSD 源代码已从其 GitHub 镜像克隆到 `~/git/NetBSD`。请注意，“上游”可能已经添加或删除了文件，因此我们要确保删除操作也被传播。`[net/rsync](https://cgit.freebsd.org/ports/tree/net/rsync/)` 通常已安装，因此我将使用它。

```sh
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

至关重要的是验证你正在导入的源代码来自可信的来源。许多开源项目使用加密签名来签署代码更改、Git 标签和/或源代码 tarball。始终验证这些签名，并使用诸如 Jail、chroot 等隔离机制，结合专用的、非特权用户账户，该账户与你日常使用的账户不同（有关详细信息，请参阅下文的"更新 FreeBSD 源树"部分），直到你确信所导入的源代码是安全的。跟踪上游开发并偶尔检查上游代码更改，有助于提高代码质量并使所有参与者受益。导入到供应商区域之前，检查 `git diff` 结果也是一个好主意。

始终运行 `git diff` 和 `git status` 命令，并仔细检查结果。如有疑问，使用 `git annotate` 查看供应商分支或上游 Git 仓库，了解是谁做了更改以及为什么。

上面的示例中我们使用了 `-m` 来说明，但你应该在编辑器中撰写适当的消息（使用提交消息模板）。

同样，重要的是使用 `git tag -a` 创建一个注解标签，否则推送将被拒绝。只有注解标签才能被推送。注解标签使你有机会输入提交消息，输入你导入的版本以及该版本中的任何重要新特性或修复。

#### 5.4.4. 更新 FreeBSD 副本

此时，你可以将导入的供应商代码推送到我们的仓库中的 `vendor` 分支。

```sh
% git push --follow-tags freebsd vendor/NetBSD/mtree
```

`--follow-tags` 告诉 `git push` 也推送与本地提交版本相关联的标签。

#### 5.4.5. 更新 FreeBSD 源树

现在你需要更新 FreeBSD 中的 mtree。源代码位于 `contrib/mtree` 目录下，因为它是上游软件。

有时，我们可能需要对贡献的代码进行更改，以更好地满足 FreeBSD 的需求。尽可能地，请尝试将本地更改贡献回上游项目，这有助于它们更好地支持 FreeBSD，也节省了将来导入更新时解决冲突的时间。

```sh
% cd ../src
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree
```

这将生成一个子树合并提交，将 `contrib/mtree` 与本地的 `vendor/NetBSD/mtree` 分支合并。检查合并结果的 diff 和上游分支的内容。如果合并减少了本地更改，仅保留了空行或缩进更改等琐碎差异，请尝试修改本地更改以减少与上游的差异，或尝试将剩余的更改贡献回上游项目。如果有冲突，你需要在提交之前解决它们。在合并提交消息中包含有关合并更改的详细信息。

一些开源软件包含 `configure` 脚本，用于生成定义如何构建代码的文件；通常，这些生成的文件（如 `config.h`）应在导入过程中更新。在执行此操作时，始终记住这些脚本是在当前用户的凭据下运行的可执行代码。此过程应始终在隔离环境中运行，理想情况下是在没有网络访问权限的 Jail 内，并使用非特权账户；或者，至少使用与你日常使用的账户不同的专用账户，这样可以最大限度地减少遇到可能导致数据丢失或更严重的恶意代码的风险。使用隔离的 Jail 还可以防止 `configure` 脚本检测到本地安装的软件包，从而避免出现意外结果。

在测试你的更改时，首先在 chroot 或 Jail 环境中运行，甚至可以先在虚拟机中测试，尤其是对于内核或库的修改。这种方法有助于防止与工作环境发生不良交互。对于许多基本系统组件都使用的库修改，这尤其有益。

#### 5.4.6. 将你的更改与最新的 FreeBSD 源树重新基准化

由于当前的政策建议避免使用合并，如果在你有机会推送之前，FreeBSD 的 `main` 已经向前移动，那么你将不得不重新执行合并。

常规的 `git rebase` 或 `git pull --rebase` 无法像合并提交那样重新基准化，因此你需要重新创建该提交。

以下步骤应帮助你轻松地重新创建合并提交，就好像 `git rebase --merge-commits` 正常工作一样：

- 进入仓库的顶层目录
- 创建一个包含已合并树内容的旁支 `XXX`
- 更新旁支 `XXX`，使其与 FreeBSD 的 `main` 分支合并并保持最新。
  - 在最坏的情况下，你仍然需要解决合并冲突（如果有的话），但这种情况应该非常罕见。
  - 解决冲突，并在需要时将多个提交压缩为 1 个提交（如果没有冲突，则不需要压缩）
- 切换到 `main` 分支
- 创建一个分支 `YYY`（如果出现问题，可以更容易地撤销）
- 重新进行子树合并
- 在子树合并中解决冲突之前，切换到 `XXX` 分支，将其内容覆盖到当前的 `main` 分支上。
  - 末尾的 `.` 是很重要的，且要确保处于仓库的顶层目录。
  - 而不是切换到 `XXX` 分支，它会将 `XXX` 的内容覆盖到仓库顶部。
- 使用之前的提交消息提交结果（假设 `XXX` 分支只有一次合并提交）。
- 确保这两个分支的内容是相同的。
- 根据需要进行任何审查，包括让其他人检查一下，如果你认为这是必要的。
- 推送提交，如果再次遇到推送竞争失败，只需重新执行这些步骤（参见下文的操作步骤）。
- 提交到上游之后，删除这些分支，它们只是临时的。

以下是使用 `mtree` 示例时，执行上述操作的命令（`#` 后面的部分是注释，用于帮助将命令与上面的描述对应起来）：

```sh
% cd ../src			# 进入仓库顶层目录
% git checkout -b XXX		# 为合并创建一个新的临时 XXX 分支
% git fetch freebsd		# 获取来自上游的更改
% git merge freebsd/main	# 合并更改并解决冲突
% git checkout -b YYY freebsd/main # 创建一个新的临时 YYY 分支以重新操作
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree # 重新进行子树合并
% git checkout XXX .		# XXX 分支包含了解决冲突的内容
% git commit -c XXX~1		# -c 使用重新基准化之前的提交消息
% git diff XXX YYY		# 应该为空
% git show YYY			# 应该只包含你想要的更改，并且是来自供应商分支的合并提交
```

注意：如果提交出现问题，你可以通过重新执行创建分支的命令，并使用 `-B` 参数来重置 `YYY` 分支，以便从头开始：

```sh
% git checkout -B YYY freebsd/main # 如果重新开始更容易，可以创建一个新的临时 YYY 分支
```

#### 5.4.7. 推送更改

当你认为更改已经准备好时，可以将其推送到 GitHub 或 GitLab 上的 fork 以供他人审查。Git 的一个优点是它允许你发布草稿供其他人查看。虽然 Phabricator 很适合内容审查，但发布更新后的供应商分支和合并提交使得其他人可以查看详细信息，因为这些更改最终将出现在仓库中。

经过审查后，当你确信这是一个好的更改时，你可以将其推送到 FreeBSD 仓库：

```sh
% git push freebsd YYY:main	# 将提交推送到上游的 'main' 分支
% git branch -D XXX		# 删除临时的 XXX 分支
% git branch -D YYY		# 删除临时的 YYY 分支
```

注意：我使用 `XXX` 和 `YYY` 是为了使它们显得明显，它们是糟糕的名称，应该只在你的机器上使用。如果你将这样的名称用于其他工作，那么你需要选择不同的名称，否则可能会丢失其他工作。这些名称并没有什么神奇之处。上游不会允许你推送它们，但无论如何，请注意上述命令。某些命令的语法与常规用法略有不同，而这种不同的行为对这个食谱的成功至关重要。

#### 5.4.8. 如果需要重做操作

如果你尝试在上一节中进行推送并且失败，那么你应执行以下操作来“重做”事情。这个序列会保持提交消息始终为 `XXX~1`，以便简化提交过程。

```sh
% git checkout -B XXX YYY	# 重新创建临时分支 XXX 并切换到它
% git merge freebsd/main	# 合并更改并解决冲突
% git checkout -B YYY freebsd/main # 重新创建新的临时分支 YYY 以重新操作
% git subtree merge -P contrib/mtree vendor/NetBSD/mtree # 重新进行子树合并
% git checkout XXX .		# XXX 分支包含了解决冲突的内容
% git commit -c XXX~1		# -c 重新使用重新基准化之前的提交消息
```

然后，像之前一样检查并推送。

### 5.5. 创建一个新的供应商分支

有多种方式可以创建一个新的供应商分支。推荐的方式是创建一个新的仓库，然后将其与 FreeBSD 合并。如果要将 `glorbnitz` 导入到 FreeBSD 树中，版本为 3.1415。为了简单起见，我们不对该版本进行修剪。它是一个简单的用户命令，可以将 `nitz` 设备置于不同的魔法 `glorb` 状态，且其足够小，修剪不会节省太多空间。

#### 5.5.1. 创建仓库

```sh
% cd /some/where
% mkdir glorbnitz
% cd glorbnitz
% git init
% git checkout -b vendor/glorbnitz
```

此时，你已创建了一个新的仓库，所有新的提交都将在 `vendor/glorbnitz` 分支上进行。

如果你更熟悉，可以直接在 FreeBSD 克隆中执行此操作，使用 `git checkout --orphan vendor/glorbnitz`。

#### 5.5.2. 将源代码复制进去

由于这是一个新的导入，你可以直接使用 `cp`、`tar` 或甚至 `rsync` 将源代码复制进去，如上所示。假设没有点文件，我们将添加所有内容。

```sh
% cp -r ~/glorbnitz/* .
% git add *
```

此时，你应该有一个干净的 `glorbnitz` 复制，准备提交。

```sh
% git commit -m "Import GlorbNitz frobnosticator revision 3.1415"
```

如上所示，我使用了 `-m` 选项以简化操作，但实际上你应该创建一个解释什么是 Glorb，以及为什么要使用 Nitz 来获得它的提交消息。并非每个人都会知道，因此在实际提交时，你应该遵循 [提交日志消息](https://docs.freebsd.org/en/articles/committers-guide/#commit-log-message) 部分，而不是模仿这里使用的简短风格。

#### 5.5.3. 现在将其导入到我们的仓库

现在，你需要将分支导入到我们的仓库。

```sh
% cd /path/to/freebsd/repo/src
% git remote add glorbnitz /some/where/glorbnitz
% git fetch glorbnitz vendor/glorbnitz
```

请注意，`vendor/glorbnitz` 分支已添加到仓库中。此时，**/some/where/glorbnitz** 可以删除，如果你愿意的话。它只是为了达到某个目的而存在。

#### 5.5.4. 打标签并推送

从这里开始的步骤与更新供应商分支时类似，只是没有更新供应商分支的步骤。

```sh
% git worktree add ../glorbnitz vendor/glorbnitz
% cd ../glorbnitz
% git tag --annotate vendor/glorbnitz/3.1415
# 确保提交正确，可以使用 "git show" 查看
% git push --follow-tags freebsd vendor/glorbnitz
```

“正确”意味着：

1. 所有正确的文件都存在
2. 没有错误的文件存在
3. 供应商分支指向一个合理的地方
4. 标签看起来很好，且是注释的
5. 标签的提交消息简要概述自上次标签以来的新变化

#### 5.5.5. 最终将其合并到主树中

```
% cd ../src
% git subtree add -P contrib/glorbnitz vendor/glorbnitz
# 确保提交正常，可以使用 "git show"
% git commit --amend   # 最后的提交消息检查
% git push freebsd
```

这里“正常”意味着：

1. 所有正确的文件，并且没有错误的文件，已合并到 `contrib/glorbnitz` 中。
2. 树中没有其他更改。
3. 提交消息看起来[正常](https://docs.freebsd.org/en/articles/committers-guide/#commit-log-message)。应该包含自上次向 FreeBSD `main` 分支合并以来的更改总结以及任何警告。
4. 如果有任何重要的用户可见更改、升级注意事项等，应更新 `RELNOTES` 和 `UPDATING` 文件。

>**注意**
>
>这还没有将 `glorbnitz` 连接到构建中。如何做到这一点取决于被导入的软件，超出了本教程的范围。

##### 5.5.5.1. 保持更新

随着时间的推移，现在是更新树以包含最新的上游更改的时刻。切换到 `main` 分支时，确保没有差异。在执行以下操作之前，最好先将更改提交到一个分支（或使用 `git stash`）。

如果你习惯使用 `git pull`，我们强烈建议使用 `--ff-only` 选项，并将其设置为默认选项。或者，如果你在 `main` 分支上有暂存的更改，`git pull --rebase` 会更有用。

```sh
% git config --global pull.ff only
```

如果你只希望此设置应用于当前仓库，可能需要省略 `--global`。

```sh
% cd freebsd-src
% git checkout main
% git pull (--ff-only|--rebase)
```

有一个常见的陷阱，`git pull` 组合命令会尝试执行合并，这有时会创建一个以前不存在的合并提交。这个过程可能更难恢复。

我们也推荐使用更长的命令形式。

```sh
% cd freebsd-src
% git checkout main
% git fetch freebsd
% git merge --ff-only freebsd/main
```

这些命令会将你的树重置为 `main` 分支，然后从你最初拉取的源更新它。在执行此操作之前，切换到 `main` 非常重要，这样它才能向前推进。现在，是时候将更改推进：

```sh
% git rebase -i main working
```

这将弹出一个交互式屏幕，你可以在其中更改默认设置。现在，直接退出编辑器即可。一切应该顺利应用。如果没有，你需要解决冲突。[这篇 GitHub 文档](https://docs.github.com/en/free-pro-team@latest/github/using-git/resolving-merge-conflicts-after-a-git-rebase)可以帮助你解决这个问题。

##### 5.5.5.2. 将更改推送到上游

首先，确保推送 URL 已正确配置为上游仓库。

```sh
% git remote set-url --push freebsd ssh://git@gitrepo.freebsd.org/src.git
```

然后，验证你的用户名和电子邮件是否配置正确。我们要求它们与 FreeBSD 集群中的 `passwd` 条目完全匹配。

在 `freefall.freebsd.org` 上使用

```sh
freefall% gen-gitconfig.sh
```

可以获得你可以直接使用的配方，前提是 **/usr/local/bin** 在 PATH 中。

下面的命令会将 `working` 分支合并到上游的 `main` 分支。重要的是在执行此操作之前，你要确保你的更改完全符合 FreeBSD 源仓库中的要求。这种语法会将 `working` 分支推送到 `main`，将 `main` 分支推进。只有当这导致 `main` 的线性更改时（例如，没有合并）时，你才可以执行此操作。

```sh
% git push freebsd working:main
```

如果由于提交竞争导致你的推送被拒绝，请在重新尝试之前重新基准化你的分支：

```sh
% git checkout working
% git fetch freebsd
% git rebase freebsd/main
% git push freebsd working:main
```

##### 5.5.5.3. 将更改推送到上游（替代方法）

有些人发现，在推送到远程仓库之前，将更改合并到本地 `main` 分支上会更容易。此外，`git arc stage` 会将更改从分支移到本地 `main`，当你只需要做分支的子集时。操作与之前部分类似：

```sh
% git checkout main
% git merge --ff-only `working`
% git push freebsd
```

如果你输了比赛，那么再试一次，尝试以下操作：

```sh
% git pull --rebase
% git push freebsd
```

这些命令会获取最新的 `freebsd/main`，然后将本地的 `main` 更改基准化到其上，这是当你丢失提交竞争时所需要的。注意：使用此技巧无法合并供应商分支的提交。

##### 5.5.5.4. 查找 Subversion 修订版

你需要确保已获取备注（有关详细信息，请参阅 [日常使用](https://docs.freebsd.org/en/articles/committers-guide/#git-mini-daily-use)）。获取这些备注后，它们将在 `git log` 命令中显示，如下所示：

```sh
% git log
```

如果你有特定版本，可以使用以下构造命令：

```sh
% git log --grep revision=XXXX
```

这样可以找到特定的修订版。`commit` 后的十六进制数字是你可以用来引用此提交的哈希值。

### 5.6. Git 常见问题

本节提供了许多可能会经常出现的问题的针对性答案，适用于用户和开发人员。

>**注意**
>
> 我们使用 FreeBSD 仓库的常规约定，源为 `freebsd`，而不是默认的 `origin`，以便让人们将其用于自己的开发，并最小化将更改错误推送到错误仓库的风险。

#### 5.6.1. 用户

##### 5.6.1.1. 如何用一个仓库副本跟踪 `-current` 和 `-stable`？

**Q:** 尽管磁盘空间不是大问题，但使用仓库副本更高效。在 SVN 镜像中，我可以从同一个仓库签出多个树。如何用 Git 来实现？

**A:** 你可以使用 Git 工作树。这里有多种方法可以做到，但最简单的方法是使用克隆来跟踪 `-current`，并使用工作树来跟踪稳定版本。虽然使用“裸仓库”作为一种应对方法已被提出，但它更为复杂，且不在此文档中介绍。

首先，你需要克隆 FreeBSD 仓库，以下示例中将其克隆到 `freebsd-current` 以减少混淆。`$URL` 是适合你的镜像源：

```sh
% git clone -o freebsd --config remote.freebsd.fetch='+refs/notes/*:refs/notes/*' $URL freebsd-current
```

克隆完成后，你可以简单地从中创建一个工作树：

```sh
% cd freebsd-current
% git worktree add ../freebsd-stable-12 stable/12
```

这将把 `stable/12` 签出到名为 `freebsd-stable-12` 的目录，它与 `freebsd-current` 目录平行。创建后，更新工作树的方式与预期类似：

```sh
% cd freebsd-current
% git checkout main
% git pull --ff-only
# 从上游的更改现在已本地化，并且当前树已更新
% cd ../freebsd-stable-12
% git merge --ff-only freebsd/stable/12
# 现在你的 stable/12 也已更新
```

我建议使用 `--ff-only`，因为它更安全，可以避免意外进入“合并噩梦”，其中树中会有额外的更改，从而迫使你进行复杂的合并，而不是简单的合并。

这里有一篇 [详细的写作](https://adventurist.me/posts/00296)，可以更深入地探讨此话题。

#### 5.6.2. 开发人员

##### 5.6.2.1. 哎呀！我不小心提交到了 `main` 分支，而不是另一个分支

**Q:** 有时，我会犯错，不小心提交到了 `main` 分支。我该怎么办？

**A:** 首先，不要惊慌。

其次，不要推送。实际上，如果你没有推送，几乎所有问题都可以解决。本节的所有答案假设没有进行推送。

以下解答假设你已经提交到了 `main` 分支，并且想创建一个名为 `issue` 的分支：

```sh
% git branch issue                # 创建 'issue' 分支
% git reset --hard freebsd/main   # 将 'main' 重置回官方的提交
% git checkout issue              # 回到原先的地方
```

##### 5.6.2.2. 哎呀！我不小心提交到了错误的分支

**Q:** 我正在 `wilma` 分支上开发一个功能，但不小心在 `wilma` 中提交了与 `fred` 分支相关的更改。我该怎么办？

**A:** 这个解答与之前类似，不过是使用了 `cherry-pick`。假设 `wilma` 分支上只有一次提交，但这种方法也可以推广到更复杂的情况。假设这是 `wilma` 上的最后一次提交（因此使用 `wilma` 在 `git cherry-pick` 命令中），但这也可以推广。

```sh
# 我们在 wilma 分支上
% git checkout fred		# 切换到 fred 分支
% git cherry-pick wilma		# 复制误提交的提交
% git checkout wilma		# 回到 wilma 分支
% git reset --hard HEAD^	# 将 wilma 回退到上一个提交
```

Git 专家通常会先将 `wilma` 分支回退 1 次提交，然后切换到 `fred`，然后使用 `git reflog` 查看被删除的提交是什么，再将其 `cherry-pick` 到 `fred` 分支。

**Q:** 如果我想将一些更改提交到 `main`，但出于某些原因保留其余的更改在 `wilma` 中，怎么办？

**A:** 上面相同的技巧也适用于你想将当前分支的一部分更改提交到 `main`，而保留分支其余部分（例如发现了一个不相关的拼写错误或修复了一个偶然的 bug）。你可以将这些更改 `cherry-pick` 到 `main`，然后推送到上游仓库。完成后，清理工作也非常简单：只需要使用 `git rebase -i`。Git 会注意到你已经这样做了，并自动跳过相同的更改（即使你更改了提交消息或稍微调整了提交）。你无需返回 `wilma` 分支来调整它：只需执行 rebase 操作即可！

**Q:** 我想将 `wilma` 分支的一些更改拆分到 `fred` 分支中，怎么做？

**A:** 更一般的答案与之前相同。你需要先检出或创建 `fred` 分支，逐一 `cherry-pick` 需要的更改，然后对 `wilma` 分支进行 rebase，移除你已 `cherry-pick` 到 `fred` 分支的更改。使用 `git rebase -i main wilma` 会打开编辑器，你可以删除与 `fred` 分支中已复制提交对应的 `pick` 行。如果一切顺利，且没有冲突，那么操作完成。如果遇到冲突，你需要在过程中解决它们。

另一种做法是先检出 `wilma`，然后创建 `fred` 分支，使其指向树中的相同位置。然后，你可以对这两个分支执行 `git rebase -i`，在编辑器中保留要放入 `fred` 或 `wilma` 分支的更改 `pick` 行，删除其他行。一些人会在开始拆分之前创建一个名为 `pre-split` 的标签/分支，以防拆分过程中出现问题。万一需要撤销拆分操作，可以执行以下步骤：

```sh
% git checkout pre-split	# 回到起点
% git branch -D fred		   # 删除 fred 分支
% git checkout -B wilma		# 重置 wilma 分支
% git branch -d pre-split	# 假装什么也没发生
```

最后一步是可选的。如果你打算重新尝试拆分，你可以省略此步骤。

**Q:** 但我在阅读时没有看到你的建议，没创建分支，现在 `fred` 和 `wilma` 已经完全搞乱了。我该如何找回 `wilma` 在开始之前的状态？我不知道我做了多少次更改。

**A:** 一切尚未丢失。只要不久之前做的更改太多（几百次提交的话），你依然可以找回之前的状态。

假设我创建了一个 `wilma` 分支并提交了一些更改，然后决定将其拆分成 `fred` 和 `wilma`。如果在拆分过程中没有出现问题，一切应该顺利。但假设有些不正常的事情发生了，你可以使用 `git reflog` 来查看做过的更改：

```sh
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

在这里，我们可以看到做过的更改。你可以通过这些信息找出哪里出错了。我想指出几点：首先，`HEAD@{X}` 是一个“提交点”，因此可以将它作为命令的参数。虽然如果该命令对仓库做了提交操作，X 的数字会发生变化。你也可以使用哈希值（第一列）。

接下来，“Encourage contributions” 是我在决定拆分之前提交到 `wilma` 的最后一次提交。你也可以看到这个哈希值在我创建 `fred` 分支时也出现了。我从 `fred` 开始执行 rebase 操作，可以看到“开始”步骤、每个步骤以及完成步骤。虽然我们在这里不需要这个信息，但它可以帮助你清楚了解发生了什么。幸运的是，修复这个问题的方法是按照之前解答中的步骤执行，但使用哈希值 `869cbd3` 代替 `pre-split`。虽然这种方法看起来有点冗长，但由于你是一次处理一件事，记住它很容易。你还可以将操作堆叠在一起：

```sh
% git checkout -B wilma 869cbd3
% git branch -D fred
```

这样，你就准备好重新尝试了。使用 `checkout -B` 并指定哈希值结合了检出和为其创建分支的操作。使用 `-B` 而非 `-b` 会强制移动已经存在的分支。这两种方式都可以使用，这正是 Git 强大（也有时让人头疼）的原因之一。我更倾向于使用 `git checkout -B xxxx hash` 来避免先检出哈希值后再创建/移动分支，因为那样会出现令人不安的“分离头”状态警告：

```sh
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

这将产生相同的效果，但我需要阅读更多内容，而“分离头”状态不是我喜欢思考的图像。

##### 5.6.2.3. 哎呀！我执行了 `git pull` 却创建了一个合并提交，该怎么办？

**问：** 我习惯性地对自己的开发树执行了 `git pull`，结果在 `main` 上创建了一个合并提交。我该如何恢复？

**答：** 当你在检出开发分支时调用 pull，就会出现这种情况。

许多开发者使用 `git pull --rebase` 来避免这种情况。

在 pull 之后，你将检出新的合并提交。Git 支持 `HEAD^#` 语法来查看合并提交的父提交：

```sh
git log --oneline HEAD^1   # 查看第一个父提交的提交
git log --oneline HEAD^2   # 查看第二个父提交的提交
```

从这些日志中，你可以轻松识别哪个提交是你的开发工作。然后，只需将分支重置为相应的 `HEAD^#`：

```sh
git reset --hard HEAD^1
```

此外，在此阶段执行 `git pull --rebase` 会将你的更改 rebase 到最新的 `freebsd/main`。

**问：** 但我还需要修复 `main` 分支。我该怎么做？

**答：** Git 在 `freebsd/` 命名空间中跟踪远程仓库分支。要修复 `main` 分支，只需让它指向远程的 `main`：

```sh
git branch -f main freebsd/main
```

Git 中的分支并没有什么神奇之处：它们只是图上的标签，通过提交自动向前移动。因此上述操作可行，因为你只是移动了一个标签。分支没有需要保留的元数据。

##### 5.6.2.4. 混合和匹配分支

**问：** 我有两个分支 `worker` 和 `async`，我想把它们合并成一个名为 `feature` 的分支，同时保留两个分支中的提交记录，应该怎么做？

**答：** 这可以通过 `cherry pick` 来实现。

```sh
% git checkout worker
% git checkout -b feature	# 创建一个新的分支
% git cherry-pick main..async	# 将变更引入
```

现在你就有了一个新的分支 `feature`。这个分支结合了两个分支的提交。你可以通过 `git rebase` 进一步修改它。

**问：** 我有一个名为 `driver` 的分支，我想将其拆分为 `kernel` 和 `userland`，这样我就可以分别独立演进它们，并在每个分支准备好时提交它们。

**答：** 这需要一些准备工作，但 `git rebase` 可以完成大部分工作。

```sh
% git checkout driver		# 检出 driver 分支
% git checkout -b kernel	# 创建 kernel 分支
% git checkout -b userland	# 创建 userland 分支
```

现在你有了两个相同的分支。接下来是将提交拆分到这两个分支中的过程。我们假设 `driver` 中的所有提交都进入 `kernel` 或 `userland`，而不是两个分支。

```sh
% git rebase -i main kernel
```

在编辑器中，保留你想要的提交（通过 `p` 或 `pick` 行），删除不需要的提交（这看起来很吓人，但如果出错了，你可以通过 `driver` 分支重新开始，因为你还没有修改它）。

```sh
% git rebase -i main userland
```

对 `userland` 分支做同样的事。

**问：** 哇！我按照上述操作了，但忘记了 `kernel` 分支的一个提交。如何恢复这个提交？

**答：** 你可以通过 `driver` 分支找到缺失提交的哈希，并将其 `cherry-pick` 到 `kernel` 分支。

```sh
% git checkout kernel
% git log driver
% git cherry-pick $HASH
```

**问：** 好的。我的情况和上面一样，但我的提交都混在一起了。我需要将一个提交的部分内容放到一个分支，剩余部分放到另一个分支。事实上，我有好几个这样的提交。你提到的选择提交的 `rebase` 方法看起来有点棘手。

**答：** 在这种情况下，你最好先整理原始分支，将提交拆分开来，然后使用上述方法将分支拆分。

假设现在只有一个干净的提交。你可以使用 `git rebase` 中的 `edit` 选项，或者直接在当前提交上使用该操作。无论哪种方式，步骤都一样。首先，我们需要备份一个提交，并将更改留在工作区：

```sh
% git reset HEAD^
```

注意：此时不要加上 `--hard`，因为那样会将工作区的更改删除。

现在，如果你幸运的话，需要拆分的更改完全是基于文件的。在这种情况下，你可以像平常一样使用 `git add` 来将每组文件提交一次。注意：当你执行重置操作时，提交消息会丢失，所以如果需要保存消息，应该先保存一份副本（不过，你可以通过 `git log $HASH` 恢复提交消息）。

如果不幸运，可能需要将文件拆分开来。这时你可以使用以下工具逐个处理文件：

```sh
git add -i foo/bar.c
```

这将逐步显示差异，并提示你是否包含或排除每个变更块。完成后，使用 `git commit` 提交剩余的更改。如果有多个文件需要处理，你可以多次运行这个命令，通常我发现一次处理一个文件并使用 `git rebase -i` 将相关提交合并会更简单些。

##### 5.6.2.5. 加入 FreeBSD GitHub 组织

**问：** 我如何加入 FreeBSD GitHub 组织？

**答：** 请参见 [我们的 GitHub Wiki 信息](https://wiki.freebsd.org/GitHub#Joining_the_Organisation) 页面了解详情。简而言之，所有 FreeBSD 提交者都可以加入。非提交者申请加入的，将根据具体情况进行审核。

#### 5.6.3. 克隆和镜像

**问：** 我想镜像整个 Git 仓库，怎么做？

**答：** 如果你只是想镜像仓库，可以使用以下命令：

```sh
% git clone --mirror $URL
```

但如果你希望用它做其他事情，那么你会需要重新克隆仓库。

首先，这会创建一个 "bare 仓库"，即只有仓库数据库，而没有已检出的工作区。这对于镜像非常好，但对于日常工作来说就不太合适了。你可以通过 `git worktree` 来解决这个问题：

```sh
% git clone --mirror https://git.freebsd.org/ports.git ports.git
% cd ports.git
% git worktree add ../ports main
% git worktree add ../quarterly branches/2020Q4
% cd ../ports
```

但是，如果你不打算将镜像用于进一步的本地克隆，那它就不太适合了。

第二个缺点是，Git 通常会重写上游的引用（如分支名、标签等），使得你的本地引用可以独立于上游发展。这意味着，如果你在该仓库中提交更改，除非是私人项目的分支，否则你会丢失更改。

**问：** 那我应该怎么办？

**答：** 你可以将所有上游仓库的引用放入本地仓库的私有命名空间。Git 是通过 'refspec' 来克隆所有内容的，默认的 refspec 是：

```sh
fetch = +refs/heads/*:refs/remotes/freebsd/*
```

这意味着它只会获取分支的引用。

然而，FreeBSD 仓库包含了很多其他内容。你可以为每个引用命名空间添加显式的 refspec，或者选择获取所有内容。要设置你的仓库来执行这一操作：

```sh
git config --add remote.freebsd.fetch '+refs/*:refs/freebsd/*'
```

这会将上游仓库中的所有内容放入本地仓库的 `refs/freebsd/` 命名空间。请注意，这也会获取所有未转换的 vendor 分支，并且它们关联的引用数量非常庞大。

你需要使用完整的名称来引用这些 "refs"，因为它们不在 Git 的常规命名空间中。

```sh
git log refs/freebsd/vendor/zlib/1.2.10
```

上述命令会查看 zlib 的 vendor 分支日志，从 1.2.10 开始。

### 5.7. 与他人合作

在像 FreeBSD 这样的大型项目中，良好的合作能力是软件开发的关键之一。在 FreeBSD 项目的 Git 仓库中，当前不允许用户创建的分支被推送到仓库中。因此，如果你希望与他人共享你的更改，你必须使用其他机制，例如托管的 GitLab 或 GitHub 来共享用户生成的分支。

以下是如何创建一个基于 FreeBSD `main` 分支的用户生成分支，并将其推送到 GitHub 的步骤。

在开始之前，确保你的本地 Git 仓库是最新的，并且设置了正确的远程源 [如上所示](https://docs.freebsd.org/en/articles/committers-guide/#keeping_current)。

```sh
% git remote -v
freebsd https://git.freebsd.org/src.git (fetch)
freebsd ssh://git@gitrepo.freebsd.org/src.git (push)
```

第一步是按照这些 [指南](https://docs.github.com/en/github/getting-started-with-github/fork-a-repo) 在 GitHub 上创建一个 [FreeBSD](https://github.com/freebsd/freebsd-src) 的 fork。这个 fork 的目标应该是你的个人 GitHub 账户（比如我的是 gvnn3）。

然后，在本地系统上添加一个指向你 fork 的远程源：

```sh
% git remote add github git@github.com:gvnn3/freebsd-src.git
% git remote -v
github	git@github.com:gvnn3/freebsd-src.git (fetch)
github	git@github.com:gvnn3/freebsd-src.git (push)
freebsd	https://git.freebsd.org/src.git (fetch)
freebsd	ssh://git@gitrepo.freebsd.org/src.git (push)
```

现在，你可以创建一个分支 [如上所示](https://docs.freebsd.org/en/articles/committers-guide/#keeping_a_local_branch)。

```sh
% git checkout -b gnn-pr2001-fix
```

在你的分支中进行任何修改。完成后，进行构建和测试，当你准备好与他人合作时，就可以将你的更改推送到托管的分支了。在推送之前，你需要设置适当的上游分支，因为第一次尝试推送到 GitHub 时，Git 会提示你：

```sh
% git push github
fatal: The current branch gnn-pr2001-fix has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream github gnn-pr2001-fix
```

按照 Git 的建议设置推送上游分支后，推送就会成功：

```sh
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

之后对同一分支的更改将默认正确推送：

```sh
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

此时，你的工作已推送到 GitHub 上的分支，并且你可以与其他协作者共享该链接。

### 5.8. 合并 GitHub Pull 请求

本节介绍了如何将提交到 FreeBSD GitHub 镜像库的 GitHub Pull 请求合并到 FreeBSD 仓库。虽然这不是提交补丁的官方方式，但有时通过这种方式提交的修复非常有效，直接将它们合并到提交者的树上，再从那里推送到 FreeBSD 的树上是最简单的做法。类似的步骤也可以用于将其他仓库的分支合并进来。在合并他人的 Pull 请求时，特别需要小心检查所有更改，确保它们和提交时所展示的一致。

开始之前，确保本地 Git 仓库是最新的，并且正确设置了源仓库 [如上所示](https://docs.freebsd.org/en/articles/committers-guide/#keeping_current)。另外，请确保本地仓库有以下远程仓库设置：

```
% git remote -v
freebsd https://git.freebsd.org/src.git (fetch)
freebsd ssh://git@gitrepo.freebsd.org/src.git (push)
github https://github.com/freebsd/freebsd-src (fetch)
github https://github.com/freebsd/freebsd-src (fetch)
```

Pull 请求通常很简单：请求只包含一个提交。在这种情况下，可以使用简化的方法，尽管上一节中介绍的方法也可以使用。这里，创建一个分支，将更改 cherry-pick（挑选提交），调整提交信息，进行测试后再推送。示例中使用了 `staging` 分支，但它可以是任何名称。该技术适用于 Pull 请求中任意数量的提交，尤其是当更改可以干净地应用到 FreeBSD 树时。当有多个提交时，特别是需要进行小调整时，使用 `git rebase -i` 比 `git cherry-pick` 更加有效。简而言之，这些命令会创建一个分支；从 Pull 请求中 cherry-pick 更改；测试；调整提交信息；并快进合并回 `main`。下面用 `$PR` 表示 PR 编号。在调整提交信息时，添加 `Pull Request: https://github.com/freebsd-src/pull/$PR`。所有提交到 FreeBSD 仓库的 Pull 请求都应至少由一人审查。审查者不必是提交者本人，但在这种情况下，提交者应当信任其他审查者的审查能力。在将 Pull 请求推送到仓库之前进行代码审查的提交者，应在提交中添加 `Reviewed by:` 行，因为在这种情况下审查并不是隐式的。还应将任何在 GitHub 上审查并批准该提交的人添加到 `Reviewed by:` 中。一如既往，应仔细检查更改是否完成了预期功能，并且不存在恶意代码。

>**注意**
>
>此外，请检查 Pull 请求的作者名称是否为匿名。GitHub 的 Web 编辑界面会生成类似如下的名称：
>
>```sh
>Author:     github-user <38923459+github-user@users.noreply.github.com>
>```
>
>应礼貌地请求作者提供更好的名称和/或电子邮件。需格外小心，确保不引入风格问题或恶意代码。

```sh
% git fetch github pull/$PR/head:staging
% git rebase -i main staging	# 将 staging 分支更新到最新，调整提交信息
<进行必要的测试>
% git checkout main
% git pull --ff-only		# 若时间已过，确保拉取最新的提交
% git checkout main
% git merge --ff-only staging
<再次测试如果需要>
% git push freebsd --push-option=confirm-author
```

对于复杂的 Pull 请求，如果有多个提交并且有冲突，可以按照以下步骤进行：

1. 检出 Pull 请求 `git checkout github/pull/XXX`
2. 创建一个分支进行 rebase `git checkout -b staging`
3. 将 `staging` 分支 rebase 到最新的 `main` 分支 `git rebase -i main staging`
4. 解决冲突并进行必要的测试
5. 将 `staging` 分支快进合并到 `main` 分支
6. 最后的 sanity check，确保一切正常
7. 推送到 FreeBSD 的 Git 仓库

这些步骤同样适用于将其他地方开发的分支合并到本地树并提交。

完成 Pull 请求后，使用 GitHub 的 Web 界面关闭它。如果你的 `github` 原始库是通过 `https://` 配置的，那么你只需要 GitHub 账户来关闭该 Pull 请求。

## 6. 版本控制历史

项目已经迁移到 [git](https://docs.freebsd.org/en/articles/committers-guide/#git-primer)。

FreeBSD 源代码仓库在 2008 年 5 月 31 日从 CVS 转移到 Subversion。第一个正式的 SVN 提交是 *r179447*。源代码仓库在 2020 年 12 月 23 日从 Subversion 转移到 Git。最后一个正式的 SVN 提交是 *r368820*，第一个正式的 Git 提交哈希是 *5ef5f51d2bef80b0ede9b10ad5b0e9440b60518c*。

FreeBSD `doc/www` 仓库在 2012 年 5 月 19 日从 CVS 转移到 Subversion。第一个正式的 SVN 提交是 *r38821*。文档仓库在 2020 年 12 月 8 日从 Subversion 转移到 Git。最后一个 SVN 提交是 *r54737*，第一个正式的 Git 提交哈希是 *3be01a475855e7511ad755b2defd2e0da5d58bbe*。

FreeBSD `ports` 仓库在 2012 年 7 月 14 日从 CVS 转移到 Subversion。第一个正式的 SVN 提交是 *r300894*。Ports 仓库在 2021 年 4 月 6 日从 Subversion 转移到 Git。最后一个 SVN 提交是 *r569609*，第一个正式的 Git 提交哈希是 *ed8d3eda309dd863fb66e04bccaa513eee255cbf*。

## 7. 设置、约定与传统

作为新开发者，需要做一些准备工作。以下步骤仅适用于已获得提交权限的开发者。如果你没有提交权限，必须由导师来执行这些步骤。

### 7.1. 对于新提交者

已获得 FreeBSD 仓库提交权限的开发者，必须遵循以下步骤。

- 在提交每个更改之前，先获得导师的批准！
- 所有 **src** 的提交首先必须提交到 FreeBSD-CURRENT，然后才能合并到 FreeBSD-STABLE。FreeBSD-STABLE 分支必须保持与该分支早期版本的 ABI 和 API 兼容性。不要合并破坏这种兼容性的更改。

**新提交者步骤**

1. 添加作者条目
   **doc/shared/authors.adoc** - 添加作者条目。后续步骤依赖于此条目，缺少此步骤会导致 **doc/** 构建失败。这是一个相对简单的任务，但仍然是测试版本控制技能的好方法。

2. 更新开发者和贡献者列表
   **doc/shared/contrib-committers.adoc** - 添加一条记录，这样就能在 [贡献者列表](https://docs.freebsd.org/en/articles/contributors/#staff-committers) 的“开发者”部分中看到它。条目按姓氏排序。

   **doc/shared/contrib-additional.adoc** - *删除* 记录。条目按名字排序。

3. 添加新闻项
   **doc/website/data/en/news/news.toml** - 添加一条记录。参考其他用于宣布新提交者的条目并遵循格式。使用提交权限批准邮件中的日期。

4. 添加 PGP 密钥
   Dag-Erling Smørgrav `des@FreeBSD.org` 已编写一个脚本 (**doc/documentation/tools/addkey.sh**) 使得此过程更简单。有关更多信息，请参见 [README](https://cgit.freebsd.org/doc/plain/documentation/static/pgpkeys/README) 文件。

   使用 **doc/documentation/tools/checkkey.sh** 来验证密钥是否符合最低最佳实践标准。

   添加和检查密钥后，将更新后的文件添加到源控制并提交。此文件中的条目按姓氏排序。

   >**注意**
   >
   >确保在仓库中有一个当前的 PGP/GnuPG 密钥非常重要。这个密钥可能会被用来进行提交者的身份验证。例如，`FreeBSD Administrators <admins@FreeBSD.org>` 可能需要它来恢复账户。完整的 `FreeBSD.org` 用户密钥环可以从 [https://docs.FreeBSD.org/pgpkeys/pgpkeys.txt](https://docs.freebsd.org/pgpkeys/pgpkeys.txt) 下载。

5. 更新导师和学员信息
   **src/share/misc/committers-<repository>.dot** - 向当前提交者部分添加一条记录，`repository` 是指 `doc`、`ports` 或 `src`，具体取决于授予的提交权限。

   在底部部分为每个导师/学员关系添加一条记录。

6. 更新 git mailmap 文件
   **src/.mailmap**、**doc/.mailmap** 和 **ports/.mailmap** - 为你在成为 FreeBSD 提交者之前创建的提交添加一条记录。

   将地址映射到你的 FreeBSD 地址，可以让我们更轻松地跟踪可能已经准备好获得提交权限的外部提交者。你也可以使用此功能来修正默认 `git log` 输出中的旧名称、拼写错误的名称等。

7. 生成 Kerberos 密码
   请参阅 [Kerberos 和 LDAP Web 密码配置](https://docs.freebsd.org/en/articles/committers-guide/#kerberos-ldap)，为使用 FreeBSD 集群的其他服务（如 [bug-tracking 数据库](https://bugs.freebsd.org/bugzilla/)）生成或设置 Kerberos 账户（在此步骤中，你将获得 bug-tracking 账户）。

8. 可选：启用 Wiki 账户
   [FreeBSD Wiki](https://wiki.freebsd.org/) 账户 - Wiki 账户允许分享项目和想法。尚未拥有账户的人可以按照 [Wiki/About 页面](https://wiki.freebsd.org/Wiki/About) 上的说明获取账户。如果需要帮助，可以联系 [wiki-admin@FreeBSD.org](mailto:wiki-admin@FreeBSD.org)。

9. 可选：更新 Wiki 信息
   Wiki 信息 - 获得 Wiki 访问权限后，有些人会在 [How We Got Here](https://wiki.freebsd.org/HowWeGotHere)、[IRC Nicks](https://wiki.freebsd.org/IRC/Nicknames)、[Dogs of FreeBSD](https://wiki.freebsd.org/Community/Dogs) 或 [Cats of FreeBSD](https://wiki.freebsd.org/Community/Cats) 页面上添加条目。

10. 可选：在 Ports 中添加个人信息
    **ports/astro/xearth/files/freebsd.committers.markers** 和 **src/usr.bin/calendar/calendars/calendar.freebsd** - 有些人会在这些文件中为自己添加条目，以显示自己所在的位置或生日日期。

11. 可选：避免重复邮件
    订阅 [doc 仓库所有分支的提交消息](https://lists.freebsd.org/subscription/dev-commits-doc-all)、[ports 仓库所有分支的提交消息](https://lists.freebsd.org/subscription/dev-commits-ports-all) 或 [src 仓库所有分支的提交消息](https://lists.freebsd.org/subscription/dev-commits-src-all) 的用户，可能希望取消订阅，以避免接收到提交消息和后续邮件的重复副本。

### 7.2. 对所有人的要求

1. 向其他开发者介绍自己，否则没人知道你是谁，也不知道你在 FreeBSD 中的工作内容。介绍不需要是详尽的传记，只需要写一两段关于你是谁、你计划在 FreeBSD 中做什么以及你的导师是谁的内容。将此邮件发送到 FreeBSD 开发者邮件列表，这样你就可以开始了！
2. 登录到 `freefall.FreeBSD.org` 并创建一个 **/var/forward/user** 文件（其中 *user* 是你的用户名），文件内容包含你希望将邮件转发到的电子邮件地址，邮件地址格式为 *你的用户名*@FreeBSD.org。此邮件包括所有提交消息以及任何发送到 FreeBSD 提交者邮件列表和 FreeBSD 开发者邮件列表的邮件。由于 `freefall` 上有些非常大的邮箱可能会在空间不足时未经警告被截断，因此请将其转发或保存到其他地方。

   >**注意**
   >
   >如果你的电子邮件系统使用 SPF 严格规则，你应该将 `mx2.FreeBSD.org` 排除在 SPF 检查之外。

   由于处理垃圾邮件给中央邮件服务器带来的巨大负担，前端服务器会进行一些基本检查，并根据这些检查丢弃一些邮件。目前，唯一的检查是连接主机的 DNS 信息，但这可能会发生变化。一些人把这些检查归咎于导致有效邮件的丢失。要关闭这些检查，可以在 `freefall.FreeBSD.org` 上创建文件 **~/.spam_lover**。

   >**注意**
   >
   >不是提交者但又是开发者的用户将无法订阅提交者或开发者邮件列表。订阅关系是依访问权限而定的。

#### 7.2.1. SMTP 访问设置

对于那些愿意通过 FreeBSD.org 基础设施发送电子邮件的人，请按照以下说明操作：

1. 将邮件客户端指向 `smtp.FreeBSD.org:587`。
2. 启用 STARTTLS。
3. 确保你的 `From:` 地址设置为 `你的用户名@FreeBSD.org`。
4. 对于身份验证，你可以使用 FreeBSD Kerberos 用户名和密码（请参见 [Kerberos 和 LDAP Web 密码配置](https://docs.freebsd.org/en/articles/committers-guide/#kerberos-ldap)）。首选使用 `你的用户名/mail` 主体，因为它仅用于验证邮件资源。

   >**注意**
   >
   >在输入用户名时不要带 `@FreeBSD.org`。
   >**注意**
   >
   >额外说明：
   >
   >- 只接受来自 `你的用户名@FreeBSD.org` 的邮件。如果你已作为某个用户认证，不允许你从另一个用户发送邮件。
   >
   >- 将附加一个包含 SASL 用户名的头部：（`认证发件人： 用户名`）。
   >
   >- 该主机对邮件发送有各种速率限制，以减少暴力攻击。

##### 7.2.1.1. 使用本地 MTA 将电子邮件转发到 FreeBSD.org SMTP 服务

也可以使用本地 MTA 将本地发送的邮件转发到 FreeBSD.org 的 SMTP 服务器。

**示例 1. 使用 Postfix**

要告诉本地 Postfix 实例将来自 `你的用户名@FreeBSD.org` 的邮件转发到 FreeBSD.org 服务器，可以在 **main.cf** 文件中添加以下内容：

```ini
sender_dependent_relayhost_maps = hash:/usr/local/etc/postfix/relayhost_maps
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/usr/local/etc/postfix/sasl_passwd
smtp_use_tls = yes
```

在 **/usr/local/etc/postfix/relayhost_maps** 中创建以下内容：

```ini
你的用户名@FreeBSD.org  [smtp.freebsd.org]:587
```

在 **/usr/local/etc/postfix/sasl_passwd** 中创建以下内容：

```ini
[smtp.freebsd.org]:587          你的用户名:你的密码
```

如果邮件服务器被其他人使用，你可能希望防止他们从你的地址发送邮件。为此，可以在 **main.cf** 中添加以下内容：

```ini
smtpd_sender_login_maps = hash:/usr/local/etc/postfix/sender_login_maps
smtpd_sender_restrictions = reject_known_sender_login_mismatch
```

在 **/usr/local/etc/postfix/sender_login_maps** 中创建以下内容：

```ini
你的用户名@FreeBSD.org 你的本地用户名
```

其中 *你的本地用户名* 是用于连接到本地 Postfix 实例的 SASL 用户名。

**示例 2. 使用 OpenSMTPD**

要告诉本地 OpenSMTPD 实例将来自 `你的用户名@FreeBSD.org` 的邮件转发到 FreeBSD.org 服务器，可以在 **smtpd.conf** 中添加以下内容：

```ini
action "freebsd" relay host smtp+tls://freebsd@smtp.freebsd.org:587 auth <secrets>
match from any auth 你的本地用户名 mail-from "_你的用户名_@freebsd.org" for any action "freebsd"
```

其中 *你的本地用户名* 是用于连接到本地 OpenSMTPD 实例的 SASL 用户名。

在 **/usr/local/etc/mail/secrets** 中创建以下内容：

```ini
freebsd	你的用户名:你的密码
```

**示例 3. 使用 Exim**

要指示本地 Exim 实例将来自 `example@FreeBSD.org` 的所有邮件转发到 FreeBSD.org 服务器，可以在 Exim **配置文件** 中添加以下内容：

```ini
Routers section: (在列表顶部)
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

在 **/usr/local/etc/exim/freebsd_send** 中创建以下内容：

```ini
example@freebsd.org:smtp.freebsd.org::587
```

### 7.3. 导师

所有新开发者在前几个月都会被分配一个导师。导师的职责是教导学员项目的规则和约定，并引导他们在开发者社区的初步步骤。导师还需要对学员在这一初期阶段的行为负责。

对于提交者：在没有得到导师批准的情况下，不能提交任何内容。提交信息中必须注明 `Approved by:` 行，记录导师的批准。

当导师认为学员已经掌握了基础，并准备独立提交时，导师会在 **mentors** 文件中发布提交。这一文件位于每个代码库的 **admin** 孤立分支中。有关如何访问这些分支的详细信息，请参见 [admin](https://docs.freebsd.org/en/articles/committers-guide/#admin-branch)。

## 8. 提交前审核

代码审查是提高软件质量的一种方式。以下准则适用于对 `src` 仓库的 `main`（-CURRENT）分支的提交。其他分支以及 `ports` 和 `docs` 树有各自的审查政策，但这些准则一般适用于需要审查的提交：

- 所有非琐碎的更改应在提交到仓库之前进行审核。
- 审核可以通过电子邮件、Bugzilla、Phabricator 或其他机制进行。尽可能地，审核应当公开进行。
- 负责代码更改的开发者同样负责进行所有必要的与审核相关的更改。
- 代码审查是一个迭代过程，直到补丁准备好提交为止。具体来说，补丁发送到审查之后，在提交前应当收到明确的“看起来不错”反馈。在这点上，只要是明确的，可以采取任何适合的反馈形式。
- 超时不能作为审查的替代。

有时代码审查会比你预期的时间更长，尤其是对于大型功能。加速补丁审查的有效方法包括：

- 审查他人的补丁。如果你提供帮助，其他人也更愿意为你提供帮助；良好的合作关系是我们的货币。
- 提醒补丁。如果它很紧急，提供为什么这个补丁对你来说很重要的理由，并每隔几天提醒一次。如果不紧急，常见的礼貌提醒频率是每周一次。记住，你是在请求其他专业开发者宝贵的时间。
- 在邮件列表、IRC 等平台寻求帮助。其他人可能能够直接帮助你，或者建议合适的审查者。
- 将补丁分解成多个较小的补丁，这些补丁彼此依赖。补丁越小，别人快速审查它的概率越高。在做大范围更改时，从一开始就保持这一思路是有帮助的，因为事后将大改动拆分成小补丁通常很困难。

开发者应当作为审查者和被审查者参与代码审查。如果有人愿意审查你的代码，你也应该回报他人，审查他们的代码。需要注意的是，虽然任何人都可以审查并提供反馈，只有相关领域的专家才能批准更改。这通常是一个经常与相关代码一起工作的提交者。

在某些情况下，可能没有相关领域的专家。在这种情况下，由经验丰富的开发者进行审查并结合适当的测试，通常也是足够的。

## 9. 提交日志消息

本节提供了一些关于提交日志格式的建议和传统。

### 9.1. 为什么提交信息很重要？

当你在 Git、Subversion 或其他版本控制系统（VCS）中提交更改时，你需要写一段描述该提交的文本——即提交信息。这个提交信息有多重要？是否应该花费一些精力来撰写它？如果你只是写了“修复了一个 bug”，真的有关系吗？

大多数项目有多个开发者，并且持续了一段时间。提交信息是与其他开发者进行沟通的重要方式，无论是当前开发者还是未来的开发者。

FreeBSD 拥有数百名活跃开发者和数十年的历史，涉及数十万次的提交。在这段时间里，开发者社区逐渐意识到好的提交信息有多么重要；有时候，这些经验是通过痛苦的教训获得的。

提交信息至少有三个目的：

- 与其他开发者沟通
  FreeBSD 的提交会生成邮件发送到多个邮件列表。这些邮件包括提交信息和补丁本身的副本。提交信息还可以通过如 `git log` 等命令查看。这些信息有助于让其他开发者了解正在进行的更改；这些开发者可能希望测试该更改、对该主题有兴趣并希望更详细地审查，或者正在进行的项目可能会受益于与之的互动。

- 使更改可发现
  在一个拥有悠久历史的大型项目中，当调查某个问题或行为变化时，可能很难找到相关的更改。详细且冗长的提交信息可以使搜索与之相关的更改变得容易。例如，使用 `git log --since 1year --grep 'USB timeout'` 可以查找相关的历史更改。

- 提供历史文档
  提交信息为未来的开发者提供了更改的文档，也许是多年或几十年后。未来的开发者甚至可能是你，作为原作者。当今天看似显而易见的更改，可能在将来就变得不再那么明显了。

`git blame` 命令会标注源代码文件中的每一行，显示将其引入的更改（哈希值和主题行）。

在明确了提交信息的重要性之后，接下来是一个好的 FreeBSD 提交信息的要素：

### 9.2. 从主题行开始

提交信息应以一行简短的主题开始，简要总结该更改。主题本身应该允许读者快速判断更改是否值得关注。

### 9.3. 保持主题行简短

主题行应尽可能简短，同时保留必要的信息。这是为了使浏览 Git 日志更高效，并确保 `git log --oneline` 能在一个 80 列的行内显示短哈希和主题。一个好的经验法则是保持在 67 个字符以内，并尽可能控制在 50 个字符以内。

### 9.4. 如果适用，前缀主题行以组件名

如果更改涉及某个特定组件，主题行可以以该组件名称和冒号（`:`）为前缀。如果适用，请尝试使用与之前对相同文件提交时使用的前缀相同的前缀。

✓ `foo: 添加 -k 选项以保持临时数据`

将前缀包含在建议的 67 字符限制内，以便 `git log --oneline` 不会换行。

### 9.5. 主题行首字母大写

主题行的首字母应大写。前缀（如果有）除非必要，否则不需要大写（例如，`USB:` 需要大写）。

### 9.6. 不要以标点符号结尾

主题行不要以句号或其他标点符号结尾。在这方面，主题行像报纸头条一样。

### 9.7. 使用空行分隔主题和正文

主题和正文之间应使用空行进行分隔。

一些简单的提交不需要正文，只有主题。

✓ `ls: 修复使用文本中的拼写错误`

### 9.8. 限制消息的行宽为 72 列

`git log` 和 `git format-patch` 会将提交信息缩进 4 个空格。限制在 72 列内能确保右侧有匹配的边距。将消息限制在 72 个字符以内还能够使提交信息在格式化补丁中低于 RFC 2822 提出的电子邮件行长度限制（78 字符）。这个限制对多种工具有良好的兼容性，避免了较长行长度带来的换行不一致问题。

### 9.9. 使用现在时，命令式语气

这有助于简短的主题行，并保持一致性，包括自动生成的提交信息（例如，`git revert` 生成的提交信息）。当阅读一系列提交主题时，这一点尤为重要。将主题视作完成句子“当应用时，该更改将会……”的部分。

✓ `foo: 实现 -k（keep）选项`
✗ `foo: 实现了 -k 选项`
✗ `该更改在 foo 中实现了 -k 选项`
✗ `添加了 -k 选项`

### 9.10. 聚焦于“做了什么”和“为什么做”，而不是“如何做”

解释更改实现了什么，以及为什么要做这个更改，而不是陈述具体如何实现。

不要假设读者熟悉该问题。解释更改的背景和动机。如果有基准数据，提供这些数据。

如果更改有任何限制或不完整的方面，在提交信息中描述它们。

### 9.11. 考虑是否可以将提交信息中的某些部分写成代码注释

有时在编写提交信息时，你可能会写下一两句话，解释某些复杂或令人困惑的更改。在这种情况下，考虑是否将这些解释作为代码中的注释，这样它们可以直接帮助未来的开发者（包括你自己）理解代码。

### 9.12. 为将来的自己编写提交信息

当你为一个更改编写提交信息时，你心中已经掌握了所有相关背景——是什么促使了这次更改，考虑过并拒绝的其他方案、更改的限制等等。想象一下自己在一两年后再次查看这个更改时，写提交信息的方式应该包含所有这些必要的背景信息，以帮助未来的你理解为什么做出这个更改。

### 9.13. 提交信息应该是自包含的

你可以在提交信息中引用邮件列表帖子、基准测试结果网页或代码审查链接。然而，提交信息应包含所有相关信息，以防这些引用在未来不再可用。

类似地，一个提交可能会引用之前的提交，比如修复 bug 或回退更改的情况。除了提交标识符（修订号或哈希值），还应包括引用提交的主题行（或其他合适的简短引用）。随着每次版本控制系统（VCS）迁移（从 CVS 到 Subversion，再到 Git），早期系统的修订标识符可能变得难以追溯。

### 9.14. 在脚注中包含适当的元数据

除了在每个提交中包含有意义的消息外，可能还需要附加一些额外的信息。

这些信息由一行或多行构成，每行包含关键字或短语、一个冒号、制表符进行格式化，然后是附加信息。

对于那些可以有多个值的关键字（例如，`PR:` 后跟以逗号分隔的 PR 列表），可以多次使用相同的关键字，以避免歧义或提高可读性。

关键字或短语如下：

| **`PR:`**                | 受此提交影响的缺陷报告（如果有的话）（通常是通过关闭此报告）。多个 PR 可以在一行中列出，使用逗号或空格分隔。                     |
| ------------------------ | ---------------------------------------------------------------------------- |
| **`Reported by:`**       | 报告问题的人的姓名和电子邮件地址；对于开发者，通常仅使用 FreeBSD 集群上的用户名。通常在没有 PR 的情况下使用，例如问题是通过邮件列表报告的。 |
| **`Submitted by:`**（已废弃） | 提交了更改但未提供完整有效补丁（尤其是没有有效电子邮件）的作者姓名。提交的补丁应通过 `git commit --author` 设置作者，提供完整姓名和有效电子邮件地址。在迁移到 Git 允许分开作者和提交者字段之前，此字段用于贡献的补丁。          |
| `Reviewed by:` | 审查此更改的人员的姓名和电子邮件地址；对于开发者，只需提供 FreeBSD 集群上的用户名。如果补丁提交到邮件列表进行审查，并且审查结果是积极的，那么只需包括列表名称。如果审查者不是项目成员，则提供姓名、电子邮件，并在 Ports 的情况下，如果是外部角色，如维护者。开发者审查：`Reviewed by: 用户名`。非开发者的 Ports 维护者审查：`Reviewed by: 全名 <valid@email> (maintainer)`。 |
| `Tested by:` | 测试此更改的人员的姓名和电子邮件地址；对于开发者，只需提供 FreeBSD 集群上的用户名。 |
| `Discussed with:` | 提供有意义反馈、对补丁做出贡献的人员的姓名和电子邮件地址；对于开发者，只需提供 FreeBSD 集群上的用户名。通常用于感谢那些没有明确审查、测试或批准更改，但仍然为更改周围的讨论做出贡献，从而改进并更好地理解其对 FreeBSD 项目的影响的人。 |
| `Approved by:` | 批准更改的人员的姓名和电子邮件地址；对于开发人员，仅为 FreeBSD 集群中的用户名。有几种情况是常见的需要批准：当一个新提交者处于导师指导下；提交到被 LOCKS 文件（src）覆盖的区域；在发布周期期间；提交到你没有提交权限的仓库（例如，src 提交者提交到 docs）；提交到由其他人维护的 Port。在导师指导下时，在提交之前获得导师的批准。在此字段中输入导师的用户名，并注明他们是导师：`Approved by: 导师的用户名 (mentor)`。如果是团队批准了这些提交，则应包含团队名称，后跟括号中的批准者用户名。例如：`Approved by: re (用户名)`。 |
| `Obtained from:`         | 获取代码的项目名称（如果有的话）。不要使用此行来注明个人的姓名。                                                                                                                                                                                                                                                                                                                            |
| `Fixes:`                 | 此更改修复的提交的 Git 短哈希和标题行，通过 `git log -n 1 --pretty=format:'%h ("%s")' GIT-COMMIT-HASH` 返回。                                                                                                                                                                                                                                                                     |
| `MFC after:`             | 若要接收在稍后日期 MFC 的电子邮件提醒，请指定计划进行 MFC 的天数、周数或月数。                                                                                                                                                                                                                                                                                                                |
| `MFC to:`                | 如果提交应合并到某些稳定分支，请指定分支名称。                                                                                                                                                                                                                                                                                                                                     |
| `MFH:`                   | 如果提交将合并到某个 ports 季度分支名称，请指定季度分支。例如 `2021Q2`。                                                                                                                                                                                                                                                                                                                |
| `Relnotes:`              | 如果该更改是下一个发布版本的发布说明的候选项，请设置为 `yes`。候选项包括用户可见的更改、新功能、兼容性破坏等。如果忘记设置此行，或想提供更多细节，可在 src 树根目录的 `RELNOTES` 文件中添加条目。`RELNOTES` 文件用于生成下一个发布版本的发布说明。请勿使用 `Relnotes:` 行描述更改：其唯一有效值是 `yes`。                                                                                                                                                                                                                                                                                                                          |
| `Security:`              | 如果更改与安全漏洞或安全暴露相关，请包括一个或多个参考或问题描述。如果可能，包含 VuXML URL 或 CVE ID。                                                                                                                                                                                                                                                                                                |
| `Event:`                 | 进行此提交的事件描述。如果这是一个定期事件，请在其中添加年份甚至月份。例如，可以写为 `FooBSDcon 2019`。这一行的目的是对会议、聚会及其他类型的聚集给予认可，并展示这些活动的意义。请不要将此行用于 `Sponsored by:`，该行用于注明资助特定功能或开发人员的组织。                                                                                                                                                                                                             |
| `Sponsored by:`          | 为此更改提供赞助的组织（如果有的话）。多个组织请用逗号分隔。如果仅部分工作得到资助，或不同作者获得了不同程度的资助，请在每个赞助商名称后给出适当的说明。例如，`Example.com (alice, code refactoring), Wormulon (bob), Momcorp (cindy)` 表示 Alice 得到了 Example.com 的资助以进行代码重构，Wormulon 资助了 Bob 的工作，Momcorp 资助了 Cindy 的工作。其他作者未获得资助或选择不列出资助。                                                                                                   |
| `Pull Request:`          | 该更改作为拉取请求或合并请求提交到 FreeBSD 的公共只读 Git 仓库。应包括拉取请求的完整 URL，因为它们通常作为代码审查。例如：`https://github.com/freebsd/freebsd-src/pull/745`                                                                                                                                                                                                                                     |
| `Closes:`                | 此更改结束了在指定 GitHub Pull 请求中讨论的补丁系列，并关闭该请求。应包括拉取请求的完整 URL，因为它们通常作为代码审查。例如：`https://github.com/freebsd/freebsd-src/pull/745`                                                                                                                                                                                                                                      |
| `Co-authored-by:`        | 提交的其他作者的姓名和电子邮件地址。GitHub 对 Co-authored-by 说明有详细描述，详情见 [https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors](https://docs.github.com/en/pull-requests/committing-changes-to-your-project/creating-and-editing-commits/creating-a-commit-with-multiple-authors)。 |
| `Signed-off-by:`         | 该 ID 认证符合 [https://developercertificate.org/](https://developercertificate.org/) 的要求。                                                                                                                                                                                                                                                                       |
| `Differential Revision:` | Phabricator 审查的完整 URL。此行 *必须是最后一行*。例如：`https://reviews.freebsd.org/D1708`。                                                                                                                                                                                                                                                                                  |


**示例 4. 基于 PR 的提交日志**

该提交基于 John Smith 提交的 PR 中的补丁。提交消息中的 "PR" 字段已填写。

```sh
...

PR:		12345
```

提交者使用 `git commit --author "John Smith <John.Smith@example.com>"` 设置补丁的作者。

**示例 5. 需要审核的提交日志**

虚拟内存系统正在进行更改。补丁已发布到适当的邮件列表（在此案例中为 `freebsd-arch`），并且更改已获得批准。

```sh
...

Reviewed by:	-arch
```

**示例 6. 需要批准的提交日志**

提交一个 Port，在与列出的 MAINTAINER 一起工作后，MAINTAINER 允许继续提交。

```sh
...

Approved by:	abc (maintainer)
```

其中 *abc* 是批准人的账户名。

**示例 7. 引入来自 OpenBSD 的代码的提交日志**

提交一些基于 OpenBSD 项目工作的代码。

```sh
...

Obtained from:	OpenBSD
```

**示例 8. 提交到 FreeBSD-CURRENT 的更改，计划在稍后日期合并到 FreeBSD-STABLE**

提交一些代码，该代码将在两周后从 FreeBSD-CURRENT 合并到 FreeBSD-STABLE 分支。

```sh
...

MFC after:	2 weeks
```

其中 *2* 是计划进行 MFC 的天数、周数或月数。`weeks` 选项可以是 `day`、`days`、`week`、`weeks`、`month`、`months`。

这些信息常常需要组合在一起。

考虑以下情况：一个用户提交了包含来自 NetBSD 项目的代码的 PR。查看 PR 后，开发人员发现这是他们通常不工作的区域，因此他们让 `arch` 邮件列表审核了该更改。由于该更改复杂，开发人员决定在一个月后进行 MFC，以便进行充分的测试。

提交中应包含的额外信息如下：

**示例 9. 组合提交日志示例**

```sh
PR:		54321
Reviewed by:	-arch
Obtained from:	NetBSD
MFC after:	1 month
Relnotes:	yes
```

## 10. 新文件的首选许可证

FreeBSD 项目的完整许可证政策记录在 [FreeBSD 许可证政策](https://docs.freebsd.org/en/articles/license-guide/) 文章中。本节的其余部分旨在帮助你入门。作为一条规则，若有疑问，请询问。提供建议比修复源代码树更容易。

强烈推荐使用仅 SPDX 标记。FreeBSD 项目使用以下文本作为首选许可证：

```c
/*
 * Copyright (c) [year] [your name]
 *
 * SPDX-License-Identifier: BSD-2-Clause
 */
```

首选顺序是版权声明在前，然后是 `SPDX-License-Identifier`，但两种顺序均可接受。对 FreeBSD 的新贡献应使用 BSD-2-Clause 许可证。FreeBSD 项目不允许在新代码中使用"广告条款"。如果树中的代码包含广告条款，请考虑切换到不含该条款的许可证。

FreeBSD 项目不鼓励使用全新的许可证或标准许可证的变体。新许可证需要得到 [core@FreeBSD.org](mailto:core@FreeBSD.org) 的批准才能存入 `src` 仓库。树中使用的许可证越多，给那些希望利用这些代码的人带来的问题就越多，通常是由于不清晰的许可证条款导致的意外后果。

项目政策要求，某些非 BSD 许可证下的代码只能放置在仓库的特定部分，在某些情况下，编译必须是有条件的，甚至默认禁用。例如，GENERIC 内核必须仅在与 BSD 许可证相同或实质上相似的许可证下编译。GPL、APSL、CDDL 等许可证的软件不得编译到 GENERIC 中。

开发者需注意，在开源中，正确处理 "开源" 和 "源代码" 一样重要，因为不当处理知识产权会带来严重后果。任何问题或疑虑应立即向核心团队报告。

## 11. 跟踪授予 FreeBSD 项目的许可证

FreeBSD 项目的仓库中存在一些软件或数据，已授予特殊许可证以便我们使用。一个例子是 Terminus 字体，用于 [vt(4)](https://man.freebsd.org/cgi/man.cgi?query=vt&sektion=4&format=html)。在这里，作者 Dimitar Zhekov 允许我们使用 `Terminus BSD Console` 字体，并为其提供了一个 2 条款 BSD 许可证，而不是他通常使用的开放字体许可证。

显然，记录所有此类许可证授权是明智的。为此，[core@FreeBSD.org](mailto:core@FreeBSD.org) 决定保持它们的档案。每当 FreeBSD 项目被授予特殊许可证时，我们要求通知 [core@FreeBSD.org](mailto:core@FreeBSD.org)。任何参与安排此类许可证授权的开发者，请将以下详细信息发送给 [core@FreeBSD.org](mailto:core@FreeBSD.org)：

- 授予特殊许可证的个人或组织的联系信息。
- 仓库中哪些文件、目录等受该许可证授权，包括提交的任何特别授权材料的修订号。
- 该许可证生效的日期。除非另有约定，否则此日期应为软件作者授予许可证的日期。
- 许可证文本。
- 适用于 FreeBSD 使用授权材料的任何限制、局限或例外。
- 任何其他相关信息。

待 [core@FreeBSD.org](mailto:core@FreeBSD.org) 确认所有必要的详细信息已收集并正确无误，秘书将发送一封包含许可证详细信息的 PGP 签名确认函。此确认函将永久存档，并作为我们永久的许可证授予记录。

许可证档案应仅包含许可证授权的详细信息；此处不应讨论任何关于许可证或其他主题的内容。访问许可证档案中的数据将根据请求提供给 [core@FreeBSD.org](mailto:core@FreeBSD.org)。

## 12. 树中的 SPDX 标签

该项目在源代码库中使用 [SPDX](https://spdx.dev/) 标签。对于新文件，强烈推荐使用仅 SPDX 标记（不复制完整的许可证文本）。当文件仅包含版权声明和 `SPDX-License-Identifier` 标签时，该 SPDX 标签指定了该文件的许可证。当文件同时包含 `SPDX-License-Identifier` 标签和完整的许可证原文时，SPDX 标签为信息性标签，出现不一致时以许可证原文为准。该项目从 SPDX 的有效[短许可证标识符列表](https://spdx.org/licenses/)中提取标识符，并仅使用 `SPDX-License-Identifier` 标签。有关 SPDX 表达式在 FreeBSD 软件集中使用的完整规范，请参见 [FreeBSD 许可证政策](https://docs.freebsd.org/en/articles/license-guide/) 文章。

## 13. 开发者关系

当你在自己的代码上直接工作，或者在已经明确由你负责的代码上工作时，通常无需在提交前与其他提交者进行确认。当你在系统中某个明显无人维护的区域修复 bug（我们为此感到羞愧的确存在一些此类区域）时，同样适用。修改系统中由正式或非正式方式维护的部分时，可以考虑请求审查，就像开发者在成为提交者之前会做的一样。对于 ports，联系 **Makefile** 中列出的 `MAINTAINER`。

要判断某个区域是否被维护，可以查看树根目录中的 MAINTAINERS 文件。如果没有列出任何人，请扫描修订历史，查看过去谁提交了更改。要列出过去两年内给定文件的所有提交作者的名字和电子邮件地址，并按提交次数降序排列，可以使用：

```sh
% git -C /path/to/repo shortlog -sne --since="2 years" -- relative/path/to/file
```

如果查询没有得到回应，或者提交者以其他方式表明对该区域缺乏兴趣，可以继续提交。

>**重要**
>
>避免向维护者发送私人邮件。其他人可能对这次对话感兴趣，而不仅仅是最终的结果。

如果对提交有任何疑虑，请在提交之前进行审查。最好在提交前得到批评，而不是等到它已经是仓库的一部分后再遭到批评。如果提交导致争议爆发，可能需要考虑在问题解决之前撤销更改。记住，使用版本控制系统我们总是可以把它恢复回来。

不要诋毁他人的想法。如果他们看到的问题解决方案不同，甚至问题本身不同，可能并不是因为他们愚蠢，或者有可疑的出身，或者他们想摧毁辛勤的工作、个人形象或 FreeBSD，而是因为他们有不同的世界观。和而不同。

诚实地表达分歧。从解决方案的优点出发辩论，诚实地指出其不足，并且愿意以开放的心态看待他们的解决方案，甚至是他们对问题的看法。

接受修正。我们都是容易犯错的。当你犯错时，道歉并继续前进。不要自责，也不要因为错误去责怪别人。不要浪费时间在尴尬或自责上，赶紧修正问题并继续前进。

寻求帮助。寻找（并提供）同行审查。开源软件的一个优势是有更多人参与其中的审查；如果没有人愿意审查代码，那就失去了这一优势。

## 14. 如果有疑虑……

当你不确定某件事时，无论是技术问题还是项目约定，都要确保向他人请教。如果你保持沉默，你将永远无法取得进展。

如果是技术问题，请在公共邮件列表上提问。避免私下给知道答案的个人发送邮件。这样，所有人都可以从问题和答案中学习。

对于与项目相关的或行政性的问题，请按照以下顺序提问：

- 你的导师或前导师。
- 在 IRC、邮件等平台上的有经验的提交者。
- 任何拥有“帽子”的团队，他们能给你一个权威的答案。
- 如果仍然不确定，可以在 FreeBSD 开发者邮件列表上提问。

如果你的问题得到解答，而且没有人指出有文档明确回答了你的问题，请记录下来，因为其他人也会有同样的问题。

## 15. Bugzilla

FreeBSD 项目使用 Bugzilla 来追踪 bug 和变更请求。如果你提交了一个修复或建议，并且该内容已经在 PR 数据库中存在，请确保关闭该 PR。如果你有时间，也可以考虑关闭与你的提交相关的其他 PR。

拥有非 `FreeBSD.org` Bugzilla 账户的提交者，可以通过以下步骤将旧账户与 `FreeBSD.org` 账户合并：

1. 使用旧账户登录。
2. 打开一个新的 bug，选择 `Services` 作为产品，选择 `Bug Tracker` 作为组件。在 bug 诉说中列出你希望合并的账户。
3. 使用 `FreeBSD.org` 账户登录，并在新开 bug 中发表评论以确认所有权。有关如何为 `FreeBSD.org` 账户生成或设置密码的更多信息，请参见 [Kerberos 和 LDAP Web 密码](https://docs.freebsd.org/en/articles/committers-guide/#kerberos-ldap)。
4. 如果要合并的账户超过两个，请从每个账户发布评论。

你可以在以下链接了解更多关于 Bugzilla 的信息：

- [FreeBSD 问题报告处理指南](https://docs.freebsd.org/en/articles/pr-guidelines/)
- [https://www.FreeBSD.org/support](https://www.freebsd.org/support/)

## 16. Phabricator

FreeBSD 项目使用 [Phabricator](https://reviews.freebsd.org/) 来处理代码审查请求。详细信息请参见 [Phabricator wiki 页面](https://wiki.freebsd.org/Phabricator)。

拥有非 `FreeBSD.org` Phabricator 账户的提交者，可以通过以下步骤将旧账户重命名为 `FreeBSD.org` 账户：

1. 将你的 Phabricator 账户电子邮件更改为你的 `FreeBSD.org` 邮件。
2. 使用你的 `FreeBSD.org` 账户，在我们的 bug 跟踪系统中打开一个新的 bug，更多信息请参见 [Bugzilla](https://docs.freebsd.org/en/articles/committers-guide/#bugzilla)。选择 `Services` 作为产品，选择 `Code Review` 作为组件。在 bug 描述中请求将你的 Phabricator 账户重命名，并提供你的 Phabricator 用户链接。例如，`https://reviews.freebsd.org/p/bob_example.com/`

>**重要**
>
>Phabricator 账户不能合并，请勿创建新账户。

## 17. 谁是谁

除了仓库管理者之外，还有其他 FreeBSD 项目成员和团队，你可能会在作为提交者的过程中接触到。简要介绍如下（并非全面列举）：

`Documentation Engineering Team (doceng@FreeBSD.org)`：doceng 团队负责文档构建基础设施，批准新的文档提交者，并确保 FreeBSD 网站和 FTP 站点上的文档与 Subversion 树保持同步。该团队不处理冲突解决问题。绝大多数与文档相关的讨论发生在 [FreeBSD 文档项目邮件列表](https://lists.freebsd.org/subscription/freebsd-doc) 上。关于 doceng 团队的更多信息，请参见其 [章程](https://www.freebsd.org/internal/doceng/)。希望参与文档工作的提交者应熟悉 [文档项目入门指南](https://docs.freebsd.org/en/books/fdp-primer/)。

`Konstantin Belousov (kib@FreeBSD.org), Dave Cottlehuber (dch@FreeBSD.org), Marc Fonvieille (blackend@FreeBSD.org), John Hixson (jhixson@FreeBSD.org), Xin Li (delphij@FreeBSD.org), Ed Maste (emaste@FreeBSD.org), Mahdi Mokhtari (mmokhi@FreeBSD.org), Colin Percival (cperciva@FreeBSD.org), Doug Rabson (dfr@FreeBSD.org), Muhammad Moinur Rahman (bofh@FreeBSD.org)`：这些成员属于 `Release Engineering Team (re@FreeBSD.org)`。该团队负责设置发布截止日期和控制发布过程。在代码冻结期间，发布工程师对即将发布版本的系统更改拥有最终权力。如果你希望从 FreeBSD-CURRENT 合并更改到 FreeBSD-STABLE（无论在任何给定时刻这些值为何），这些人是你应该联系的对象。

`Gordon Tetlow (gordon@FreeBSD.org)`：`Gordon Tetlow` 是 [FreeBSD 安全官](https://www.freebsd.org/security/)，负责管理 `Security Officer Team (security-officer@FreeBSD.org)`。

FreeBSD 提交者邮件列表 `{dev-src-all}`、`{dev-ports-all}` 和 `{dev-doc-all}` 是版本控制系统用来发送提交消息的邮件列表。*永远*不要直接向这些列表发送邮件。只有在邮件简短且与提交直接相关时，才可以回复这些列表。

FreeBSD 开发者邮件列表：所有提交者都已订阅该列表。这个列表是为提交者的“社区”问题设立的论坛。举例来说，核心投票、公告等。

FreeBSD 开发者邮件列表仅供 FreeBSD 提交者使用。为了开发 FreeBSD，提交者必须能够公开讨论将要解决的问题，直到它们公开发布之前。正在进行中的工作讨论不适合公开发布，否则可能会对 FreeBSD 产生不利影响。

所有 FreeBSD 提交者应确保未经所有作者许可，不将 FreeBSD 开发者邮件列表的消息公开或转发给列表外的人。违规者将被从 FreeBSD 开发者邮件列表中移除，从而暂停提交权限。反复或严重违反者可能会被永久撤销提交权限。

该列表 *不是* 用于代码审查或任何技术讨论的地方。实际上，使用该列表进行此类讨论会对 FreeBSD 项目造成伤害，因为它给人一种错误印象，认为所有影响到 FreeBSD 使用社区的决策都是在一个封闭的列表中做出的，而没有做到“公开”。最后但同样重要的是 *绝对不要* 向 FreeBSD 开发者邮件列表发送电子邮件并将其他 FreeBSD 邮件列表作为抄送/密件抄送对象。这样做会极大地削弱该列表的作用。

## 18. SSH 快速入门指南

1. 如果你不希望每次使用 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 时都输入密码，并且你使用密钥进行身份验证，[ssh-agent(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-agent&sektion=1&format=html) 可以为你提供便利。如果你想使用 [ssh-agent(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-agent&sektion=1&format=html)，请确保在运行其他应用程序之前先启动它。例如，X 用户通常会在他们的 **.xsession** 或 **.xinitrc** 中执行此操作。有关详细信息，请参阅 [ssh-agent(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-agent&sektion=1&format=html)。

2. 使用 [ssh-keygen(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-keygen&sektion=1&format=html) 生成密钥对。密钥对将保存在你的 **$HOME/.ssh/** 目录中。
   >**重要**
   >
   >仅支持 ECDSA、Ed25519 或 RSA 密钥。

3. 将你的公钥（**$HOME/.ssh/id_ecdsa.pub**、**$HOME/.ssh/id_ed25519.pub** 或 **$HOME/.ssh/id_rsa.pub**）发送给设置你为提交者的人，以便将其放入 `freefall` 上的 **/etc/ssh-keys/** 目录中的 **yourlogin**。

现在，你可以使用 [ssh-add(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-add&sektion=1&format=html) 进行一次会话的身份验证。它会提示输入私钥的密码短语，然后将其存储在身份验证代理（[ssh-agent(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-agent&sektion=1&format=html)）中。使用 `ssh-add -d` 可以从代理中移除存储的密钥。

通过一个简单的远程命令进行测试：`ssh freefall.FreeBSD.org ls /usr`。

有关更多信息，请参阅 [security/openssh-portable](https://cgit.freebsd.org/ports/tree/security/openssh-portable/)、[ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html)、[ssh-add(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-add&sektion=1&format=html)、[ssh-agent(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-agent&sektion=1&format=html)、[ssh-keygen(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-keygen&sektion=1&format=html) 和 [scp(1)](https://man.freebsd.org/cgi/man.cgi?query=scp&sektion=1&format=html)。

有关添加、修改或删除 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 密钥的信息，请参阅 [此文章](https://wiki.freebsd.org/clusteradm/ssh-keys)。

## 19. FreeBSD 提交者的 Coverity® 可用性

所有 FreeBSD 开发者都可以访问 FreeBSD 项目软件的 Coverity 分析结果。所有有兴趣获取自动化 Coverity 运行的分析结果的人，可以在 [Coverity Scan](http://scan.coverity.com/) 注册。

FreeBSD 维基包含了一个小指南，供有兴趣使用 Coverity® 分析报告的开发者参考：[https://wiki.freebsd.org/CoverityPrevent](https://wiki.freebsd.org/CoverityPrevent)。请注意，此小指南仅供 FreeBSD 开发者阅读，因此如果你无法访问此页面，你需要请求某人将你添加到适当的 Wiki 访问列表中。

最后，所有打算使用 Coverity® 的 FreeBSD 开发者都应随时通过向 FreeBSD 开发者的邮件列表提问来获取更多的详细信息和使用信息。

## 20. FreeBSD 提交者的详细规则清单

所有参与 FreeBSD 项目的人员都应遵守 *行为规范*，可以从 [https://www.FreeBSD.org/internal/code-of-conduct](https://www.freebsd.org/internal/code-of-conduct/) 获取该规范。作为提交者，你是项目的公众面貌，你的行为对公众对该项目的看法有着至关重要的影响。本指南扩展了 *行为规范* 中与提交者相关的部分。

1. 尊重其他提交者。
2. 尊重其他贡献者。
3. 在提交之前讨论任何重大更改。
4. 尊重现有的维护者（若在 **Makefile** 中的 `MAINTAINER` 字段或顶层目录中的 **MAINTAINER** 字段中列出）。
5. 如果维护者要求，一切有争议的更改必须在争议解决之前撤回。与安全相关的更改可以在安全官员的裁定下覆盖维护者的意见。
6. 更改应先提交到 FreeBSD-CURRENT，再提交到 FreeBSD-STABLE，除非发布工程师特别允许，或者它们不适用于 FreeBSD-CURRENT。任何非琐碎或非紧急的更改应该在 FreeBSD-CURRENT 中至少等待 3 天，以便进行充分的测试。发布工程师对 FreeBSD-STABLE 分支拥有与规则 #5 中维护者相同的权力。
7. 不要与其他提交者公开争吵；这会显得不好。
8. 尊重所有代码冻结，及时阅读 `committers` 和 `developers` 邮件列表，了解何时实施代码冻结。
9. 对于任何程序有疑问时，先询问！
10. 提交更改之前请先进行测试。
11. 在没有 *明确* 获得相关维护者的批准的情况下，不要提交外部软件。

如上所述，违反这些规则可能会导致暂停提交权限，或者在多次违规后永久撤销提交权限。核心成员有权暂时暂停提交权限，直到核心小组有机会审查该问题。在“紧急情况”（如提交者对代码库造成损害）下，代码库管理员也可以暂时暂停权限。只有 2/3 多数的核心成员有权暂停提交权限超过一周，或永久撤销权限。此规则的存在并不是为了让核心成为一群残酷的独裁者，而是为了给项目提供一种安全保护措施。如果某人失控，能够立即采取行动而不被辩论拖延是非常重要的。在所有情况下，提交者的权限被暂停或撤销时，有权要求由核心进行“听证”，暂停的总时长将在那时确定。提交者在暂停权限后还可以在 30 天后请求复审，并且每 30 天复审一次（除非暂停期少于 30 天）。完全撤销权限的提交者可以在 6 个月后请求复审。该复审政策是 *严格非正式的*，在所有情况下，核心小组保留在认为原决策正确时可以选择是否审理复审请求的权利。

在项目运营的其他方面，核心小组是提交者的一个子集，必须遵守 *相同的规则*。仅仅因为某人是核心成员，并不意味着他们可以越过任何已定的规则；核心小组的“特别权力”仅在作为小组行动时才生效，而非个人行为。作为个体，核心团队成员首先是提交者，其次才是核心成员。

### 20.1. 细节

1. 尊重其他提交者。
   这意味着你需要将其他提交者视为同行开发者。尽管我们偶尔会试图证明相反的事情，但成为提交者并不是因为愚蠢，而最让人不爽的事情就是被同行如此对待。无论我们是否总是感到彼此尊重（每个人都有情绪低落的时候），我们仍然必须始终*以尊重的态度*对待其他提交者，无论是在公开论坛上还是在私人邮件中。

   长期合作的能力是这个项目最宝贵的资产，它远比任何代码更重要，将关于代码的争论转化为影响我们长期和谐合作的问题，是无法接受的。

   为了遵守这一规则，当你生气时，不要发送电子邮件，或者以可能让他人感到不必要对抗的方式行事。先冷静下来，然后思考如何以最有效的方式沟通，来让对方相信你的论点是正确的，而不是为了短期内的宣泄而进行一场长期的争论。这不仅是非常不好的“能量经济学”，反复在公众场合表现出攻击性行为会严重影响我们合作的能力，项目领导层会严肃处理，并可能导致提交权限的暂停或终止。项目领导层会考虑到公开和私下的沟通，但不会主动披露私人沟通，除非相关提交者自愿提供。

   这些都不是项目领导层希望看到的，但团结至上。没有任何代码或好建议值得为此做出妥协。

2. 尊重其他贡献者。
   你曾不是提交者。曾几何时，你也是位贡献者。请始终记住这一点。记住曾经尝试寻求帮助和关注的感受。不要忘记你作为贡献者时的感受。记住那个时候，自己是多么重要。不要打击、贬低或轻视贡献者。尊重他们。他们是我们未来的提交者。与提交者一样，他们对项目的贡献同样重要。毕竟，在成为提交者之前，你也是如此贡献过。切记这一点。

   请参考[尊重其他提交者](https://docs.freebsd.org/en/articles/committers-guide/#respect)中的要点，并同样适用于贡献者。

3. 在提交任何重大更改之前*进行讨论*。
   仓库不是初次提交更改的地方，那里是用来纠正错误或者争论的，通常是在邮件列表或通过使用 Phabricator 服务时进行。提交只有在达成一定共识后才会发生。这并不意味着在纠正每个明显的语法错误或手册页拼写错误之前都需要得到许可，只是要培养一种感觉，当提议的更改不是那么显而易见时，最好先征求一些反馈。在一般情况下，如果某个变更明显比之前的版本更好，人们不介意大规模的更改，但他们不喜欢被 *突然* 改变。确保事情走上正轨的最佳方式是让一个或多个提交者进行代码审查。

   如果不确定，请请求审查！

4. 尊重现有的维护者（如果已列出）。
   FreeBSD 的许多部分并不是“由某个特定的个人拥有”，换句话说，没人会跳出来大喊“不要改动我的区域”，但在某些情况下，最好先确认一下。我们使用的一种惯例是在 **Makefile** 中为任何正在积极维护的包或子树列出维护者；请参阅[源代码树指南和政策](https://docs.freebsd.org/en/books/developers-handbook/#policies)中的相关文档。如果某些代码段有多个维护者，一个维护者对相关区域的提交需要至少由另一个维护者进行审查。在不清楚“维护者身份”的情况下，可以查看相应文件的仓库日志，看看是否有某个人最近在该区域进行过大量工作。

5. 任何有争议的更改必须在维护者要求时撤回，直到争议解决。如果涉及安全相关的更改，则可以在安全官员的裁量下覆盖维护者的意愿。
   在冲突时（每一方都坚信自己是对的），这可能很难接受，但版本控制系统让我们不必让争论继续下去，直接撤回有争议的更改，冷静下来后再找出最佳的解决方案。如果更改最终被证明是正确的，完全可以轻松地恢复。如果结果证明这不是最好的选择，那么用户就不必在争论期间忍受无效的更改。很少会有人在仓库中要求撤回更改，因为大多数争议都在提交前通过讨论解决，但在这种罕见的情况下，撤回更改应毫不犹豫，以便我们立即着手解决它是否合理的问题。

6. 更改应提交到 FreeBSD-CURRENT 而不是 FreeBSD-STABLE，除非发布工程师特别许可，或除非更改与 FreeBSD-CURRENT 无关。任何重要或非紧急的更改也应在 FreeBSD-CURRENT 中放置至少3天，确保充分的测试。发布工程师对 FreeBSD-STABLE 分支拥有与规则5相同的权威。
   这是一个“不要争论”的问题，因为最终如果更改导致问题，责任由发布工程师承担。请尊重这一点，并在涉及 FreeBSD-STABLE 分支时全力配合发布工程师。FreeBSD-STABLE 的管理可能对外人来说显得过于保守，但要记住保守主义是 FreeBSD-STABLE 的标志，不同规则适用于 FreeBSD-STABLE 分支，不能立即将更改合并到 FreeBSD-STABLE。FreeBSD-CURRENT 开发者需要一定的时间进行测试，因此请允许时间过去，除非该更改对 FreeBSD-STABLE 至关重要、时间敏感或明显到不需要更多测试（例如拼写修正、显而易见的bug或拼写错误修复等）。换句话说，运用常识。

   对于安全分支（例如 `releng/9.3`）的更改，必须经过 `Security Officer Team` 成员或在某些情况下，`Release Engineering Team` 成员的批准。

7. 不要在公开场合与其他提交者争斗；这看起来很不好。
   FreeBSD 项目有公共形象，必须维护好这个形象，这对我们所有人都很重要，特别是在我们吸引新成员时。即便大家都极力控制情绪，也难免会发生情绪失控、言辞激烈的情况。最好的做法是在此类情况发生时尽量减少影响，直到大家冷静下来为止。不要在公开场合发泄愤怒的言辞，也不要将私人通信或其他私人交流转发到公开邮件列表、邮件别名、即时通讯渠道或社交媒体网站上。一个人在私下发送的言辞通常比在公开场合更直白、尖锐，这些内容不适合公开场合——它们只会加剧已经不好的局势。如果发送者至少有尊重，不将其公之于众，那么你也应当同样尊重这一点，保持私密。如果你觉得自己受到其他开发者的不公对待，导致痛苦，请将此事报告给核心小组，而不是公开化。核心小组会尽力充当和平使者，恢复理性。如果争论涉及对代码库的更改，且参与者未能达成和解，核心小组可能会指定一个双方同意的第三方来解决争端。所有相关方必须同意接受该第三方的决定。

8. 尊重所有代码冻结并及时阅读 `committers` 和 `developers` 邮件列表，以便了解代码冻结的生效时间。
   在代码冻结期间提交未经批准的更改是一个非常大的错误，提交者需要保持最新的状态，了解发生的事情，避免在长时间缺席后跳进来并提交积累的 10 兆字节的内容。那些经常滥用此行为的人，将被暂停提交权限，直到他们从我们在格林兰岛举办的 FreeBSD 快乐再教育营返回。

9. 对于任何程序不确定时，先询问！
   许多错误都是因为某人在匆忙中假设自己知道正确的做法。如果你以前没有做过，通常你并不知道我们是如何操作的，真的需要先问一问，否则你会在公开场合完全让自己出丑。问“我到底该怎么做？”没有什么可耻的。我们已经知道你是个聪明人，否则你不会成为提交者。

10. 在提交之前测试你的更改。
    如果你的更改涉及内核，确保你能编译 GENERIC 和 LINT。如果你的更改涉及其他地方，确保你仍然能够进行 `make world`。如果你的更改涉及某个分支，确保在运行该代码的机器上进行测试。如果你的更改可能影响到其他架构，务必在所有受支持的架构上进行测试。请确保你的更改适用于[支持的工具链](https://docs.freebsd.org/en/articles/committers-guide/#compilers)。请参阅[FreeBSD 内部页面](https://www.freebsd.org/internal/)获取可用资源的列表。随着其他架构被添加到 FreeBSD 支持的平台列表中，适当的共享测试资源也将提供。

11. 未经相应维护者 *明确* 批准，不要提交第三方软件。
    第三方软件是指所有位于 **src/contrib**、**src/crypto** 或 **src/sys/contrib** 目录下的软件。

上述目录用于通常从供应商分支导入的第三方软件。将内容提交到这些目录可能会在导入新版本的软件时带来不必要的麻烦。一般来说，考虑将补丁发送到上游供应商。补丁可以在维护者许可下先提交到 FreeBSD。

修改上游软件的原因包括希望对紧密耦合的依赖关系进行严格控制，或是上游代码的可移植性问题。无论原因如何，尽量减少复刻的维护负担对其他维护者是有益的。避免提交琐碎或外观上的更改，因为这些会使之后的合并变得更加困难：这些补丁需要在每次导入时手动重新验证。

如果某个软件没有维护者，建议你承担起责任。如果不确定当前的维护者是谁，可以发送邮件至 [FreeBSD 架构和设计邮件列表](https://lists.freebsd.org/subscription/freebsd-arch)进行询问。

### 20.2. 多架构政策

为了使 FreeBSD 在我们支持的平台上保持良好的可移植性，核心团队制定了以下规定：

> 重大设计工作（包括重大 API 和 ABI 更改）必须先在至少一个 Tier 1 平台上证明其可行性，然后才能提交到源代码树中。

开发者还应了解我们关于硬件架构长期支持的 Tier 政策。这里的规则旨在为开发过程提供指导，并且与该部分中列出的特性和架构要求不同。在发布时，架构上特性支持的 Tier 规则比开发过程中的更改规则更为严格。

### 20.3. 多编译器政策

FreeBSD 基本系统使用 Clang 和 GCC 进行构建。该项目以一种谨慎且可控的方式进行，以最大化这种额外工作的好处，同时将额外工作保持在最低限度。支持 Clang 和 GCC 提高了用户的灵活性。这些编译器有不同的优缺点，支持两者让用户可以根据自己的需求选择最佳的编译器。Clang 和 GCC 支持类似的 C 和 C++ 方言，只需相对较少的条件代码。该项目通过使用两种编译器的特性，增加了代码覆盖率并提高了代码质量。通过支持这一范围，项目能够在更多用户环境中构建并利用更多 CI 环境，从而提高用户的便利性，并为他们提供更多测试工具。通过仔细地将支持的版本范围限制在这些编译器的现代版本，项目避免了过度增加测试矩阵。旧的和冷门的编译器以及旧的语言方言有极其有限的支持，允许用户程序使用它们进行构建，但不要求基本系统必须使用这些编译器进行构建。具体的平衡仍在不断演变，以确保额外工作的好处大于它所带来的负担。该项目曾经支持非常旧的 Intel 编译器或旧版本的 GCC，但我们用精心选择的现代编译器替代了对这些过时编译器的支持。本节记录了我们在哪里使用不同的编译器，以及对此的期望。

FreeBSD 基本系统包含一个内置的 Clang 编译器。由于它在源代码树中，因此这是最受支持的编译器。所有更改在提交之前必须能够用它编译通过。根据更改的性质，应使用此编译器进行完整的测试。

FreeBSD 基本系统还支持多个版本的 Clang 和 GCC 作为树外编译器。对于大规模或风险较高的更改，提交者应使用受支持的 GCC 版本进行测试构建。树外编译器以包的形式提供。GCC 编译器以 `${TARGET_ARCH}-gcc${VERSION}` 包的形式提供，例如 [aarch64-gcc14](https://cgit.freebsd.org/ports/tree/devel/freebsd-gcc14/)。Clang 编译器以 `llvm${VERSION}` 包的形式提供，例如 [llvm18](https://cgit.freebsd.org/ports/tree/devel/llvm18/)。该项目运行自动化 CI 任务，以使用这些编译器构建所有内容。提交者应修复他们的更改破坏的 CI 任务。提交者可以通过将 `CROSS_TOOLCHAIN` 设置为包名来测试用户空间或单个内核的构建，例如 `CROSS_TOOLCHAIN=aarch64-gcc14` 或 `CROSS_TOOLCHAIN=llvm18`。对于 universe 或 tinderbox 构建，`USE_GCC_TOOLCHAINS=gcc${VERSION}` 使用适当的 GCC 编译器包构建所有架构。对于使用树外 Clang 的 universe 或 tinderbox 构建，传递 `CROSS_TOOLCHAIN=llvm${VERSION}`。请注意，虽然基本系统中的所有架构都可以由 Clang 编译，但只有少数架构可以完全由 GCC 构建。

FreeBSD 项目还在 GitHub 上拥有一些 CI 管道。对于 GitHub 上的拉取请求和推送到 GitHub 分支的某些代码，运行了一些交叉编译作业。这些作业测试 FreeBSD 使用比内置编译器落后一个或多个主版本的 Clang 版本进行构建。

FreeBSD 项目还在升级编译器。Clang 和 GCC 都是快速发展的目标。一些更改可能会在编译器到达之前先在源代码树中实施，例如移除旧式的 K&R 函数声明和定义。提交者应尽量注意这一点，并乐于调查新编译器可能带来的代码问题或更改。此外，在新的编译器版本加入源代码树后，若怀疑存在未被检测的回归，可能需要使用旧版本进行编译。

除了编译器，LLVM 的 LLD 和 GNU 的 binutils 也由编译器间接使用。提交者应注意汇编语法和链接器特性的差异，并确保两者的变种都能正常工作。这些组件将作为 FreeBSD CI 任务的一部分进行测试，适用于 Clang 或 GCC。

FreeBSD 项目提供了头文件和库，允许使用其他编译器来构建非基本系统中的软件。这些头文件支持将环境尽可能严格地与标准一致，支持 ANSI-C 先前方言，直到 C89，并处理我们庞大的 Ports 集合中发现的其他边缘情况。这个支持限制了旧标准在头文件等位置的淘汰，但不限制基本系统更新到更新的方言，也不要求基本系统整体上使用这些旧标准进行编译。破坏这一支持将导致 Ports 集合中的软件包构建失败，因此应尽量避免，并在容易修复时尽快修复。

FreeBSD 构建系统目前适应这些不同的环境。随着编译器中新警告的加入，项目尝试修复它们。然而，有时这些警告需要大量重构，因此会通过使用根据编译器版本评估为适当值的 make 变量以某种方式被抑制。开发者应注意这一点，确保任何特定于编译器的标志都能正确地进行条件处理。

#### 20.3.1. 当前编译器版本

对于给定分支（如 `main` 或 `stable/X`），受支持的编译器版本会随时间变化。受支持编译器版本的权威来源是在 GitHub 的交叉构建操作和 Jenkins 中测试的自动化 CI 任务。

| 分支 | 内置编译器 | llvm12 | llvm13 | llvm14 | llvm15 | llvm18 | amd64-gcc12 | amd64-gcc13 | amd64-gcc14 | amd64-gcc15 | riscv64-gcc15 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| main | llvm 19 | | | | Y | Y | Y | Y | Y | Y | Y |
| stable/15 | llvm 19 | | | Y | | Y | Y | Y | Y | | |
| stable/14 | llvm 19 | Y | Y | Y | | | Y | | Y | | |
| stable/13 | llvm 19 | Y | Y | Y | | | Y | | Y | | |

GCC 工具链通过 Jenkins 中的 CI 任务对 amd64 和 riscv64 进行测试。LLVM 工具链通过 GitHub 的交叉构建操作对 aarch64 和 amd64 进行测试。

### 20.4. 其他建议

在提交文档更改时，在提交之前使用拼写检查工具。对于所有 XML 文档，通过运行 `make lint` 和 [textproc/igor](https://cgit.freebsd.org/ports/tree/textproc/igor/) 验证格式化指令是否正确。

对于手册页，运行 [sysutils/manck](https://cgit.freebsd.org/ports/tree/sysutils/manck/) 和 [textproc/igor](https://cgit.freebsd.org/ports/tree/textproc/igor/) 来验证所有交叉引用和文件引用是否正确，并且手册页是否安装了所有适当的 `MLINKS`。

不要将样式修复与新功能混合。样式修复是指不修改代码功能的任何更改。混合这些更改会在询问修订差异时混淆功能性更改，这可能会隐藏任何新引入的 bug。不要在提交 **doc/** 时将空白更改与内容更改混合。差异中的额外杂乱会使翻译人员的工作更加困难。相反，请将任何样式或空白更改放在单独的提交中，并在提交消息中明确标明这些更改。

### 20.5. 弃用功能

当需要从基本系统中删除功能时，请尽可能遵循以下指南：

1. 在手册页和可能的发布说明中提到该选项、工具或接口已弃用。使用弃用的功能会生成警告。
2. 该选项、工具或接口在下一个主要（零点版本）发布之前保留。
3. 该选项、工具或接口被移除且不再文档化。它现在已过时。通常，最好在发布说明中指出其移除。

### 20.6. 隐私与保密

1. 大多数 FreeBSD 的事务是公开的。
   FreeBSD 是一项 *开源* 项目。这意味着不仅任何人都可以使用源代码，而且大多数开发过程都向公众开放，接受公众审查。

2. 某些敏感事项必须保持私密或处于禁运状态。
   不幸的是，无法实现完全透明。作为 FreeBSD 开发者，你将拥有一定程度的特权信息访问权限。因此，你需要遵守某些保密要求。有时保密的需求来自外部合作者或具有特定时间限制的要求。大多数情况下，这是关于不公开私人通信的。

3. 安全官员单独控制安全公告的发布。
   当有影响多个操作系统的安全问题时，FreeBSD 经常依赖提前获取信息，以便为协调发布准备安全公告。除非可以信任 FreeBSD 开发者维护安全性，否则不会提前提供此类信息。安全官员负责控制漏洞信息的预发布访问，并决定所有公告的发布时机。如果有相关知识的开发者能够在保密条件下提供帮助，安全官员可以请求协助准备安全修复。

4. 与核心小组的通信应保密，直到必要时。
   向核心小组的通信最初将被视为保密。然而，核心小组的大多数事务最终会被总结在每月或每季度的核心报告中。但会小心避免公开任何敏感细节。某些特别敏感的议题可能完全不会报告，并且只会保留在核心小组的私人档案中。

5. 对于访问某些商业敏感数据，可能需要签署保密协议。
   访问某些商业敏感数据可能仅在签署保密协议的情况下才可获得。FreeBSD 基金会的法律人员必须在任何具有约束力的协议签署之前进行咨询。

6. 未经许可不得公开私人通信。
   除了上述特定要求外，一般期望未经所有相关方的同意，不得公开开发者之间的私人通信。在将消息转发到公共邮件列表或发布到可能被原始通信者之外的其他人访问的论坛或网站之前，请先征得许可。

7. 项目专用或受限访问渠道的通信必须保持私密。
   类似于个人通信，某些内部通信渠道（包括 FreeBSD 提交者专用邮件列表和受限访问的 IRC 渠道）被视为私人通信。发布这些来源的材料需要获得许可。

8. 核心小组可以批准公开。
   如果由于通信者数量过多或因不合理地拒绝发布而难以获得许可，核心小组可以批准发布这些私密事务，前提是这些事务值得更广泛的公开。

## 21. 支持多种架构

FreeBSD 是一个高度可移植的操作系统，旨在支持多种不同类型的硬件架构。保持机器相关（MD）代码与机器无关（MI）代码的清晰分离，并尽量减少 MD 代码，是我们保持对当前硬件趋势灵活性的战略重要组成部分。每增加一种硬件架构的支持，都会显著增加代码维护、工具链支持和发布工程的成本。它还会大幅提高对内核更改进行有效测试的成本。因此，强烈的动机是区分各种架构支持的不同级别，同时在一些被视为 FreeBSD “目标受众”的关键架构上保持强大的支持。

### 21.1. 一般意图声明

FreeBSD 项目专注于“生产质量的商业现货（COTS）工作站、服务器和高端嵌入式系统”。通过将重点保持在这些环境中感兴趣的有限架构集上，FreeBSD 项目能够维持高水平的质量、稳定性和性能，同时最小化项目中各个支持团队的负担，如 Port 团队、文档团队、安全官员和发布工程团队。硬件支持的多样性通过提供新特性和使用机会，拓宽了 FreeBSD 用户的选择，但这些好处必须始终仔细考虑与额外平台支持相关的实际维护成本。

FreeBSD 项目将平台目标分为四个级别。每个级别包括消费者可以依赖的保证清单，以及项目和开发者为履行这些保证所承担的义务。这些清单定义了每个级别的最低保证。项目和开发者可能提供超出最低保证的额外支持，但这些额外支持不作保证。每个平台目标会根据每个稳定分支分配到一个特定的级别。因此，某个平台目标可能会在并行的稳定分支中分配到不同的级别。

### 21.2. 平台目标

硬件平台的支持由两个组件组成：内核支持和用户空间应用程序二进制接口（ABI）。内核平台支持包括使 FreeBSD 内核能够在硬件平台上运行所需的内容，如机器相关的虚拟内存管理和设备驱动程序。用户空间 ABI 指定了用户进程与 FreeBSD 内核和基本系统库进行交互的接口。用户空间 ABI 包括系统调用接口、公共数据结构的布局和语义、以及传递给子程序的参数的布局和语义。某些 ABI 组件可能由规范定义，如 C++ 异常对象的布局或 C 函数的调用约定。

FreeBSD 内核也使用 ABI（有时称为内核二进制接口（KBI）），包括公共数据结构的语义和布局，以及内核中公共函数的参数的语义和布局。

FreeBSD 内核可以支持多个用户空间 ABI。例如，FreeBSD 的 amd64 内核支持 FreeBSD amd64 和 i386 用户空间 ABI，以及 Linux x86_64 和 i386 用户空间 ABI。FreeBSD 内核应支持一个“本地” ABI 作为默认 ABI。本地 ABI 通常与内核 ABI 共享某些特性，如 C 调用约定、基本类型的大小等。

内核和用户空间 ABI 都有相应的级别定义。在常见的情况下，平台的内核和 FreeBSD ABI 被分配到相同的级别。

#### 21.2.1. 第一层：完全支持的架构

第一层平台是最成熟的 FreeBSD 平台。它们由安全官员、发布工程和 Ports 管理团队支持。第一层架构预计在 FreeBSD 操作系统的所有方面（包括安装和开发环境）都具有生产质量。

FreeBSD 项目为第一层平台的消费者提供以下保证：

- 发布工程团队将提供官方的 FreeBSD 发布镜像。
- 将为受支持的版本提供安全通告和勘误通知的二进制更新和源代码补丁。
- 将为受支持的分支提供安全通告的源代码补丁。
- 跨平台的安全通告的二进制更新和源代码补丁通常会在公告时提供。
- 用户空间 ABI 的变化通常会包含兼容性补丁，以确保在该平台为第一层的任何稳定分支上编译的二进制文件正确运行。这些补丁可能不会在默认安装中启用。如果对 ABI 更改没有提供兼容性补丁，缺少补丁的情况将会在发布说明中清楚地记录。
- 某些内核 ABI 部分的变化将包括兼容性补丁，以确保在该分支上最早支持的版本中编译的内核模块正确运行。请注意，并非所有内核 ABI 部分都受到保护。
- 第三方软件的官方二进制包将由 Port 团队提供。对于嵌入式架构，这些包可能会从不同架构交叉构建。
- 大多数相关的 Port 应该能够构建，或者具有适当的过滤器，防止不适当的 Port 被构建。
- 不特定于平台的新功能将在所有第一层架构上完全功能。
- 用于与较旧稳定分支编译的二进制文件的功能和兼容性补丁可能会在较新的主版本中删除。这些删除将会在发布说明中清楚地记录。
- 第一层平台应该有完整的文档。基本操作将在 FreeBSD 手册中记录。
- 第一层平台将包含在源代码树中。
- 第一层平台应该能够自我托管，无论是通过内建工具链还是外部工具链。如果需要外部工具链，将提供外部工具链的官方二进制包。

为了保持第一层平台的成熟，FreeBSD 项目将维护以下资源来支持开发：

- 在 FreeBSD.org 集群或其他易于所有开发者访问的位置提供构建和测试自动化支持。嵌入式平台可能使用 FreeBSD.org 集群中的仿真器代替实际硬件。
- 包含在 `make universe` 和 `make tinderbox` 目标中。
- 在 FreeBSD 集群中为包构建提供专用硬件（无论是本地构建还是通过 qemu-user）。

为了维持第一层平台的状态，开发者需要提供以下内容：

- 对源代码树的更改不得故意破坏第一层平台的构建。
- 第一层架构必须拥有成熟、健康的用户生态系统和活跃的开发者。
- 开发者应该能够在常见的非嵌入式第一层系统上构建包。这可以意味着，如果平台在该平台上常见，使用本地构建，或者可以意味着在其他第一层架构上进行交叉构建。
- 更改不能破坏用户空间 ABI。如果需要更改 ABI，应通过符号版本控制或共享库版本提升提供对现有二进制文件的 ABI 兼容性。
- 合并到 Stable 分支的更改不能破坏内核 ABI 的受保护部分。如果需要更改内核 ABI，应对更改进行修改，以保持现有内核模块的功能。

#### 21.2.2. 第二层：开发性和小众架构

第二层平台是功能性存在，但成熟度较低的 FreeBSD 平台。它们不受安全官员、发布工程和 Ports 管理团队的支持。

第二层平台可能是仍在积极开发中的第一层平台候选者。达到生命周期终结的架构也可能从第一层状态转为第二层状态，因为持续保持系统在生产质量状态下的资源可用性减少。受到良好支持的小众架构也可以是第二层。

FreeBSD 项目为第二层平台的消费者提供以下保证：

- Port 基础设施应包括对第二层架构的基本支持，足以支持构建 Port 和包。这包括对基本包（如 ports-mgmt/pkg）的支持，但不能保证所有 Port 都能够构建或正常运行。
- 不特定于平台的新功能，如果未实现，应该在所有第二层架构上可行。
- 第二层平台将包含在源代码树中。
- 第二层平台应该能够自我托管，无论是通过内建工具链还是外部工具链。如果需要外部工具链，将提供外部工具链的官方二进制包。
- 第二层平台应提供功能性内核和用户空间，即使未提供官方发布的分发版。

为了保持第二层平台的成熟，FreeBSD 项目将维护以下资源来支持开发：

- 包含在 `make universe` 和 `make tinderbox` 目标中。

为了维持第二层平台的状态，开发者需要提供以下内容：

- 对源代码树的更改不得故意破坏第二层平台的构建。
- 第二层架构必须拥有活跃的用户和开发者生态系统。
- 虽然允许更改破坏用户空间 ABI，但不应无故破坏 ABI。重大用户空间 ABI 更改应限制在主版本中。
- 尚未在第二层架构上实现的新功能，应提供在这些架构上禁用它们的方式。

#### 21.2.3. 第三层：实验性架构

第三层平台至少有部分 FreeBSD 支持。它们不受安全官员、发布工程和 Ports 管理团队的支持。

第三层平台是处于开发初期的架构，面向非主流硬件平台，或者是被认为不太可能在未来广泛使用的遗留系统。第三层平台的初步支持可能存在于一个独立的仓库中，而非主源代码仓库。

FreeBSD 项目不为第三层平台的消费者提供任何保证，也不承诺维护支持开发的资源。第三层平台可能并不总是可构建的，内核或用户空间 ABI 也不被视为稳定的。

#### 21.2.4. 不支持的架构

其他平台在任何形式上都不受项目支持。项目以前将这些平台称为第四层系统。

平台转为不受支持后，所有对该平台的支持将从源代码、Ports 和文档树中移除。请注意，Ports 支持应在平台仍受 Ports 支持的分支中保持。

### 21.3. 改变架构层级的政策

系统只能在获得 FreeBSD 核心团队批准后，才能从一个层级移动到另一个层级。核心团队将与安全官员、发布工程和 Ports 管理团队合作做出这一决定。要将平台晋升到更高的层级，必须在晋升完成之前满足任何缺失的支持保证。

## 22.  Port 相关常见问题

### 22.1. 添加新 Port

#### 22.1.1. 如何添加新 Port ？

将 Port 添加到树中相对简单。Port 准备好添加后，如后文所述[这里](https://docs.freebsd.org/en/articles/committers-guide/#ports-qa-add-new-extra)，你需要在该类别的 **Makefile** 中添加 Port 的目录条目。在该 **Makefile** 中， Port 按字母顺序列出，并添加到 `SUBDIR` 变量中，如下所示：

```
SUBDIR += newport
```

Port 及其类别的 Makefile 准备好后，可以提交新 Port ：

```
% git add category/Makefile category/newport
% git commit
% git push
```

>**提示**
>
>不要忘记按照[此处说明](https://docs.freebsd.org/en/articles/committers-guide/#port-commit-message-formats)为 Ports 树设置 Git 钩子；专门开发了特定的钩子来验证类别的 **Makefile**。

#### 22.1.2. 添加新 Port 时需要注意的其他事项？

检查 Port，最好确保它能够正确编译和打包。

[Porters Handbook](https://docs.freebsd.org/en/books/porters-handbook/testing) 中的测试章节包含更详细的说明。请参阅 [Portclippy / Portfmt](https://docs.freebsd.org/en/books/porters-handbook/testing#testing-portclippy) 和 [poudriere](https://docs.freebsd.org/en/books/porters-handbook/testing#testing-poudriere) 部分。

你不必消除所有警告，但请确保修复简单的警告。

如果 Port 来自于一个之前没有为项目做过贡献的提交者，将该人的名字添加到 [Additional Contributors](https://docs.freebsd.org/en/articles/contributors/#contrib-additional) 部分的 FreeBSD 贡献者列表中。

如果 Port 是通过 PR 提交的，请关闭 PR。关闭 PR 时，状态改为 `Issue Resolved`，并将解决方案设为 `Fixed`。

>**注意**
>
>如果由于某些原因无法使用 [poudriere](https://docs.freebsd.org/en/books/porters-handbook/testing#testing-poudriere) 来测试新 Port，则最基本的测试包括以下步骤：`# make install`
>
>```sh
># make package
># make deinstall
># pkg add 上述构建的包
># make deinstall
># make reinstall
># make package`
>```
>
>请注意，poudriere 是包构建的参考工具，如果无法在 poudriere 中构建 Port，Port 将被移除。

### 22.2. 移除现有 Port

#### 22.2.1. 如何移除现有 Port？

首先，请阅读关于仓库副本的章节。在移除 Port 之前，你需要验证是否没有其他 Port 依赖于它。

- 确保该 Port 在 Ports 中没有依赖：

  - 该 Port 的 PKGNAME 在最近的 INDEX 文件中只出现一次。
  - 没有其他 Port 在其 Makefile 中包含该 Port 的目录或 PKGNAME。
    >**技巧**
    >
    >使用 Git 时，考虑使用 [git-grep(1)](https://man.freebsd.org/cgi/man.cgi?query=git-grep&sektion=1&format=html)，它比 `grep -r` 更快速。

- 然后，移除 Port：

  - 使用 `git rm` 移除该 Port 的文件和目录。
  - 从父目录的 **Makefile** 中移除该 Port 的 `SUBDIR` 列表。
  - 在 **ports/MOVED** 中添加条目。
  - 如果该 Port 在 **ports/LEGAL** 中，移除该 Port。

另外，你也可以使用 **ports/Tools/scripts** 中的 rmport 脚本。此脚本由 Vasil Dimov 编写 [vd@FreeBSD.org](mailto:vd@FreeBSD.org)。如有关于此脚本的问题，请将问题发送到 [FreeBSD ports 邮件列表](https://lists.freebsd.org/subscription/freebsd-ports)，并请抄送当前维护者 Chris Rees [crees@FreeBSD.org](mailto:crees@FreeBSD.org)。

### 22.3. 如何将 Port 移到新位置？

1. 对 Ports 进行彻底检查，确保没有其他 Port 依赖于旧的 Port 位置/名称，并更新它们。仅运行 `grep` 在 **INDEX** 上是不够的，因为一些 Port 的依赖项是通过编译时选项启用的。建议对 Ports 执行完整的 [git-grep(1)](https://man.freebsd.org/cgi/man.cgi?query=git-grep&sektion=1&format=html)。
2. 从旧类别的 Makefile 中移除 `SUBDIR` 条目，并将 `SUBDIR` 条目添加到新类别的 Makefile 中。
3. 在 **ports/MOVED** 中添加条目。
4. 在 **ports/security/vuxml** 中搜索 xml 文件中的条目，并相应地调整它们。特别是，检查以前的包是否有可能包含新 Port 的版本。
5. 使用 `git mv` 移动 Port。
6. 提交更改。

### 22.4. 如何将 Port 复制到新位置？

1. 使用 `cp -R old-cat/old-port new-cat/new-port` 复制 Port。
2. 将新 Port 添加到 **new-cat/Makefile** 中。
3. 修改 **new-cat/new-port** 中的内容。
4. 提交更改。

### 22.5.  Port 冻结

#### 22.5.1. 什么是“ Port 冻结”？

“Port 冻结”是指在发布之前将 Port 树置于受限状态。这是为了确保发布的包具有更高的质量，通常持续几周。在此期间，修复构建问题并构建发布包。这个做法现在不再使用，因为发布包是从当前稳定的季度分支构建的。

有关如何将提交合并到季度分支的更多信息，请参阅 [如何请求合并提交到季度分支的授权？](https://docs.freebsd.org/en/articles/committers-guide/#ports-qa-misc-request-mfh)。

### 22.6. 季度分支

#### 22.6.1. 请求将提交合并到季度分支的程序是什么？

自 2020 年 11 月 30 日起，无需明确请求批准即可提交到季度分支。

#### 22.6.2. 如何将提交合并到季度分支？

将提交合并到季度分支的过程（我们称之为 MFH，历史原因）与将提交合并到 `src` 仓库的 MFC 非常相似，因此基本流程如下：

```sh
% git checkout 2021Q2
% git cherry-pick -x $HASH
(验证一切是否正常，例如通过构建测试)
% git push
```

其中 `$HASH` 是你希望复制到季度分支的提交的哈希值。`-x` 参数确保将 `main` 分支的 `$HASH` 哈希值包含在季度分支的新提交信息中。

### 22.7. 创建新类别

#### 22.7.1. 创建新类别的程序是什么？

请参阅 [Proposing a New Category](https://docs.freebsd.org/en/books/porters-handbook/#proposing-categories) 以了解在 Porter's Handbook 中的详细步骤。完成该程序并将 PR 分配给 Ports 管理团队 [portmgr@FreeBSD.org](mailto:portmgr@FreeBSD.org) 后，是否批准该请求由他们决定。如果他们批准了，责任在于：

1. 执行任何必要的迁移（仅适用于物理类别）。
2. 更新 **ports/Mk/bsd.port.mk** 中的 `VALID_CATEGORIES` 定义。
3. 将 PR 返还给你。

#### 22.7.2. 我需要做什么来实现一个新的物理类别？

1. 升级每个已迁移 Port 的 **Makefile**。此时不要将新类别连接到构建。
   要做到这一点，你需要：

   1. 修改 Port 的 `CATEGORIES`（记住，这是该步骤的目标）。将新类别列在第一位。这样有助于确保 `PKGORIGIN` 正确。
   2. 运行 `make describe`。由于接下来你将在几步中运行的顶层 `make index` 是对整个 Port 层次结构执行 `make describe`，在此步骤中捕获错误可以避免之后再次运行该步骤。
   3. 如果你想更彻底，可以在此时运行 [portlint(1)](https://man.freebsd.org/cgi/man.cgi?query=portlint&sektion=1&format=html)。
2. 检查 `PKGORIGIN` 是否正确。 Port 系统使用每个 Port 的 `CATEGORIES` 条目来创建其 `PKGORIGIN`，该值用于将已安装的包连接到它们构建时使用的 Port 目录。如果此条目错误，常用的 Port 工具，如 [pkg-version(8)](https://man.freebsd.org/cgi/man.cgi?query=pkg-version&sektion=8&format=html) 和 [portupgrade(1)](https://man.freebsd.org/cgi/man.cgi?query=portupgrade&sektion=1&format=html)，将无法正常工作。
   要检查这一点，请使用 **chkorigin.sh** 工具：`env PORTSDIR=/path/to/ports sh -e /path/to/ports/Tools/scripts/chkorigin.sh`。这将检查 Port 树中的每个 Port ，甚至是那些尚未连接到构建的 Port ，因此你可以在迁移操作后直接运行它。提示：别忘了检查你刚刚迁移的 Port 的从属 Port 的 `PKGORIGIN`！
3. 在本地系统上测试所提议的更改：首先，注释掉旧 Port 类别 **Makefile** 中的 SUBDIR 条目；然后在 **ports/Makefile** 中启用新类别的构建。运行 `make checksubdirs` 来检查 SUBDIR 条目。接下来，在 **ports/** 目录中运行 `make index`。即使在现代系统上，这个过程也可能需要超过 40 分钟；但是，这是一个必要的步骤，以防止其他人遇到问题。
4. 完成上述步骤后，你可以提交更新后的 **ports/Makefile**，以将新类别连接到构建中，并提交旧类别的 **Makefile** 更改。
5. 在 **ports/MOVED** 中添加适当的条目。
6. 通过修改以下内容更新文档：

   - 在 Porter’s Handbook 中更新 [类别列表](https://docs.freebsd.org/en/books/porters-handbook/#PORTING-CATEGORIES)。
7. 只有在完成上述所有步骤并且没有人再报告新 Port 的问题时，才能从其原来的位置删除旧 Port 。

#### 22.7.3. 我需要做什么来实现一个新的虚拟类别？

这比物理类别简单得多，仅需进行一些修改：

- 在 Porter’s Handbook 中更新 [类别列表](https://docs.freebsd.org/en/books/porters-handbook/#PORTING-CATEGORIES)。

### 22.8. 杂项问题

#### 22.8.1. 是否可以在没有维护者批准的情况下提交更改？

对于大多数 Port ，以下类型的修复可以在没有维护者批准的情况下提交：

- Port 的基础设施更改（即现代化，但不改变功能）。例如，涵盖了转换为新的 `USES` 宏、启用详细构建以及切换到新的 Port 系统语法。
- 微小且已 *测试* 的构建和运行时修复。
- Port 的文档或元数据更改，如 **pkg-descr** 或 `COMMENT`。

以下情况除外：由 Ports 管理团队 <[portmgr@FreeBSD.org](mailto:portmgr@FreeBSD.org)> 或安全官员团队 <[security-officer@FreeBSD.org](mailto:security-officer@FreeBSD.org)> 维护的任何内容。永远不允许未经授权的提交对这些团队管理的 Port 进行更改。

#### 22.8.2. 我如何知道我的 Port 是否构建正确？

每周会多次构建包。如果 Port 失败，维护者将收到来自 `pkg-fallout@FreeBSD.org` 的电子邮件。

所有包构建的报告（官方的、实验性的和非回归的）都汇总在 [pkg-status.FreeBSD.org](https://pkg-status.FreeBSD.org/)。

#### 22.8.3. 我添加了一个新 Port 。是否需要将其添加到 **INDEX** 文件中？

不需要。可以通过运行 `make index` 来生成该文件，或者通过 `make fetchindex` 下载一个预生成的版本。

#### 22.8.4. 是否还有其他我不能触碰的文件？

任何直接位于 **ports/** 下的文件，或者位于以大写字母开头的子目录中的文件（如 **Mk/**、**Tools/** 等）。特别是， Ports 管理团队 [portmgr@FreeBSD.org](mailto:portmgr@FreeBSD.org) 非常在意 **ports/Mk/bsd.port*.mk** 文件，因此除非你愿意面对他们的怒火，否则不要对这些文件进行提交更改。

#### 22.8.5. 当分发文件更改但版本没有改变时，更新 Port 的校验和的正确程序是什么？

当由于作者更新文件而不更改 Port 修订时，更新分发文件的校验和，提交信息中应包含原始文件和新文件之间相关差异的摘要，以确保分发文件没有被损坏或恶意更改。如果当前版本的 Port 已经在 Port 树中存在较长时间，通常可以在 ftp 服务器上找到旧的分发文件；否则，应联系作者或维护者了解为何分发文件发生更改。

#### 22.8.6. 如何请求 Port 树的实验性测试构建（exp-run）？

在提交对 Port 树有重大影响的补丁之前，必须完成 exp-run。补丁可以针对 Port 树或基本系统。

使用提交者提供的补丁将进行完整的包构建，提交者需要在提交前修复检测到的问题（*fallout*）。

1. 访问 [Bugzilla 新 PR 页面](https://bugs.freebsd.org/submit)。
2. 选择补丁涉及的产品。
3. 像往常一样填写缺陷报告，记得附上补丁。
4. 如果顶部显示“Show Advanced Fields”，点击它。它现在会显示为“Hide Advanced Fields”。此时将有更多的字段可用。如果已经显示为“Hide Advanced Fields”，则无需进行任何操作。
5. 在“Flags”部分，将“exp-run”设置为 `?`。对于所有其他字段，将鼠标悬停在任何字段上可以查看更多详细信息。
6. 提交。等待构建运行。
7. Ports 管理团队 [portmgr@FreeBSD.org](mailto:portmgr@FreeBSD.org) 会回复可能的 fallout。
8. 根据 fallout：

   - 如果没有 fallout，程序到此为止，变更可以提交，前提是已获得其他所需的批准。

     1. 如果有 fallout，*必须* 修复它，你可以直接修复 Port 树中的 Port ，或将其添加到提交的补丁中。
     2. 完成修复后，返回第 6 步并说明已修复 fallout，然后等待 exp-run 再次运行。只要有破损的 Port ，就重复此过程。

## 23. 非提交者开发者的特定问题

一些有权访问 FreeBSD 机器的人没有提交权限。几乎本文件的所有内容同样适用于这些开发者（除非是特定于提交的内容以及与提交相关的邮件列表成员资格）。特别是，我们建议你阅读：

- [行政详情](https://docs.freebsd.org/en/articles/committers-guide/#admin)

- [适用于所有人](https://docs.freebsd.org/en/articles/committers-guide/#conventions-everyone)
  >**注意**
  >
  >请确保你的导师将你添加到“附加贡献者”（**doc/shared/contrib-additional.adoc**）中——如果尚未列出你。

- [开发者关系](https://docs.freebsd.org/en/articles/committers-guide/#developer.relations)

- [SSH 快速入门指南](https://docs.freebsd.org/en/articles/committers-guide/#ssh.guide)

- [FreeBSD 提交者规则大全](https://docs.freebsd.org/en/articles/committers-guide/#rules)

## 24. 关于分析的信息

自 2022 年起，FreeBSD 项目网站使用 [Plausible Analytics](https://plausible.io/) 收集关于网站使用的匿名统计数据。

此前，Google 分析于 2012 年 12 月 12 日至 2022 年 3 月 3 日期间启用。

## 25. 杂项问题

### 25.1. 如何访问 people.FreeBSD.org 来发布个人或项目信息？

`people.FreeBSD.org` 就是 `freefall.FreeBSD.org`。只需创建一个 **public_html** 目录。你放入该目录中的任何内容将自动显示在 [https://people.FreeBSD.org/](https://people.freebsd.org/) 下。

### 25.2. 邮件列表归档存储在哪里？

邮件列表归档存储在 `freefall.FreeBSD.org` 上的 **/local/mail** 目录下。

### 25.3. 我想指导一位新提交者。我需要遵循什么流程？

请参阅内部页面上的 [新帐户创建流程](https://www.freebsd.org/internal/new-account/) 文档。

## 26. FreeBSD 提交者的福利和特权

### 26.1. 认可

被认可为优秀的软件工程师，这是最持久的价值。此外，有机会与每位工程师梦寐以求想见到的最优秀的人合作，亦是巨大的福利！

### 26.2. `Gandi.net`

[Gandi](https://gandi.net/) 提供网站托管、云计算、域名注册和 X.509 证书服务。

Gandi 向所有 FreeBSD 开发者提供 E-rate 折扣。为了简化获取折扣的过程，首先设置 Gandi 账户，填写账单信息并选择货币。然后，使用你的 `@freebsd.org` 邮件地址发送邮件至 [non-profit@gandi.net](mailto:non-profit@gandi.net)，并注明你的 Gandi 账户。

### 26.3. `rsync.net`

[rsync.net](https://rsync.net/) 提供针对 UNIX 用户优化的云存储服务，适用于外部备份。其服务完全在 FreeBSD 和 ZFS 上运行。

rsync.net 向 FreeBSD 开发者提供免费的 500 GB 永久账户。只需使用你的 `@freebsd.org` 邮件地址在 [https://www.rsync.net/freebsd.html](https://www.rsync.net/freebsd.html) 上注册，即可获得此免费账户。
