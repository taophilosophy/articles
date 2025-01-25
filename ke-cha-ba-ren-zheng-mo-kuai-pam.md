# 可插拔认证模块（PAM）

2001-2003 年版权归网络关联技术公司所有。

<details open="" data-immersive-translate-walked="d750a8fa-aa9a-4a71-b060-b79bfe927d2b"><summary data-immersive-translate-walked="d750a8fa-aa9a-4a71-b060-b79bfe927d2b" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

本文由 ThinkSec AS 和 Network Associates Laboratories（Network Associates, Inc.的安全研究部门，根据 DARPA/SPAWAR 合同 N66001-01-C-8035（“CBOSS”））编写，作为 DARPA CHATS 研究计划的一部分，专为 FreeBSD 项目编写。

Linux 是 Linus Torvalds 的注册商标。

Motif、OSF/1 和 UNIX 是 The Open Group 在美国和其他国家的注册商标，IT DialTone 和 The Open Group 是 The Open Group 的商标。

太阳、太阳微系统、Java、Java 虚拟机、JDK、JRE、JSP、JVM、Netra、OpenJDK、Solaris、StarOffice、SunOS 和 VirtualBox 是 Sun Microsystems, Inc. 在美国和其他国家的商标或注册商标。

制造商和销售商用来区分其产品的许多名称被视为商标。 如果这些名称出现在本文档中，并且 FreeBSD 项目意识到商标声明，这些名称后面将跟有“™”或“®”符号。

</details>

摘要

本文描述了可插拔认证模块（PAM）库的基本原理和机制，并解释了如何配置 PAM，如何将 PAM 集成到应用程序中，以及如何编写 PAM 模块。

---

## 1. 引言

可插拔认证模块（PAM）库是一个通用的用于认证相关服务的 API，允许系统管理员通过安装新的 PAM 模块来简单地添加新的认证方法，并通过编辑配置文件来修改认证策略。

PAM 由 Sun Microsystems 的 Vipin Samar 和 Charlie Lai 于 1995 年定义和开发，自那以后变化不大。1997 年，开放组织发布了 X/Open 单点登录 (XSSO) 初步规范，该规范标准化了 PAM API，并添加了单一（或集成）登录的扩展。在撰写本文时，该规范尚未被采纳为标准。

尽管本文主要关注使用 OpenPAM 的 FreeBSD 5.x，但它同样适用于使用 Linux-PAM 的 FreeBSD 4.x 以及其他操作系统如 Linux 和 Solaris™。

## 2. 术语和约定

### 2.1. 定义

关于 PAM 的术语是相当混乱的。Samar 和 Lai 的原始论文以及 XSSO 规范都没有试图正式定义涉及 PAM 的各种参与者和实体的术语，而他们使用的术语（但未定义）有时会误导和模糊。建立一致和明确的术语的第一次尝试是由 Andrew G. Morgan（Linux-PAM 的作者）于 1999 年撰写的白皮书。尽管 Morgan 的术语选择是一大进步，但在本作者看来并不完美。接下来的内容是一种尝试，受 Morgan 启发，为所有涉及 PAM 的参与者和实体定义精确和明确的术语。

账户申请人请求仲裁人员的一组凭证。

请求认证的用户或实体。

拥有必要特权以验证申请者凭据并具有授权批准或拒绝请求的用户或实体。

响应 PAM 请求时将调用的模块序列。该链包括关于调用模块顺序、传递给它们的参数以及如何解释结果的信息。

客户端负责代表申请人发起认证请求，并从他那里获取必要的认证信息的应用程序。

PAM 提供的四个基本功能组之一：认证、账户管理、会话管理和认证令牌更新之一。

模块一个或多个相关功能的集合，实现特定认证功能，并集成为单一（通常是动态加载的）二进制文件，由单一名称标识。

策略描述特定服务的 PAM 请求处理配置语句的完整集合。一个策略通常包括四个链，每个链对应一个设施，尽管有些服务不使用所有四个设施。

服务器应用程序代表仲裁者与客户端交流，检索认证信息，验证申请者的凭据并批准或拒绝请求。

服务提供类似或相关功能并需要类似认证的服务器类别。PAM 策略基于每个服务定义，因此所有声称相同服务名称的服务器将受到相同策略的约束。

服务被服务器提供给申请人的上下文。PAM 的四个设施之一，会话管理，专门负责建立和撤销此上下文。

与帐户相关的一块信息，例如密码或口令，申请人必须提供以证明其身份。

同一申请人向同一服务器实例发出的一系列请求，从认证和会话建立开始，到会话拆除结束。

### 2.2. 用法示例

本节旨在通过一些简单的示例来阐释上述定义的一些术语的含义。

#### 2.2.1. 客户端和服务器是一体的

这个简单的例子展示了 alice 切换到 root 。

```sh
% whoami
alice

% ls -l `which su`
-r-sr-xr-x  1 root  wheel  10744 Dec  6 19:06 /usr/bin/su

% su -
Password: xi3kiune
# whoami
root
```

- 申请人是 alice 。
- 这个账户是 root 。
- su(1)进程既是客户端又是服务器。
- 验证令牌是 xi3kiune 。
- 仲裁者是 root ，这就是 su(1)被设置为 setuid root 的原因。

#### 2.2.2. 客户端和服务器是分开的

下面的示例显示 eve 尝试启动对 login.example.com 的 ssh(1) 连接，请求以 bob 登录，并成功。Bob 应该选择一个更好的密码！

```sh
% whoami
eve

% ssh bob@login.example.com
bob@login.example.com's password:
% god
Last login: Thu Oct 11 09:52:57 2001 from 192.168.0.1
Copyright (c) 1980, 1983, 1986, 1988, 1990, 1991, 1993, 1994
	The Regents of the University of California.  All rights reserved.
FreeBSD 4.4-STABLE (LOGIN) 4: Tue Nov 27 18:10:34 PST 2001

Welcome to FreeBSD!
%
```

- 申请人是 eve 。
- 客户端是 Eve 的 ssh(1)进程。
- 服务器是 login.example.com 上的 sshd(8) 进程。
- 账户是 bob 。
- 认证令牌为 god 。
- 尽管在此示例中未显示，仲裁员是 root 。

#### 2.2.3. 示例政策

下面是 FreeBSD 的默认策略，适用于 sshd ：

```sh
sshd	auth		required	pam_nologin.so	no_warn
sshd	auth		required	pam_unix.so	no_warn try_first_pass
sshd	account		required	pam_login_access.so
sshd	account		required	pam_unix.so
sshd	session		required	pam_lastlog.so	no_fail
sshd	password	required	pam_permit.so
```

- 该策略适用于 sshd 服务（不一定局限于 sshd(8)  服务器）。
- auth ， account ， session 和 password 是设施。
- pam_nologin.so，pam_unix.so，pam_login_access.so，pam_lastlog.so 和 pam_permit.so 都是模块。从这个例子可以看出，pam_unix.so 至少提供两个功能（认证和账户管理）。

## 3. PAM Essentials

### 3.1. Facilities and Primitives

PAM API 提供了四个设施中的六种不同的认证基元，下面对它们进行描述。

auth 认证。该设施关注的是对申请人进行认证并建立帐户凭据。它提供了两种基元：

- pam_authenticate(3) 通过请求认证令牌并将其与存储在数据库中的值或从认证服务器获取的值进行比较，对申请人进行认证。
- pam_setcred(3) 建立帐户凭据，如用户 ID、组成员资格和资源限制。

account 帐户管理。此功能处理与帐户可用性无关的问题，例如基于一天中的时间或服务器工作负载的访问限制。它提供单个基元：

- pam_acct_mgmt(3) 验证所请求的帐户是否可用。

session 会话管理。此功能处理与会话建立和拆除相关的任务，例如登录记账。它提供两个原语：

- pam_open_session(3) 执行与会话建立相关的任务：在 utmp 和 wtmp 数据库中添加条目，启动 SSH 代理等。
- pam_close_session(3) 执行与会话拆除相关的任务：在 utmp 和 wtmp 数据库中添加条目，停止 SSH 代理等。

password 密码管理。此功能用于更改与帐户关联的身份验证令牌，无论是因为其已过期还是因为用户希望更改它。它提供一个单一的原语：

- pam_chauthtok(3) 更改身份验证令牌，可选地验证其是否足够难以猜测，以前是否已使用过等。

### 3.2. 模块

模块是 PAM 中一个非常核心的概念；毕竟，它们是 "PAM" 中的 "M"。PAM 模块是一个自包含的程序代码片段，它在一个或多个设施中实现原语，用于特定的机制；例如，身份验证设施的可能机制包括 UNIX® 密码数据库、NIS、LDAP 和 Radius。

#### 3.2.1. 模块命名

FreeBSD 在单个模块中实现每个机制，命名为 pam_mechanism.so （例如，UNIX® 机制的命名为 pam_unix.so ）。其他实现有时会为不同的设施分别创建模块，并在模块名称中包含设施名称和机制名称。例如，Solaris™ 有一个 pam_dial_auth.so.1 模块，通常用于验证拨号用户。

#### 3.2.2. 模块版本控制

FreeBSD 的原始 PAM 实现基于 Linux-PAM，并未使用 PAM 模块的版本号。这通常会导致与旧版本系统库链接的传统应用程序出现问题，因为无法加载所需模块的匹配版本。

另一方面，OpenPAM 会寻找与 PAM 库（当前为 2）具有相同版本号的模块，仅在无法加载版本化模块时才会退而使用无版本化模块。因此，可以为传统应用程序提供传统模块，同时允许新的（或新构建的）应用程序利用最新模块。

虽然 Solaris™ PAM 模块通常具有版本号，但它们并非真正版本化，因为该数字是模块名称的一部分，必须包含在配置中。

### 3.3. 链和策略

当服务器启动 PAM 事务时，PAM 库会尝试加载 pam_start(3)调用中指定的服务的策略。策略指定了如何处理认证请求，并在配置文件中定义。这是 PAM 的另一个核心概念：管理员可以通过简单编辑文本文件来调整系统安全策略（在更广义上的词义上）。

策略包括四个链，每个链对应四个 PAM 设施中的一个。每个链是一系列配置语句，每个语句指定要调用的模块，一些（可选的）要传递给模块的参数，以及描述如何解释来自模块的返回码的控制标志。

理解控制标志对于理解 PAM 配置文件至关重要。有四种不同的控制标志：

binding 如果模块成功且链中没有较早失败的模块，链将立即终止且请求被授予。如果模块失败，链的其余部分将被执行，但最终将拒绝请求。

这个控制标志是由 Sun 在 Solaris™ 9（SunOS™ 5.9）中引入的，也受 OpenPAM 支持。

required 如果模块执行成功，则执行链条的其余部分，并且除非其他模块失败，否则授予请求。如果模块失败，则还将执行链条的其余部分，但最终将拒绝请求。

requisite 如果模块执行成功，则执行链条的其余部分，并且除非其他模块失败，否则授予请求。如果模块失败，则立即终止链条并拒绝请求。

如果模块成功且链中之前的模块均未失败，链将立即终止并授予请求。如果模块失败，则忽略该模块并执行链的其余部分。

由于此标志的语义可能有些混乱，特别是当它用于链中的最后一个模块时，建议如果实现支持，则改用 binding 控制标志。

optional 执行该模块，但忽略其结果。如果链中的所有模块均标记为 optional ，则所有请求将始终被授予。

当服务器调用六个 PAM 原语中的一个时，PAM 检索所属基础设施的链，并按列表中的顺序调用链中的每个模块，直到到达末尾，或确定不再需要进一步处理（要么因为 binding 或 sufficient 模块成功，要么因为 requisite 模块失败）。只有在至少调用了一个模块且所有非可选模块均成功的情况下，才授予请求。

请注意，同一个模块在同一个链中多次列出是可能的，尽管这种情况并不常见。例如，一个在目录服务器中查找用户名称和密码的模块可以使用不同的参数多次调用，指定要联系的不同目录服务器。PAM 将同一链中的同一模块的不同出现视为不同且无关的模块。

### 3.4. 事务

典型 PAM 事务的生命周期如下所述。请注意，如果其中任何步骤失败，服务器应向客户端报告合适的错误消息并中止事务。

1. 如有必要，服务器通过与 PAM 独立的机制获取仲裁者凭据-最常见的情况是由 root 启动，或者是通过设置为 setuid root 。
2. 服务器调用 pam_start(3) 来初始化 PAM 库并指定其服务名称和目标账户，并注册一个合适的会话函数。
3. 服务器获取与事务相关的各种信息（如申请人的用户名和客户端运行的主机名），并使用 pam_set_item(3)将其提交给 PAM。
4. 服务器调用 pam_authenticate(3)来对申请人进行身份验证。
5. 服务器调用 pam_acct_mgmt(3)来验证请求的账户是否可用和有效。如果密码正确但已过期，pam_acct_mgmt(3)将返回 PAM_NEW_AUTHTOK_REQD 而不是 PAM_SUCCESS 。
6. 如果前一步返回 PAM_NEW_AUTHTOK_REQD ，服务器现在调用 pam_chauthtok(3)强制客户端为请求的账户更改认证令牌。
7. 现在申请人已经得到适当的认证，服务器调用 pam_setcred(3)来建立请求账户的凭证。它能够这样做是因为它代表仲裁者行事，并持有仲裁者的凭证。
8. 一旦正确的凭证被建立，服务器调用 pam_open_session(3)来设置会话。
9. 服务器现在执行客户请求的任何服务-例如，向申请人提供 shell。
10. 一旦服务器完成为客户提供服务，它调用 pam_close_session(3)来终止会话。
11. 最后，服务器调用 pam_end(3)通知 PAM 库它已完成，并释放在事务过程中分配的任何资源。

## 4. PAM 配置

### 4.1. PAM 策略文件

#### 4.1.1. /etc/pam.conf

传统的 PAM 策略文件是/etc/pam.conf。该文件包含了系统的所有 PAM 策略。文件的每一行描述了链中的一步，如下所示：

```sh
login   auth    required        pam_nologin.so  no_warn
```

字段依次为：服务名称，设施名称，控制标志，模块名称和模块参数。任何额外的字段都被解释为额外的模块参数。

为每个服务/设施对构建一个单独的链，因此，同一服务和设施的行出现的顺序很重要，但列出单个服务和设施的顺序则无关紧要。原始 PAM 论文中的示例按设施分组配置行，Solaris™ stock pam.conf 仍然如此，但 FreeBSD 的 stock 配置按服务分组配置行。无论哪种方式都可以；任何方式都是合理的。

#### 4.1.2. /etc/pam.d 目录

OpenPAM 和 Linux-PAM 支持另一种配置机制，在 FreeBSD 中是首选机制。在这种方案中，每个策略都包含在一个单独的文件中，其名称为其适用的服务名称。这些文件存储在/etc/pam.d/中。

这些针对每个服务的策略文件只有四个字段，而不是 pam.conf 的五个字段：服务名称字段被省略。因此，而不是前一节中的示例 pam.conf 行，用户将在/etc/pam.d/login 中有以下行：

```sh
auth    required        pam_nologin.so  no_warn
```

由于这种简化的语法，可以通过将每个服务名称链接到同一个策略文件来为多个服务使用相同的策略。例如，要为 su 和 sudo 服务使用相同的策略，可以按照以下步骤进行：

```sh
# cd /etc/pam.d
# ln -s su sudo
```

这是因为服务名称是根据文件名确定的，而不是在策略文件中指定的，因此同一个文件可以用于多个不同名称的服务。

由于每个服务的策略存储在单独的文件中，pam.d 机制还可以非常方便地为第三方软件包安装附加策略。

#### 4.1.3. 策略搜索顺序

正如我们前面所看到的，PAM 政策可以在许多地方找到。如果同一服务的政策存在于多个地方会发生什么？

很重要的是要理解 PAM 的配置系统是以链为中心的。

### 4.2. 配置行的分解

如《PAM 策略文件》中所述，/etc/pam.conf 中的每一行都包括四个或更多字段：服务名称、设施名称、控制标志、模块名称以及零个或多个模块参数。

服务名称通常（虽然不总是）是语句适用的应用程序名称。如果不确定，请参考各个应用程序的文档以确定它使用的服务名称。

如果您使用 /etc/pam.d/ 而不是 /etc/pam.conf，则服务名称由策略文件的名称指定，并且从实际配置行中省略，然后以设施名称开头。

设施是《设施与原语》中描述的四个设施关键字之一。

同样，控制标志是《链和策略》中描述的四个关键字之一，用于描述如何解释模块返回码。Linux-PAM 支持一种替代语法，允许您指定与每个可能的返回码关联的动作，但应避免使用，因为它是非标准的，并与 Linux-PAM 调度服务调用的方式密切相关（这与 Solaris™ 和 OpenPAM 的方式有很大不同）。毫不奇怪，OpenPAM 不支持此语法。

### 4.3. 政策

要正确配置 PAM，了解政策的解释方式至关重要。

当应用程序调用 pam_start(3)时，PAM 库会加载指定服务的政策，并构建四个模块链（每个设施一个）。如果这些链中的一个或多个为空，则使用 other 服务的政策中的相应链进行替换。

当应用程序后来调用六个 PAM 原语中的一个时，PAM 库检索相应设施的链，并按照配置中列出的顺序调用链中的每个模块中的适当服务函数。在每次调用服务函数之后，使用模块类型和服务函数返回的错误代码来确定接下来会发生什么。除了我们在下面讨论的几个例外情况外，以下表格适用：

表 1. PAM 链执行摘要

|            | PAM_SUCCESS       | PAM_IGNORE | other               |
| ---------- | ----------------- | ---------- | ------------------- |
| binding    | if (!fail) break; | -          | fail = true;        |
| required   | -                 | -          | fail = true;        |
| requisite  | -                 | -          | fail = true; break; |
| sufficient | if (!fail) break; | -          | -                   |
| optional   | -                 | -          | -                   |

如果 fail 在链的末尾为真，或者达到"break"时，调度程序将返回第一个失败模块返回的错误代码。否则，它将返回 PAM_SUCCESS 。

首个需要注意的异常是，错误代码 PAM_NEW_AUTHTOK_REQD 被视为成功，但如果没有模块失败，并且至少有一个模块返回了 PAM_NEW_AUTHTOK_REQD ，调度程序将返回 PAM_NEW_AUTHTOK_REQD 。

第二个异常是 pam_setcred(3)将 binding 和 sufficient 模块视为 required 。

第三个也是最后一个异常是 pam_chauthtok(3)两次运行整个链条（一次用于初步检查，一次用于实际设置密码），在预检阶段它将 binding 和 sufficient 模块视为 required 。

## 5. FreeBSD PAM 模块

### 5.1. pam_deny(8) 是最简单的模块之一，它会对任何请求做出{{0}}的响应。它非常适合快速禁用一个服务（将其添加到每个链的顶部），或者用于终止{{1}}模块的链。

pam_deny(8) 模块是可用模块之一；它对任何请求均有 PAM_AUTH_ERR 的响应。它适用于快速禁用服务（将其添加到每个链的顶部），或用于终止 sufficient 模块的链。

### 5.2. [pam_echo(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_echo&sektion=8&format=html)

pam_echo(8) 模块只是将其参数传递给对话函数作为 PAM_TEXT_INFO 消息。 它主要用于调试，但也可以在启动身份验证过程之前显示诸如“未经授权的访问将被起诉”之类的消息。

### 5.3。 pam_exec(8) 

pam_exec(8) 模块将其第一个参数视为要执行的程序的名称，并将其余参数作为命令行参数传递给该程序。 一个可能的应用是在登录时使用它来运行一个程序，该程序可以挂载用户的主目录。

### 5.4. [pam_ftpusers(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_ftpusers&sektion=8&format=html)

pam_ftpusers(8) 模块

### 5.5. [pam_group(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_group&sektion=8&format=html)

pam_group(8) 模块根据用户是否属于特定文件组（通常为 wheel su(1)）接受或拒绝申请者。它主要用于保持 BSD su(1)的传统行为，但也有许多其他用途，比如排除某些用户组不让其访问特定服务。

### 5.6. [pam_guest(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_guest&sektion=8&format=html)

pam_guest(8) 模块允许使用固定登录名进行访客登录。可以对密码设置各种要求，但默认行为是只要登录名是访客帐户的就允许任何密码。pam_guest(8) 模块可以轻松用于实现匿名 FTP 登录。

### 5.7. [pam_krb5(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_krb5&sektion=8&format=html)

pam_krb5(8) 模块

### 5.8. [pam_ksu(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_ksu&sektion=8&format=html)

pam_ksu(8) 模块

### 5.9. [pam_lastlog(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_lastlog&sektion=8&format=html)

pam_lastlog(8) 模块

### 5.10. [pam_login_access(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_login_access&sektion=8&format=html)

pam_login_access(8) 模块提供了一个实现账户管理原语的方法，该方法强制执行 login.access(5)表中指定的登录限制。

### 5.11. [pam_nologin(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_nologin&sektion=8&format=html)

当 /var/run/nologin 存在时，pam_nologin(8)  模块拒绝非 root 用户登录。这个文件通常由 shutdown(8)  在距离计划关机时间少于五分钟时创建。

### 5.12. [pam_passwdqc(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_passwdqc&sektion=8&format=html)

pam_passwdqc(8)  模块

### 5.13. [pam_permit(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_permit&sektion=8&format=html)

pam_permit(8) 模块是可用的最简单模块之一；它对任何请求都响应 PAM_SUCCESS 。它作为一个占位符很有用，用于一个或多个链本来会是空的服务。

### 5.14. [pam_radius(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_radius&sektion=8&format=html)

pam_radius(8) 模块

### 5.15. [pam_rhosts(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_rhosts&sektion=8&format=html)

pam_rhosts(8) 模块

### 5.16. [pam_rootok(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_rootok&sektion=8&format=html)

pam_rootok(8) 模块仅在调用它的进程的真实用户 ID 为 0 时报告成功。这对于像 su(1)或 passwd(1)这样的非网络服务很有用，这些服务应自动允许 root 访问。

### 5.17. [pam_securetty(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_securetty&sektion=8&format=html)

pam_securetty(8) 模块

### 5.18. [pam_self(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_self&sektion=8&format=html)

pam_self(8) 模块仅在申请者的名称与目标帐户的名称完全匹配时报告成功。它在非网络服务（如 su(1)）中非常有用，可以轻松验证申请者的身份。

### 5.19. pam_ssh(8) 模块

pam_ssh(8) 模块提供身份验证和会话服务。身份验证服务允许具有在其~/.ssh 目录中有密码保护的 SSH 密钥的用户通过输入密码短语来进行身份验证。会话服务启动 ssh-agent(1)并预先加载在身份验证阶段解密的密钥。此功能特别适用于本地登录，无论是在 X 环境（使用 xdm(8) 或其他 PAM-aware X 登录管理器）还是在控制台。

### 5.20. pam_tacplus(8) 模块

pam_tacplus(8)  模块

### 5.21. [pam_unix(8) ](https://man.freebsd.org/cgi/man.cgi?query=pam_unix&sektion=8&format=html)

pam_unix(8)  模块实现了传统的 UNIX® 密码认证，使用 getpwnam(3) 获取目标账户的密码并与申请人提供的密码进行比较。它还提供账户管理服务（强制执行账户和密码过期时间）和密码更改服务。这可能是单一最有用的模块，因为绝大多数管理员会希望至少对某些服务保持历史行为。

## 6. PAM 应用程序编程

本节尚未撰写。

## 7. PAM 模块编程

本节尚未编写。

## 附录 A：示例 PAM 应用程序

以下是使用 PAM 的 su(1)的最小实现。请注意，它使用 OpenPAM 特定的 openpam_ttyconv(3)对话函数，在 security/openpam.h 中有原型。如果您希望在具有不同 PAM 库的系统上构建此应用程序，则必须提供自己的对话函数。一个强大的对话功能令人惊讶地难以实现；示例 PAM 对话功能中提供的对话功能是一个很好的起点，但不应用于实际应用程序中。

```c
/*-
 * Copyright (c) 2002,2003 Networks Associates Technology, Inc.
 * All rights reserved.
 *
 * This software was developed for the FreeBSD Project by ThinkSec AS and
 * Network Associates Laboratories, the Security Research Division of
 * Network Associates, Inc.  under DARPA/SPAWAR contract N66001-01-C-8035
 * ("CBOSS"), as part of the DARPA CHATS research program.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 * 3. The name of the author may not be used to endorse or promote
 *    products derived from this software without specific prior written
 *    permission.
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
 * $P4: //depot/projects/openpam/bin/su/su.c#10 $
 * $FreeBSD: head/en_US.ISO8859-1/articles/pam/su.c 38826 2012-05-17 19:12:14Z hrs $
 */

#include <sys/param.h>
#include <sys/wait.h>

#include <err.h>
#include <pwd.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <syslog.h>
#include <unistd.h>

#include <security/pam_appl.h>
#include <security/openpam.h>	/* for openpam_ttyconv() */

extern char **environ;

static pam_handle_t *pamh;
static struct pam_conv pamc;

static void
usage(void)
{

	fprintf(stderr, "Usage: su [login [args]]\n");
	exit(1);
}

int
main(int argc, char *argv[])
{
	char hostname[MAXHOSTNAMELEN];
	const char *user, *tty;
	char **args, **pam_envlist, **pam_env;
	struct passwd *pwd;
	int o, pam_err, status;
	pid_t pid;

	while ((o = getopt(argc, argv, "h")) != -1)
		switch (o) {
		case 'h':
		default:
			usage();
		}

	argc -= optind;
	argv += optind;

	if (argc > 0) {
		user = *argv;
		--argc;
		++argv;
	} else {
		user = "root";
	}

	/* initialize PAM */
	pamc.conv = &openpam_ttyconv;
	pam_start("su", user, &pamc, &pamh);

	/* set some items */
	gethostname(hostname, sizeof(hostname));
	if ((pam_err = pam_set_item(pamh, PAM_RHOST, hostname)) != PAM_SUCCESS)
		goto pamerr;
	user = getlogin();
	if ((pam_err = pam_set_item(pamh, PAM_RUSER, user)) != PAM_SUCCESS)
		goto pamerr;
	tty = ttyname(STDERR_FILENO);
	if ((pam_err = pam_set_item(pamh, PAM_TTY, tty)) != PAM_SUCCESS)
		goto pamerr;

	/* authenticate the applicant */
	if ((pam_err = pam_authenticate(pamh, 0)) != PAM_SUCCESS)
		goto pamerr;
	if ((pam_err = pam_acct_mgmt(pamh, 0)) == PAM_NEW_AUTHTOK_REQD)
		pam_err = pam_chauthtok(pamh, PAM_CHANGE_EXPIRED_AUTHTOK);
	if (pam_err != PAM_SUCCESS)
		goto pamerr;

	/* establish the requested credentials */
	if ((pam_err = pam_setcred(pamh, PAM_ESTABLISH_CRED)) != PAM_SUCCESS)
		goto pamerr;

	/* authentication succeeded; open a session */
	if ((pam_err = pam_open_session(pamh, 0)) != PAM_SUCCESS)
		goto pamerr;

	/* get mapped user name; PAM may have changed it */
	pam_err = pam_get_item(pamh, PAM_USER, (const void **)&user);
	if (pam_err != PAM_SUCCESS || (pwd = getpwnam(user)) == NULL)
		goto pamerr;

	/* export PAM environment */
	if ((pam_envlist = pam_getenvlist(pamh)) != NULL) {
		for (pam_env = pam_envlist; *pam_env != NULL; ++pam_env) {
			putenv(*pam_env);
			free(*pam_env);
		}
		free(pam_envlist);
	}

	/* build argument list */
	if ((args = calloc(argc + 2, sizeof *args)) == NULL) {
		warn("calloc()");
		goto err;
	}
	*args = pwd->pw_shell;
	memcpy(args + 1, argv, argc * sizeof *args);

	/* fork and exec */
	switch ((pid = fork())) {
	case -1:
		warn("fork()");
		goto err;
	case 0:
		/* child: give up privs and start a shell */

		/* set uid and groups */
		if (initgroups(pwd->pw_name, pwd->pw_gid) == -1) {
			warn("initgroups()");
			_exit(1);
		}
		if (setgid(pwd->pw_gid) == -1) {
			warn("setgid()");
			_exit(1);
		}
		if (setuid(pwd->pw_uid) == -1) {
			warn("setuid()");
			_exit(1);
		}
		execve(*args, args, environ);
		warn("execve()");
		_exit(1);
	default:
		/* parent: wait for child to exit */
		waitpid(pid, &status, 0);

		/* close the session and release PAM resources */
		pam_err = pam_close_session(pamh, 0);
		pam_end(pamh, pam_err);

		exit(WEXITSTATUS(status));
	}

pamerr:
	fprintf(stderr, "Sorry\n");
err:
	pam_end(pamh, pam_err);
	exit(1);
}
```

## 附录 B: 示例 PAM 模块

以下是 pam_unix(8)  的最小实现，仅提供身份验证服务。它应该能够在大多数 PAM 实现中构建和运行，但如果有 OpenPAM 扩展的话会利用：请注意 pam_get_authtok(3) 的使用，它极大地简化了提示用户输入密码的过程。

```c
/*-
 * Copyright (c) 2002 Networks Associates Technology, Inc.
 * All rights reserved.
 *
 * This software was developed for the FreeBSD Project by ThinkSec AS and
 * Network Associates Laboratories, the Security Research Division of
 * Network Associates, Inc.  under DARPA/SPAWAR contract N66001-01-C-8035
 * ("CBOSS"), as part of the DARPA CHATS research program.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 * 3. The name of the author may not be used to endorse or promote
 *    products derived from this software without specific prior written
 *    permission.
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
 * $P4: //depot/projects/openpam/modules/pam_unix/pam_unix.c#3 $
 * $FreeBSD: head/en_US.ISO8859-1/articles/pam/pam_unix.c 38826 2012-05-17 19:12:14Z hrs $
 */

#include <sys/param.h>

#include <pwd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

#include <security/pam_modules.h>
#include <security/pam_appl.h>

#ifndef _OPENPAM
static char password_prompt[] = "Password:";
#endif

#ifndef PAM_EXTERN
#define PAM_EXTERN
#endif

PAM_EXTERN int
pam_sm_authenticate(pam_handle_t *pamh, int flags,
	int argc, const char *argv[])
{
#ifndef _OPENPAM
	struct pam_conv *conv;
	struct pam_message msg;
	const struct pam_message *msgp;
	struct pam_response *resp;
#endif
	struct passwd *pwd;
	const char *user;
	char *crypt_password, *password;
	int pam_err, retry;

	/* identify user */
	if ((pam_err = pam_get_user(pamh, &user, NULL)) != PAM_SUCCESS)
		return (pam_err);
	if ((pwd = getpwnam(user)) == NULL)
		return (PAM_USER_UNKNOWN);

	/* get password */
#ifndef _OPENPAM
	pam_err = pam_get_item(pamh, PAM_CONV, (const void **)&conv);
	if (pam_err != PAM_SUCCESS)
		return (PAM_SYSTEM_ERR);
	msg.msg_style = PAM_PROMPT_ECHO_OFF;
	msg.msg = password_prompt;
	msgp = &msg;
#endif
	for (retry = 0; retry < 3; ++retry) {
#ifdef _OPENPAM
		pam_err = pam_get_authtok(pamh, PAM_AUTHTOK,
		    (const char **)&password, NULL);
#else
		resp = NULL;
		pam_err = (*conv->conv)(1, &msgp, &resp, conv->appdata_ptr);
		if (resp != NULL) {
			if (pam_err == PAM_SUCCESS)
				password = resp->resp;
			else
				free(resp->resp);
			free(resp);
		}
#endif
		if (pam_err == PAM_SUCCESS)
			break;
	}
	if (pam_err == PAM_CONV_ERR)
		return (pam_err);
	if (pam_err != PAM_SUCCESS)
		return (PAM_AUTH_ERR);

	/* compare passwords */
	if ((!pwd->pw_passwd[0] && (flags & PAM_DISALLOW_NULL_AUTHTOK)) ||
	    (crypt_password = crypt(password, pwd->pw_passwd)) == NULL ||
	    strcmp(crypt_password, pwd->pw_passwd) != 0)
		pam_err = PAM_AUTH_ERR;
	else
		pam_err = PAM_SUCCESS;
#ifndef _OPENPAM
	free(password);
#endif
	return (pam_err);
}

PAM_EXTERN int
pam_sm_setcred(pam_handle_t *pamh, int flags,
	int argc, const char *argv[])
{

	return (PAM_SUCCESS);
}

PAM_EXTERN int
pam_sm_acct_mgmt(pam_handle_t *pamh, int flags,
	int argc, const char *argv[])
{

	return (PAM_SUCCESS);
}

PAM_EXTERN int
pam_sm_open_session(pam_handle_t *pamh, int flags,
	int argc, const char *argv[])
{

	return (PAM_SUCCESS);
}

PAM_EXTERN int
pam_sm_close_session(pam_handle_t *pamh, int flags,
	int argc, const char *argv[])
{

	return (PAM_SUCCESS);
}

PAM_EXTERN int
pam_sm_chauthtok(pam_handle_t *pamh, int flags,
	int argc, const char *argv[])
{

	return (PAM_SERVICE_ERR);
}

#ifdef PAM_MODULE_ENTRY
PAM_MODULE_ENTRY("pam_unix");
#endif
```

## 附录 C: 示例 PAM 对话函数

下面介绍的对话功能是 OpenPAM 的 openpam_ttyconv(3) 的一个极简化版本。它是完全功能的，应该能给读者一个对对话功能应如何行为的良好理解，但对于实际使用来说过于简单。即使您不使用 OpenPAM，也可以随意下载源代码并调整 openpam_ttyconv(3) 来适应您的需求；我们认为它尽可能健壮，适用于面向 tty 的对话功能。

```c
/*-
 * Copyright (c) 2002 Networks Associates Technology, Inc.
 * All rights reserved.
 *
 * This software was developed for the FreeBSD Project by ThinkSec AS and
 * Network Associates Laboratories, the Security Research Division of
 * Network Associates, Inc.  under DARPA/SPAWAR contract N66001-01-C-8035
 * ("CBOSS"), as part of the DARPA CHATS research program.
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions
 * are met:
 * 1. Redistributions of source code must retain the above copyright
 *    notice, this list of conditions and the following disclaimer.
 * 2. Redistributions in binary form must reproduce the above copyright
 *    notice, this list of conditions and the following disclaimer in the
 *    documentation and/or other materials provided with the distribution.
 * 3. The name of the author may not be used to endorse or promote
 *    products derived from this software without specific prior written
 *    permission.
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
 * $FreeBSD: head/en_US.ISO8859-1/articles/pam/converse.c 38826 2012-05-17 19:12:14Z hrs $
 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

#include <security/pam_appl.h>

int
converse(int n, const struct pam_message **msg,
	struct pam_response **resp, void *data)
{
	struct pam_response *aresp;
	char buf[PAM_MAX_RESP_SIZE];
	int i;

	data = data;
	if (n <= 0 || n > PAM_MAX_NUM_MSG)
		return (PAM_CONV_ERR);
	if ((aresp = calloc(n, sizeof *aresp)) == NULL)
		return (PAM_BUF_ERR);
	for (i = 0; i < n; ++i) {
		aresp[i].resp_retcode = 0;
		aresp[i].resp = NULL;
		switch (msg[i]->msg_style) {
		case PAM_PROMPT_ECHO_OFF:
			aresp[i].resp = strdup(getpass(msg[i]->msg));
			if (aresp[i].resp == NULL)
				goto fail;
			break;
		case PAM_PROMPT_ECHO_ON:
			fputs(msg[i]->msg, stderr);
			if (fgets(buf, sizeof buf, stdin) == NULL)
				goto fail;
			aresp[i].resp = strdup(buf);
			if (aresp[i].resp == NULL)
				goto fail;
			break;
		case PAM_ERROR_MSG:
			fputs(msg[i]->msg, stderr);
			if (strlen(msg[i]->msg) > 0 &&
			    msg[i]->msg[strlen(msg[i]->msg) - 1] != '\n')
				fputc('\n', stderr);
			break;
		case PAM_TEXT_INFO:
			fputs(msg[i]->msg, stdout);
			if (strlen(msg[i]->msg) > 0 &&
			    msg[i]->msg[strlen(msg[i]->msg) - 1] != '\n')
				fputc('\n', stdout);
			break;
		default:
			goto fail;
		}
	}
	*resp = aresp;
	return (PAM_SUCCESS);
 fail:
        for (i = 0; i < n; ++i) {
                if (aresp[i].resp != NULL) {
                        memset(aresp[i].resp, 0, strlen(aresp[i].resp));
                        free(aresp[i].resp);
                }
        }
        memset(aresp, 0, n * sizeof *aresp);
	*resp = NULL;
	return (PAM_CONV_ERR);
}
```

## 进一步阅读

### 论文

使登录服务独立于身份验证技术 Vipin Samar。查理莱。太阳微系统。

X/Open 单点登录初步规范。开放组织。1-85912-144-6。1997 年 6 月。

可插拔认证模块。安德鲁•G•摩根。1999 年 10 月 6 日。

### 用户手册

PAM 管理。太阳微系统。

### 相关网页

OpenPAM 主页 Dag-Erling Smørgrav。ThinkSec AS。

Linux-PAM 主页 Andrew Morgan。

Solaris PAM 主页。Sun Microsystems。
