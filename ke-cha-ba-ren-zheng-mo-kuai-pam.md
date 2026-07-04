# 可插拔认证模块（PAM）

- 原文：[Pluggable Authentication Modules](https://docs.freebsd.org/en/articles/pam/)

## 摘要

本文介绍了可插拔认证模块（PAM）库的基本原理与机制，解释了如何配置 PAM、如何将 PAM 集成进应用程序、以及如何编写 PAM 模块。


## 1. 引言

可插拔认证模块（PAM）库是一个通用的、用于认证相关服务的 API，它允许系统管理员仅通过安装新的 PAM 模块来添加新的认证方法，并且可以通过编辑配置文件来修改认证策略。

PAM 由 Sun Microsystems 的 Vipin Samar 和 Charlie Lai 于 1995 年设计和开发，自那以后基本未发生变化。1997 年，Open Group 发布了 X/Open Single Sign-on（XSSO）初步规范，它对 PAM API 进行了标准化，并增加了单点（或者说集成）登录的扩展功能。截至撰写本文时，这项规范尚未成为正式标准。

尽管本文主要聚焦于使用 OpenPAM 的 FreeBSD 5.x，但其内容同样适用于使用 Linux-PAM 的 FreeBSD 4.x，以及其他操作系统，如 Linux 和 Solaris™。


## 2. 术语与约定

### 2.1. 定义

PAM 相关术语相当混乱。Samar 与 Lai 的原始论文以及 XSSO 规范都未尝试对参与 PAM 的各方和实体进行正式定义，他们所使用（但未定义）的术语有时具有误导性或模糊性。第一个建立一致且明确术语体系的尝试是 Andrew G. Morgan（Linux-PAM 作者）在 1999 年撰写的一篇白皮书。尽管 Morgan 的术语选择是一次巨大进步，但在本文作者看来仍不尽完善。以下定义在很大程度上受到 Morgan 的启发，试图为所有参与 PAM 的行为体与实体制定精确而明确的术语。

- **account（账户）**
  申请者向仲裁者请求的一组凭证。

- **applicant（申请者）**
  发起认证请求的用户或实体。

- **arbitrator（仲裁者）**
  拥有验证申请者凭据的权限以及授予或拒绝请求权力的用户或实体。

- **chain（链）**
  响应 PAM 请求而调用的一系列模块。链中包括了调用模块的顺序、传递给模块的参数，以及如何解释模块返回结果的信息。

- **client（客户端）**
  代表申请者发起认证请求的应用程序，负责从申请者处获取所需认证信息。

- **facility（功能组）**
  PAM 提供的四类基本功能之一：认证（authentication）、账户管理（account management）、会话管理（session management）以及认证令牌更新（authentication token update）。

- **module（模块）**
  实现某一特定认证功能组的一组相关函数，集合成一个单独（通常是可动态加载）的二进制文件，并以一个名称标识。

- **policy（策略）**
  描述如何处理某一服务的 PAM 请求的完整配置语句集。一个策略通常由四条链组成，每条链对应一个功能组，尽管某些服务并不使用全部四个功能组。

- **server（服务器）**
  代表仲裁者执行操作的应用程序，用于与客户端交互、获取认证信息、验证申请者凭据，并决定是否授予请求。

- **service（服务）**
  提供相似或相关功能，并具有类似认证需求的一类服务器。PAM 策略以服务为单位定义，所有使用相同服务名的服务器都将遵循相同的策略。

- **session（会话）**
  服务器为申请者提供服务的上下文环境。PAM 的四个功能组之一 —— 会话管理（session management）—— 专门负责该上下文的创建与销毁。

- **token（令牌）**
  与账户关联的一段信息，如密码或口令，申请者需提供此信息以证明其身份。

- **transaction（事务）**
  同一申请者对同一服务器实例发起的一系列请求，从认证与会话建立开始，到会话结束为止。


### 2.2. 使用示例

本节通过一些简单的示例，说明前面定义的一些术语的含义。

#### 2.2.1. 客户端和服务器为同一实体

这个简单的例子展示了 `alice` 使用 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 切换到 `root`。

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

- 申请者是 `alice`。
- 账户是 `root`。
- [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 进程既是客户端也是服务器。
- 认证令牌是 `xi3kiune`。
- 仲裁者是 `root`，因此 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 是 setuid `root`。

#### 2.2.2. 客户端和服务器为分离实体

下面的例子展示了 `eve` 尝试发起到 `login.example.com` 的 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 连接，要求以 `bob` 身份登录，并成功登录。Bob 本该选择一个更好的密码！

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

- 申请者是 `eve`。
- 客户端是 Eve 的 [ssh(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh&sektion=1&format=html) 进程。
- 服务器是 `login.example.com` 上的 [sshd(8)](https://man.freebsd.org/cgi/man.cgi?query=sshd&sektion=8&format=html) 进程。
- 账户是 `bob`。
- 认证令牌是 `god`。
- 尽管本示例中未显示，但仲裁者是 `root`。

#### 2.2.3. 示例策略

以下是 FreeBSD 默认的 `sshd` 策略：

```ini
sshd	auth		required	pam_nologin.so	no_warn
sshd	auth		required	pam_unix.so	no_warn try_first_pass
sshd	account		required	pam_login_access.so
sshd	account		required	pam_unix.so
sshd	session		required	pam_lastlog.so	no_fail
sshd	password	required	pam_permit.so
```

- 该策略适用于 `sshd` 服务（不一定仅限于 [sshd(8)](https://man.freebsd.org/cgi/man.cgi?query=sshd&sektion=8&format=html) 服务器）。
- `auth`、`account`、`session` 和 `password` 是四个功能组。
- `pam_nologin.so`、`pam_unix.so`、`pam_login_access.so`、`pam_lastlog.so` 和 `pam_permit.so` 是模块。通过这个示例可以看出，`pam_unix.so` 至少提供了两个功能组（认证和账户管理）。

## 3. PAM 基本概念

### 3.1. 功能和原语

PAM API 提供了六个不同的认证原语，这些原语被分组到四个功能组中，具体描述如下。

`auth`
*认证。* 该功能组关注于认证申请者并建立账户凭证。它提供了两个原语：

- [pam_authenticate(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_authenticate&sektion=3&format=html) 通常通过请求认证令牌并将其与存储在数据库中的值或从认证服务器获取的值进行比较来认证申请者。
- [pam_setcred(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_setcred&sektion=3&format=html) 建立账户凭证，如用户 ID、组成员资格和资源限制。

`account`
*账户管理。* 该功能组处理与认证无关的账户可用性问题，如基于一天中的时间或服务器工作负载的访问限制。它提供了一个原语：

- [pam_acct_mgmt(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_acct_mgmt&sektion=3&format=html) 验证请求的账户是否可用。

`session`
*会话管理。* 该功能组处理与会话设置和拆卸相关的任务，如登录记账。它提供了两个原语：

- [pam_open_session(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_open_session&sektion=3&format=html) 执行与会话设置相关的任务：在 `utmp` 和 `wtmp` 数据库中添加条目，启动 SSH 代理等。
- [pam_close_session(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_close_session&sektion=3&format=html) 执行与会话拆卸相关的任务：在 `utmp` 和 `wtmp` 数据库中添加条目，停止 SSH 代理等。

`password`
*密码管理。* 该功能组用于更改与账户关联的认证令牌，要么是因为令牌已过期，要么是因为用户希望更改它。它提供了一个原语：

- [pam_chauthtok(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_chauthtok&sektion=3&format=html) 更改认证令牌，选择性地验证其是否足够难以猜测，是否未曾使用过等。

### 3.2. 模块

模块是 PAM 中一个非常核心的概念；毕竟，它们是 “PAM” 中的 "M"。PAM 模块是一个自包含的程序代码，负责实现一个或多个功能组中的原语，针对特定的机制；例如，认证功能组的机制可能包括 UNIX® 密码数据库、NIS、LDAP 和 Radius 等。

#### 3.2.1. 模块命名

FreeBSD 每个机制都实现为一个单独的模块，命名为 `pam_mechanism.so`（例如，针对 UNIX® 机制的模块是 `pam_unix.so`）。其他实现有时会为不同的功能提供单独的模块，并将功能名称与机制名称一起包括在模块名称中。例如，Solaris™ 有一个名为 `pam_dial_auth.so.1` 的模块，通常用于认证拨号用户。

#### 3.2.2. 模块版本控制

FreeBSD 最初基于 Linux-PAM 的 PAM 实现没有为 PAM 模块使用版本号。这通常会导致与旧版应用程序相关的问题，因为这些应用程序可能链接到较旧版本的系统库，而无法加载所需模块的匹配版本。

另一方面，OpenPAM 会查找与 PAM 库（当前版本为 2）具有相同版本号的模块，如果找不到对应版本的模块，则回退到没有版本号的模块。因此，可以为旧版应用程序提供旧版模块，同时允许新（或新构建的）应用程序利用最新的模块。

虽然 Solaris™ PAM 模块通常会有版本号，但它们并没有真正的版本控制，因为版本号是模块名称的一部分，必须在配置中包含该版本号。

### 3.3. 链和策略

当服务器发起 PAM 事务时，PAM 库尝试加载在 [pam_start(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_start&sektion=3&format=html) 调用中指定的服务的策略。该策略指定了认证请求应如何处理，并在配置文件中定义。这是 PAM 中的另一个核心概念：管理员可以通过简单地编辑文本文件来调整系统安全策略（在广义上理解）。

一个策略由四个链组成，每个链对应一个 PAM 功能组。每个链都是一个配置语句的序列，每个语句指定要调用的模块、传递给模块的（可选）参数以及描述如何解释模块返回码的控制标志。

理解控制标志对于理解 PAM 配置文件至关重要。控制标志有五种不同的类型：

`binding`
如果模块成功并且链中没有任何先前的模块失败，则链会立即终止，请求被批准。如果模块失败，则会执行链中的其余部分，但请求最终会被拒绝。

该控制标志由 Sun 在 Solaris™ 9（SunOS™ 5.9）中引入，OpenPAM 也支持此标志。

`required`
如果模块成功，则执行链中的其余部分，并且请求被批准，除非其他某个模块失败。如果模块失败，则链中的其余部分也会被执行，但请求最终会被拒绝。

`requisite`
如果模块成功，则执行链中的其余部分，请求被批准，除非其他某个模块失败。如果模块失败，链会立即终止，请求被拒绝。

`sufficient`
如果模块成功并且链中的先前模块没有失败，则链会立即终止，请求被批准。如果模块失败，则该模块会被忽略，链中的其余部分会被执行。

由于该标志的语义可能有些混淆，特别是在用于链中的最后一个模块时，如果实现支持，建议使用 `binding` 控制标志来代替。

`optional`
模块会被执行，但其结果会被忽略。如果链中的所有模块都标记为 `optional`，所有请求将始终被批准。

当服务器调用六个 PAM 原语中的一个时，PAM 会检索与该原语所属功能组对应的链，并按链中列出的顺序依次调用每个模块，直到到达链的末尾，或确定不再需要进一步处理（无论是因为 `binding` 或 `sufficient` 模块成功，还是因为 `requisite` 模块失败）。请求只有在至少一个模块被调用并且所有非 `optional` 模块都成功时才会被批准。

请注意，虽然不常见，但在同一链中列出相同模块多次是可能的。例如，一个用于查找用户名称和密码的模块，可以多次调用，指定不同的目录服务器进行查询。PAM 会将同一链中同一模块的不同出现视为不同、无关的模块。

### 3.4. 事务

典型 PAM 事务的生命周期如下所述。请注意，如果这些步骤中的任何一个失败，服务器应该向客户端报告适当的错误消息并中止事务。

1. 如有必要，服务器通过独立于 PAM 的机制获取仲裁者凭证——最常见的情况是通过 `root` 启动，或通过 setuid `root`。
2. 服务器调用 [pam_start(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_start&sektion=3&format=html) 来初始化 PAM 库，并指定其服务名称和目标账户，并注册适当的对话函数。
3. 服务器获取与事务相关的各种信息（例如申请人的用户名和客户端运行的主机名称），并使用 [pam_set_item(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_set_item&sektion=3&format=html) 将其提交给 PAM。
4. 服务器调用 [pam_authenticate(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_authenticate&sektion=3&format=html) 来验证申请人。
5. 服务器调用 [pam_acct_mgmt(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_acct_mgmt&sektion=3&format=html) 来验证请求的账户是否可用且有效。如果密码正确但已过期，[pam_acct_mgmt(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_acct_mgmt&sektion=3&format=html) 将返回 `PAM_NEW_AUTHTOK_REQD` 而不是 `PAM_SUCCESS`。
6. 如果前一步返回 `PAM_NEW_AUTHTOK_REQD`，服务器现在调用 [pam_chauthtok(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_chauthtok&sektion=3&format=html) 强制客户端更改请求账户的认证令牌。
7. 既然申请人已经正确地通过了认证，服务器调用 [pam_setcred(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_setcred&sektion=3&format=html) 来建立请求账户的凭证。它能够做到这一点，因为它代表仲裁者行事，并持有仲裁者的凭证。
8. 在正确的凭证建立之后，服务器调用 [pam_open_session(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_open_session&sektion=3&format=html) 来设置会话。
9. 服务器现在执行客户端请求的任何服务——例如，提供一个 shell 给申请人。
10. 服务器完成为客户端提供服务后，它调用 [pam_close_session(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_close_session&sektion=3&format=html) 来拆除会话。
11. 最后，服务器调用 [pam_end(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_end&sektion=3&format=html) 来通知 PAM 库它已经完成，可以释放在事务过程中分配的所有资源。

## 4. PAM 配置

### 4.1. PAM 策略文件

#### 4.1.1. **/etc/pam.conf**

传统的 PAM 策略文件是 **/etc/pam.conf**。该文件包含系统的所有 PAM 策略。文件中的每一行描述了链中的一步，如下所示：

```sh
login   auth    required        pam_nologin.so  no_warn
```

字段顺序如下：服务名称、功能名称、控制标志、模块名称和模块参数。任何额外的字段都被解释为额外的模块参数。

为每个服务 / 功能对构建单独的链，因此虽然相同服务和功能的行顺序是重要的，但列出服务和功能的顺序并不重要。原始 PAM 论文中的示例按功能分组配置行，Solaris™ 的默认 **pam.conf** 文件仍然这样做，但 FreeBSD 的默认配置将配置行按服务分组。两者都可以，任何一种方式都合理。

#### 4.1.2. **/etc/pam.d**

OpenPAM 和 Linux-PAM 支持一种替代配置机制，这是 FreeBSD 推荐的机制。在这种方案中，每个策略都包含在一个独立的文件中，文件名是应用该策略的服务名。这些文件存储在 **/etc/pam.d/** 目录下。

这些按服务区分的策略文件只有四个字段，而不是 **pam.conf** 中的五个：服务名称字段被省略了。因此，在 **/etc/pam.d/login** 文件中，你会看到如下行，而不是之前的 **pam.conf** 示例：

```sh
auth    required        pam_nologin.so  no_warn
```

由于这种简化的语法，可以通过将每个服务名称链接到相同的策略文件来为多个服务使用相同的策略。例如，要为 `su` 和 `sudo` 服务使用相同的策略，可以按如下方式操作：

```sh
# cd /etc/pam.d
# ln -s su sudo
```

之所以可行，是因为服务名称是通过文件名确定的，而不是在策略文件中指定的，因此相同的文件可以用于多个不同命名的服务。

由于每个服务的策略存储在单独的文件中，**pam.d** 机制还使得为第三方软件包安装额外策略变得非常容易。

#### 4.1.3. 策略搜索顺序

如前所述，PAM 策略可以在多个地方找到。如果相同服务的策略存在于多个位置，会发生什么呢？

理解 PAM 的配置系统以链为核心至关重要。

### 4.2. 配置行的解析

如 [PAM 策略文件](https://docs.freebsd.org/en/articles/pam/#pam-config-file) 中所解释的，**/etc/pam.conf** 中的每一行包含四个或更多字段：服务名称、功能名称、控制标志、模块名称以及零个或多个模块参数。

服务名称通常是（但不总是）该语句适用的应用程序的名称。如果不确定，请参考各个应用程序的文档，以确定它使用的服务名称。

请注意，如果使用 **/etc/pam.d/** 而不是 **/etc/pam.conf**，服务名称由策略文件的名称指定，并且在实际的配置行中省略，配置行从功能名称开始。

功能是 [功能和原语](https://docs.freebsd.org/en/articles/pam/#pam-facilities-primitives) 中描述的四个功能关键字之一。

同样，控制标志是 [链和策略](https://docs.freebsd.org/en/articles/pam/#pam-chains-policies) 中描述的四个关键字之一，描述如何解释模块返回的代码。Linux-PAM 支持一种替代语法，允许你为每个可能的返回码指定关联的操作，但应避免使用这种语法，因为它是非标准的，并且与 Linux-PAM 调度服务调用的方式紧密相关（与 Solaris™ 和 OpenPAM 的方式差异很大）。不出所料，OpenPAM 不支持这种语法。

### 4.3. 策略

为了正确配置 PAM，理解策略的解释方式至关重要。

当应用程序调用 [pam_start(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_start&sektion=3&format=html) 时，PAM 库加载指定服务的策略，并构建四个模块链，每个功能对应一个。如果这些链中的一个或多个为空，则会用 `other` 服务的策略中的相应链进行替换。

当应用程序稍后调用其中一个 PAM 原语时，PAM 库检索对应功能的链，并按配置中列出的顺序调用链中每个模块的相应服务函数。在每次调用服务函数后，模块类型和服务函数返回的错误代码将用于决定接下来发生的事情。以下表格适用于大多数情况，除了几个例外，我们将在下面讨论：

**表 1. PAM 链执行摘要**

|            | PAM_SUCCESS  | PAM_IGNORE | 其他               |
| ---------- | ------------- | ---------- | ---------------- |
| binding    | if (!fail) break; | -          | fail = true;     |
| required   | -             | -          | fail = true;     |
| requisite  | -             | -          | fail = true; break; |
| sufficient | if (!fail) break; | -          | -                |
| optional   | -             | -          | -                |

如果在链的末尾，或者遇到 "break" 时，`fail` 为 true，则调度器返回第一个失败模块返回的错误代码。否则，返回 `PAM_SUCCESS`。

第一个值得注意的例外是，错误代码 `PAM_NEW_AUTHTOK_REQD` 被视为成功，但是如果没有任何模块失败，并且至少有一个模块返回了 `PAM_NEW_AUTHTOK_REQD`，调度器将返回 `PAM_NEW_AUTHTOK_REQD`。

第二个例外是 [pam_setcred(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_setcred&sektion=3&format=html) 将 `binding` 和 `sufficient` 模块视为 `required` 模块。

第三个也是最后一个例外是 [pam_chauthtok(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_chauthtok&sektion=3&format=html) 会运行整个链两次（一次用于初步检查，一次用于实际设置密码），在初步阶段，它将 `binding` 和 `sufficient` 模块视为 `required` 模块。

## 5. FreeBSD PAM 模块

### 5.1. [pam_deny(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_deny&sektion=8&format=html)

[pam_deny(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_deny&sektion=8&format=html) 模块是最简单的模块之一，它对任何请求返回 `PAM_AUTH_ERR`。它对于快速禁用某个服务（将其添加到每个链的顶部）或者终止 `sufficient` 模块链非常有用。

### 5.2. [pam_echo(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_echo&sektion=8&format=html)

[pam_echo(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_echo&sektion=8&format=html) 模块将其参数作为 `PAM_TEXT_INFO` 消息传递给对话函数。它主要用于调试，但也可以用来在开始认证过程之前显示消息，例如 "Unauthorized access will be prosecuted"。

### 5.3. [pam_exec(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_exec&sektion=8&format=html)

[pam_exec(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_exec&sektion=8&format=html) 模块将其第一个参数作为要执行的程序名，剩余的参数作为命令行参数传递给该程序。一种可能的应用是在登录时运行一个程序来挂载用户的主目录。

### 5.4. [pam_ftpusers(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_ftpusers&sektion=8&format=html)

[pam_ftpusers(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_ftpusers&sektion=8&format=html) 模块

### 5.5. [pam_group(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_group&sektion=8&format=html)

[pam_group(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_group&sektion=8&format=html) 模块根据用户是否属于某个特定文件组（对 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 而言通常是 `wheel`）来接受或拒绝申请人。它主要用于维持 BSD 传统的 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 行为，但也有其他用途，例如排除某些用户组使用特定服务。

### 5.6. [pam_guest(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_guest&sektion=8&format=html)

[pam_guest(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_guest&sektion=8&format=html) 模块允许使用固定登录名的访客登录。可以对密码施加各种要求，但默认行为是只要登录名是访客账户的名字，就允许任何密码。此模块可以轻松用于实现匿名 FTP 登录。

### 5.7. [pam_krb5(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_krb5&sektion=8&format=html)

[pam_krb5(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_krb5&sektion=8&format=html) 模块

### 5.8. [pam_ksu(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_ksu&sektion=8&format=html)

[pam_ksu(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_ksu&sektion=8&format=html) 模块

### 5.9. [pam_lastlog(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_lastlog&sektion=8&format=html)

[pam_lastlog(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_lastlog&sektion=8&format=html) 模块

### 5.10. [pam_login_access(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_login_access&sektion=8&format=html)

[pam_login_access(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_login_access&sektion=8&format=html) 模块提供了一个账户管理原语的实现，强制执行 [login.access(5)](https://man.freebsd.org/cgi/man.cgi?query=login.access&sektion=5&format=html) 表中指定的登录限制。

### 5.11. [pam_nologin(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_nologin&sektion=8&format=html)

[pam_nologin(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_nologin&sektion=8&format=html) 模块在 **/var/run/nologin** 文件存在时拒绝非 root 用户的登录。这个文件通常由 [shutdown(8)](https://man.freebsd.org/cgi/man.cgi?query=shutdown&sektion=8&format=html) 在剩余时间少于五分钟时创建。

### 5.12. [pam_passwdqc(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_passwdqc&sektion=8&format=html)

[pam_passwdqc(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_passwdqc&sektion=8&format=html) 模块

### 5.13. [pam_permit(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_permit&sektion=8&format=html)

[pam_permit(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_permit&sektion=8&format=html) 模块是最简单的模块之一，它对任何请求返回 `PAM_SUCCESS`。它作为占位符很有用，适用于那些本应为空的服务链。

### 5.14. [pam_radius(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_radius&sektion=8&format=html)

[pam_radius(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_radius&sektion=8&format=html) 模块

### 5.15. [pam_rhosts(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_rhosts&sektion=8&format=html)

[pam_rhosts(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_rhosts&sektion=8&format=html) 模块

### 5.16. [pam_rootok(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_rootok&sektion=8&format=html)

[pam_rootok(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_rootok&sektion=8&format=html) 模块仅在调用它的进程的真实用户 ID 为 0 时才返回成功。这对于非网络服务（如 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 或 [passwd(1)](https://man.freebsd.org/cgi/man.cgi?query=passwd&sektion=1&format=html)）很有用，对这些服务，`root` 应自动具有访问权限。

### 5.17. [pam_securetty(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_securetty&sektion=8&format=html)

[pam_securetty(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_securetty&sektion=8&format=html) 模块

### 5.18. [pam_self(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_self&sektion=8&format=html)

[pam_self(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_self&sektion=8&format=html) 模块仅在申请人的名字与目标账户的名字匹配时返回成功。它最适用于非网络服务，如 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html)，在这些服务中，可以轻松验证申请人的身份。

### 5.19. [pam_ssh(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_ssh&sektion=8&format=html)

[pam_ssh(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_ssh&sektion=8&format=html) 模块提供了身份验证和会话服务。身份验证服务允许在 **~/.ssh** 目录中拥有口令保护的 SSH 私钥的用户通过输入口令来进行身份验证。会话服务启动 [ssh-agent(1)](https://man.freebsd.org/cgi/man.cgi?query=ssh-agent&sektion=1&format=html) 并将在认证阶段解密的密钥预加载到其中。这个特性对于本地登录特别有用，无论是在 X 窗口（使用 [xdm(8)](https://man.freebsd.org/cgi/man.cgi?query=xdm&sektion=8&format=html) 或其他支持 PAM 的 X 登录管理器）还是在控制台。

### 5.20. [pam_tacplus(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_tacplus&sektion=8&format=html)

[pam_tacplus(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_tacplus&sektion=8&format=html) 模块

### 5.21. [pam_unix(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_unix&sektion=8&format=html)

[pam_unix(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_unix&sektion=8&format=html) 模块实现了传统的 UNIX® 密码认证，使用 [getpwnam(3)](https://man.freebsd.org/cgi/man.cgi?query=getpwnam&sektion=3&format=html) 获取目标账户的密码，并将其与申请人提供的密码进行比较。它还提供账户管理服务（强制执行账户和密码到期时间）以及密码更改服务。这可能是最有用的模块，因为大多数管理员都希望至少为某些服务保留历史行为。

## 6. PAM 应用程序编程

本节尚未编写。

## 7. PAM 模块编程

本节尚未编写。

## 附录 A：示例 PAM 应用程序

以下是使用 PAM 的 [su(1)](https://man.freebsd.org/cgi/man.cgi?query=su&sektion=1&format=html) 最小实现。请注意，它使用了 OpenPAM 特有的 [openpam_ttyconv(3)](https://man.freebsd.org/cgi/man.cgi?query=openpam_ttyconv&sektion=3&format=html) 对话函数，该函数在 **security/openpam.h** 中定义。如果你希望在使用不同 PAM 库的系统上构建此应用程序，则必须提供自己的对话函数。实现一个强健的对话函数相当困难；在 [示例 PAM 对话函数](https://docs.freebsd.org/en/articles/pam/#pam-sample-conv) 中提供的实现是一个不错的起点，但不应在实际应用中使用。

```c
/*-
 * Copyright (c) 2002,2003 Networks Associates Technology, Inc.
 * All rights reserved.
 *
 * 本软件由 ThinkSec AS 和 Network Associates Laboratories（Network Associates, Inc. 的安全研究部门）为 FreeBSD 项目开发，
 * 由 DARPA/SPAWAR 合同 N66001-01-C-8035（“CBOSS”）支持，作为 DARPA CHATS 研究项目的一部分。
 *
 * 允许在源代码和二进制形式中进行再分发和使用，无论是否修改，前提是满足以下条件：
 * 1. 源代码的再分发必须保留上述版权声明、此条件列表和以下免责声明。
 * 2. 二进制形式的再分发必须在分发的文档和/或其他材料中重现上述版权声明、此条件列表和以下免责声明。
 * 3. 未经特定的书面许可，不得使用作者的名字来支持或推广基于此软件的产品。
 *
 * 本软件由作者和贡献者“按原样”提供，不提供任何明示或暗示的保证，包括但不限于适销性和适用性保证，
 * 对于任何直接、间接、附带、特殊、示范性或间接损害（包括但不限于采购替代商品或服务；使用、数据或利润的丧失；
 * 或商业中断）在任何理论的责任下，无论是基于合同、严格责任或侵权（包括过失或其他）产生的，
 * 即使在已被告知此类损害的可能性的情况下，也不承担责任。
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
#include <security/openpam.h>	/* openpam_ttyconv() 需要 */

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

	/* 初始化 PAM */
	pamc.conv = &openpam_ttyconv;
	pam_start("su", user, &pamc, &pamh);

	/* 设置一些项目 */
	gethostname(hostname, sizeof(hostname));
	if ((pam_err = pam_set_item(pamh, PAM_RHOST, hostname)) != PAM_SUCCESS)
		goto pamerr;
	user = getlogin();
	if ((pam_err = pam_set_item(pamh, PAM_RUSER, user)) != PAM_SUCCESS)
		goto pamerr;
	tty = ttyname(STDERR_FILENO);
	if ((pam_err = pam_set_item(pamh, PAM_TTY, tty)) != PAM_SUCCESS)
		goto pamerr;

	/* 验证申请人 */
	if ((pam_err = pam_authenticate(pamh, 0)) != PAM_SUCCESS)
		goto pamerr;
	if ((pam_err = pam_acct_mgmt(pamh, 0)) == PAM_NEW_AUTHTOK_REQD)
		pam_err = pam_chauthtok(pamh, PAM_CHANGE_EXPIRED_AUTHTOK);
	if (pam_err != PAM_SUCCESS)
		goto pamerr;

	/* 建立请求的凭证 */
	if ((pam_err = pam_setcred(pamh, PAM_ESTABLISH_CRED)) != PAM_SUCCESS)
		goto pamerr;

	/* 认证成功；打开会话 */
	if ((pam_err = pam_open_session(pamh, 0)) != PAM_SUCCESS)
		goto pamerr;

	/* 获取映射的用户名；PAM 可能已更改它 */
	pam_err = pam_get_item(pamh, PAM_USER, (const void **)&user);
	if (pam_err != PAM_SUCCESS || (pwd = getpwnam(user)) == NULL)
		goto pamerr;

	/* 导出 PAM 环境变量 */
	if ((pam_envlist = pam_getenvlist(pamh)) != NULL) {
		for (pam_env = pam_envlist; *pam_env != NULL; ++pam_env) {
			putenv(*pam_env);
			free(*pam_env);
		}
		free(pam_envlist);
	}

	/* 构建参数列表 */
	if ((args = calloc(argc + 2, sizeof *args)) == NULL) {
		warn("calloc()");
		goto err;
	}
	*args = pwd->pw_shell;
	memcpy(args + 1, argv, argc * sizeof *args);

	/* fork 和 exec */
	switch ((pid = fork())) {
	case -1:
		warn("fork()");
		goto err;
	case 0:
		/* 子进程：放弃权限并启动一个 shell */

		/* 设置 uid 和组 */
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
		/* 父进程：等待子进程退出 */
		waitpid(pid, &status, 0);

		/* 关闭会话并释放 PAM 资源 */
		pam_err = pam_close_session(pamh, 0);
		pam_end(pamh, pam_err);

		exit(WEXITSTATUS(status));
	}

pamerr:
	fprintf(stderr, "抱歉\n");
err:
	pam_end(pamh, pam_err);
	exit(1);
}
```

## 附录 B：示例 PAM 模块

以下是 [pam_unix(8)](https://man.freebsd.org/cgi/man.cgi?query=pam_unix&sektion=8&format=html) 的最小实现，仅提供认证服务。它应当能够与大多数 PAM 实现一起构建和运行，但如果可用，它会利用 OpenPAM 扩展：请注意使用了 [pam_get_authtok(3)](https://man.freebsd.org/cgi/man.cgi?query=pam_get_authtok&sektion=3&format=html)，这大大简化了提示用户输入密码的过程。

```c
/*-
* 版权所有 (c) 2002 Networks Associates Technology, Inc.
* 保留所有权利。
*
* 本软件是由 ThinkSec AS 和 Network Associates Laboratories（Network Associates, Inc. 安全研究部门）为 FreeBSD 项目开发的，
* 根据 DARPA/SPAWAR 合同 N66001-01-C-8035 ("CBOSS")，作为 DARPA CHATS 研究计划的一部分。
*
* 允许以源代码或二进制形式进行再分发和使用，无论是否修改，前提是满足以下条件：
* 1. 源代码的再分发必须保留上述版权声明、此条件列表和以下免责声明。
* 2. 二进制形式的再分发必须在分发的文档和/或其他材料中复制上述版权声明、此条件列表和以下免责声明。
* 3. 未经特定的书面许可，不得使用作者的姓名来支持或推广基于本软件的产品。
*
* 本软件由作者和贡献者以“原样”提供，不提供任何明示或暗示的保证，包括但不限于适销性和适合某一特定用途的暗示保证。
* 在任何情况下，作者或贡献者都不对因使用本软件而导致的任何直接、间接、偶然、特殊、示范性或间接损害负责，
* 包括但不限于替代商品或服务的采购；使用、数据或利润的损失；或业务中断，不论是在合同、严格责任或侵权（包括过失或其他）下，
* 即使已经被告知可能发生此类损害，也不承担任何责任。
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

	/* 识别用户 */
	if ((pam_err = pam_get_user(pamh, &user, NULL)) != PAM_SUCCESS)
		return (pam_err);
	if ((pwd = getpwnam(user)) == NULL)
		return (PAM_USER_UNKNOWN);

	/* 获取密码 */
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

	/* 比较密码 */
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

## 附录 C：示例 PAM 对话函数

以下是 OpenPAM [openpam_ttyconv(3)](https://man.freebsd.org/cgi/man.cgi?query=openpam_ttyconv&sektion=3&format=html) 的极大简化版本。它是完全功能的，应该能让读者大致了解对话函数的工作方式，但它对于真实世界的使用来说过于简单。即使你不使用 OpenPAM，也可以下载源代码并根据你的需求调整 [openpam_ttyconv(3)](https://man.freebsd.org/cgi/man.cgi?query=openpam_ttyconv&sektion=3&format=html)；我们认为它是一个足够健壮的 tty 导向的对话函数。

```c
/*-
 * 版权所有 (c) 2002 Networks Associates Technology, Inc.
 * 保留所有权利。
 *
 * 本软件是由 ThinkSec AS 和 Network Associates Laboratories（Network Associates, Inc. 安全研究部门）为 FreeBSD 项目开发的，
 * 根据 DARPA/SPAWAR 合同 N66001-01-C-8035 ("CBOSS")，作为 DARPA CHATS 研究计划的一部分。
 *
 * 允许以源代码或二进制形式进行再分发和使用，无论是否修改，前提是满足以下条件：
 * 1. 源代码的再分发必须保留上述版权声明、此条件列表和以下免责声明。
 * 2. 二进制形式的再分发必须在分发的文档和/或其他材料中复制上述版权声明、此条件列表和以下免责声明。
 * 3. 未经特定的书面许可，不得使用作者的姓名来支持或推广基于本软件的产品。
 *
 * 本软件由作者和贡献者以“原样”提供，不提供任何明示或暗示的保证，包括但不限于适销性和适合某一特定用途的暗示保证。
 * 在任何情况下，作者或贡献者都不对因使用本软件而导致的任何直接、间接、偶然、特殊、示范性或间接损害负责，
 * 包括但不限于替代商品或服务的采购；使用、数据或利润的损失；或业务中断，不论是在合同、严格责任或侵权（包括过失或其他）下，
 * 即使已经被告知可能发生此类损害，也不承担任何责任。
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

- Making Login Services Independent of Authentication Technologies Vipin Samar. Charlie Lai. Sun Microsystems.

- [X/Open 单点登录初步规范](https://pubs.opengroup.org/onlinepubs/8329799/toc.htm). The Open Group. 1-85912-144-6. 1997年6月。

- [可插拔认证模块](https://mirrors.kernel.org/pub/linux/libs/pam/pre/doc/draft-morgan-pam-07.txt). Andrew G. Morgan. 1999-10-06。

### 用户手册

- [PAM 管理](https://docs.oracle.com/cd/E26505_01/html/E27224/pam-1.html). Sun Microsystems。

### 相关网页

- [OpenPAM 主页](https://www.openpam.org/) Dag-Erling Smørgrav. ThinkSec AS。

- [Linux-PAM 主页](http://www.kernel.org/pub/linux/libs/pam/) Andrew Morgan。

- Solaris PAM 主页. Sun Microsystems。
