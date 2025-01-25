# LDAP 身份验证

版权 © 2007-2008 FreeBSD 文档项目

<details open="" data-immersive-translate-walked="7bfc157f-e618-4bf3-85fd-24f436fb361b"><summary data-immersive-translate-walked="7bfc157f-e618-4bf3-85fd-24f436fb361b" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标.

制造商和销售商用来区分其产品的许多标识，被视为商标。在本文件中出现这些标识时，FreeBSD 项目已经意识到商标声明，这些标识后面已经加上了“™”或“®”符号。

</details>

摘要

本文件旨在指导如何在 FreeBSD 上配置 LDAP 服务器（主要是 OpenLDAP 服务器）以进行身份验证。这对于许多服务器需要相同用户帐户的情况非常有用，例如作为 NIS 的替代方案。

---

## 1. 序言

本文档旨在让读者充分了解 LDAP 以配置 LDAP 服务器。 本文档将尝试解释用于客户端机器服务的 net/nss_ldap 和 security/pam_ldap，以便与 LDAP 服务器一起使用。

当阅读完毕时，读者应该能够配置和部署一个可以托管 LDAP 目录的 FreeBSD 服务器，并配置和部署一个可以对 LDAP 目录进行身份验证的 FreeBSD 服务器。

这篇文章不打算详尽介绍配置 LDAP 或其他讨论的服务的安全性、健壮性或最佳实践考虑因素。虽然作者尽力做一切正确的事情，但他们并未涉及超出一般范围的安全问题。这篇文章应被视为仅建立理论基础，任何实际实施应伴随着仔细的需求分析。

## 配置 LDAP

LDAP 代表“轻量级目录访问协议”，是 X.500 目录访问协议的一个子集。其最新规范在 RFC4510 及相关文档中。基本上，它是一个期望被更频繁读取而不是写入的数据库。

本文档中的示例将使用 LDAP 服务器 OpenLDAP；虽然这里的原则应该普遍适用于许多不同的服务器，但大部分具体的管理工作是针对 OpenLDAP 的。例如，有几个服务器版本在 ports 中，比如 net/openldap26-server。客户端服务器将需要相应的 net/openldap26-client 库。

LDAP 服务中基本上有两个需要配置的领域。第一个是设置服务器以正确接收连接，第二个是向服务器目录中添加条目，以便 FreeBSD 工具知道如何与其交互。

### 2.1. 为连接设置服务器

|     | 本节专门针对 OpenLDAP。如果您使用其他服务器，您需要查阅该服务器的文档。 |
| --- | ----------------------------------------------------------------------- |

#### 2.1.1. 安装 OpenLDAP

首先，安装 OpenLDAP：

安装 OpenLDAP 的示例 1

```
# cd /usr/ports/net/openldap26-server
# make install clean
```

这将安装 slapd 和 slurpd 二进制文件，以及所需的 OpenLDAP 库。

#### 2.1.2. 配置 OpenLDAP

接下来我们必须配置 OpenLDAP。

您会想要在与 LDAP 服务器的连接中要求加密；否则，您用户的密码将以明文传输，这被视为不安全。我们将要使用的工具支持两种非常相似的加密，即 SSL 和 TLS。

TLS 代表“传输层安全”。使用 TLS 的服务往往会与不支持 TLS 的相同服务在同一 ports 上连接；因此，支持 TLS 的 SMTP 服务器将侦听 25 号端口，LDAP 服务器将侦听 389 号端口。

SSL 代表“Secure Sockets Layer”，实现 SSL 的服务不会在 ports 上监听与非 SSL 对应项相同的端口。因此，SMTPS 监听 port 465（而不是 25），HTTPS 监听 443，LDAPS 监听 636。

SSL 使用不同的 port 与 TLS 的原因是 TLS 连接始于明文，而在 STARTTLS 指令之后切换到加密流量。SSL 连接从一开始就是加密的。除此之外，两者之间没有实质性的区别。

|     | 我们将调整 OpenLDAP 以使用 TLS，因为 SSL 被视为已弃用。 |
| --- | ------------------------------------------------------- |

一旦通过 ports 安装了 OpenLDAP，在/usr/local/etc/openldap/slapd.conf 中的以下配置参数将启用 TLS：

```
security ssf=128

TLSCertificateFile /path/to/your/cert.crt
TLSCertificateKeyFile /path/to/your/cert.key
TLSCACertificateFile /path/to/your/cacert.crt
```

这里， ssf=128 告诉 OpenLDAP 要求所有连接（包括搜索和更新）使用 128 位加密。这个参数可以根据您的站点安全需求进行配置，但很少需要弱化它，因为大多数 LDAP 客户端库都支持强加密。

cert.crt、cert.key 和 cacert.crt 文件对于客户端验证您是合法的 LDAP 服务器是必要的。如果您只是想要一个运行的服务器，可以使用 OpenSSL 创建一个自签名证书：

示例 2. 生成 RSA 密钥

```
% openssl genrsa -out cert.key 1024
Generating RSA private key, 1024 bit long modulus
....................++++++
...++++++
e is 65537 (0x10001)

% openssl req -new -key cert.key -out cert.csr
```

在这一点上，您应该被提示输入一些值。您可以输入任何您喜欢的值；然而，"通用名称" 值必须是 OpenLDAP 服务器的完全合格域名。在我们的情况下，以及这里的示例中，服务器是 server.example.org。不正确地设置这个值将导致客户端在建立连接时失败。这可能会导致很大的挫败感，所以请确保您密切遵循这些步骤。

最后，证书签名请求需要被签署:

示例 3. 自签名证书

```
% openssl x509 -req -in cert.csr -days 365 -signkey cert.key -out cert.crt
Signature ok
subject=/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd
Getting Private key
```

这将创建一个自签名证书，可用于 slapd.conf 中的指令，其中 cert.crt 和 cacert.crt 是同一个文件。如果您要使用许多 OpenLDAP 服务器（用于通过 slurpd 复制），您需要查看 LDAP 生成 CA 密钥并使用它来签署单独的服务器证书上的 OpenSSL 证书。

完成后，将以下内容放入 /etc/rc.conf：

```
slapd_enable="YES"
```

然后运行 /usr/local/etc/rc.d/slapd start 。这应该会启动 OpenLDAP。确认它正在监听 389

```
% sockstat -4 -p 389
ldap     slapd      3261  7  tcp4   *:389                 *:*
```

#### 2.1.3. 配置客户端

安装 net/openldap26-client port 以获取 OpenLDAP 库。客户端机器将始终拥有 OpenLDAP 库，因为目前 security/pam_ldap 和 net/nss_ldap 都只支持这些。

OpenLDAP 库的配置文件是/usr/local/etc/openldap/ldap.conf。编辑此文件以包含以下数值:

```
base dc=example,dc=org
uri ldap://server.example.org/
ssl start_tls
tls_cacert /path/to/your/cacert.crt
```

|     | 重要的是您的客户端能够访问 cacert.crt，否则他们将无法连接。 |
| --- | ----------------------------------------------------------- |

|     | 有两个名为 ldap.conf 的文件。第一个是此文件，用于 OpenLDAP 库，并定义如何与服务器通信。第二个是/usr/local/etc/ldap.conf，用于 pam_ldap。 |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------- |

此时，您应该可以在客户端计算机上运行 ldapsearch -Z ； -Z 表示“使用 TLS”。如果遇到错误，则说明某些配置错误；很可能是您的证书问题。使用 openssl(1)的 s_client 和 s_server 来确保您已正确配置和签署它们。

### 2.2. 数据库中的条目

针对 LDAP 目录的身份验证通常通过尝试以连接用户身份绑定到目录来完成。这是通过在目录上使用提供的用户名建立“简单”绑定来完成的。如果有一个条目，其 uid 等于用户名并且该条目的 userPassword 属性与提供的密码匹配，则绑定成功。

我们首先要做的事情是确定我们的用户将在目录中的哪个位置。

我们的数据库的基本条目是 dc=example,dc=org 。大多数客户端似乎期望用户的默认位置类似于 ou=people,<em>base</em> ，因此这里将使用该位置。但请记住，这是可配置的。

因此， people 组织单元的 ldif 条目将如下所示：

```
dn: ou=people,dc=example,dc=org
objectClass: top
objectClass: organizationalUnit
ou: people
```

所有用户将被创建为这个组织单位的子条目。

对于您的用户将属于的对象类，可能需要考虑一些想法。大多数工具默认会使用 people ，如果您只想提供用于身份验证的条目，则这样使用是可以的。然而，如果您还要在 LDAP 数据库中存储用户信息，您可能会想要使用 inetOrgPerson ，它有许多有用的属性。在任何情况下，相关的模式需要加载到 slapd.conf 中。

对于本例，我们将使用 person 对象类。如果您使用 inetOrgPerson ，步骤基本相同，只是需要 sn 属性。

要添加名为 tuser 的测试用户，ldif 将是：

```
dn: uid=tuser,ou=people,dc=example,dc=org
objectClass: person
objectClass: posixAccount
objectClass: shadowAccount
objectClass: top
uidNumber: 10000
gidNumber: 10000
homeDirectory: /home/tuser
loginShell: /bin/csh
uid: tuser
cn: tuser
```

我将我的 LDAP 用户的 UID 从 10000 开始，以避免与系统账户发生冲突；您可以在此处配置任何小于 65536 的数字。

我们还需要组条目。它们与用户条目一样可配置，但我们将使用以下默认值：

```
dn: ou=groups,dc=example,dc=org
objectClass: top
objectClass: organizationalUnit
ou: groups

dn: cn=tuser,ou=groups,dc=example,dc=org
objectClass: posixGroup
objectClass: top
gidNumber: 10000
cn: tuser
```

要将这些输入到您的数据库中，您可以使用 slapadd 或 ldapadd 在包含这些条目的文件上。或者，您可以使用 sysutils/ldapvi。

现在，客户端机器上的 ldapsearch 实用程序应返回这些条目。如果是这样，那么您的数据库已正确配置为用作 LDAP 认证服务器。

## 3. 客户端配置

客户端应该已经从配置客户端中获得了 OpenLDAP 库，但如果您要安装多个客户端机器，则需要在每台机器上安装 net/openldap26-client。

FreeBSD 需要安装两个 ports 来对 LDAP 服务器进行身份验证，分别是 security/pam_ldap 和 net/nss_ldap。

### 3.1. 认证

安全/pam_ldap 是通过 /usr/local/etc/ldap.conf 配置的。

|     | 这是一个不同于 OpenLDAP 库函数配置文件的文件，/usr/local/etc/openldap/ldap.conf；然而，它拥有许多相同的选项；实际上它是该文件的超集。在本节的其余部分中，ldap.conf 的引用将意味着 /usr/local/etc/ldap.conf。 |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

因此，我们将希望将我们原始的配置参数从 openldap/ldap.conf 复制到新的 ldap.conf。完成此操作后，我们想告诉安全/pam_ldap 在目录服务器上要查找什么。

我们正在使用 uid 属性来识别我们的用户。要配置这一点（尽管这是默认设置），请在 ldap.conf 中设置 pam_login_attribute 指令：

示例 4. 设置 pam_login_attribute

```
pam_login_attribute uid
```

设置了这个选项后，security/pam_ldap 将在 base 下的整个 LDAP 目录中搜索值为 uid=<em>username</em> 的条目。如果找到一个且仅有一个条目，并且成功用给定的密码绑定为该用户，则允许访问。否则将失败。

那些{{ 1001 }}不在/ etc / {{ 1002 }}中的用户将无法登录。当 Bash 设置为 LDAP 服务器上的用户{{ 1003 }}时，这一点尤为重要。Bash 未包含在 FreeBSD 的默认安装中。当从软件包或{{ 1004 }}安装时，它位于/ usr / local / bin / bash。验证服务器上的{{ 1005 }}路径是否设置正确：

```
% getent passwd username
```

当输出显示最后一列中有{{ 0 }}时，有两种选择。第一种是将用户在 LDAP 服务器上的条目更改为/ usr / local / bin / bash。第二个选项是在 LDAP 客户端计算机上创建符号链接，以便在正确位置找到 Bash：

```
# ln -s /usr/local/bin/bash /bin/bash
```

确保/ etc / {{ 1001 }}包含{{ 0 }}和{{ 1 }}的条目。然后，用户将能够使用 Bash 作为其{{ 1002 }}登录到系统。

#### 3.1.1. PAM

PAM，代表“可插拔认证模块”，是 FreeBSD 认证大部分会话的方法。要告诉 FreeBSD 我们希望使用 LDAP 服务器，我们需要在适当的 PAM 文件中添加一行。

大多数情况下，适当的 PAM 文件是 /etc/pam.d/sshd，如果你想使用 SSH（记得在 /etc/ssh/sshd_config 中设置相关选项，否则 SSH 将不使用 PAM）。

要使用 PAM 进行身份验证，请添加该行

```
auth  sufficient  /usr/local/lib/pam_ldap.so  no_warn
```

此行在文件中的确切位置以及第四列中出现的选项决定了身份验证机制的确切行为；请参阅 pam(d)

通过此配置，您应该能够对 LDAP 目录中的用户进行身份验证。PAM 将使用您的凭据执行绑定，如果成功，将告诉 SSH 允许访问。

但是允许目录中的每个用户登录到每台客户机并不是一个好主意。根据当前的配置，用户只需要一个 LDAP 条目就可以登录到一台机器。幸运的是，有一些方法可以限制用户访问。

ldap.conf 支持一个 pam_groupdn 指令；连接到这台机器的每个账户都需要是这里指定的组的成员。例如，如果你在 ldap.conf 中有

```
pam_groupdn cn=servername,ou=accessgroups,dc=example,dc=org
```

，那么只有该组的成员才能够登录。然而，有一些需要注意的事情。

该组的成员在一个或多个 memberUid 属性中指定，并且每个属性必须具有成员的完整区分名称。 因此 memberUid: someuser 不起作用；它必须是：

```
memberUid: uid=someuser,ou=people,dc=example,dc=org
```

另外，此指令在身份验证期间不会在 PAM 中进行检查，而是在帐户管理期间进行检查，因此您需要在 account 下的 PAM 文件中添加第二行。这将要求每个用户都列在组中，这不一定是我们想要的。 为了避免阻止未在 LDAP 中的用户，您应该启用 ignore_unknown_user 属性。 最后，您应该设置 ignore_authinfo_unavail 选项，以便在 LDAP 服务器不可用时不会被锁定在每台计算机上。

您的 pam.d / sshd 可能会变成这样：

示例 5. 样本 pam.d/sshd

```
auth            required        pam_nologin.so          no_warn
auth            sufficient      pam_opie.so             no_warn no_fake_prompts
auth            requisite       pam_opieaccess.so       no_warn allow_local
auth            sufficient      /usr/local/lib/pam_ldap.so      no_warn
auth            required        pam_unix.so             no_warn try_first_pass

account         required        pam_login_access.so
account         required        /usr/local/lib/pam_ldap.so      no_warn ignore_authinfo_unavail ignore_unknown_user
```

|     | 由于我们将这些行专门添加到 pam.d/sshd 中，这只会对 SSH 会话产生影响。LDAP 用户将无法在控制台登录。要更改此行为，请检查 /etc/pam.d 中的其他文件并相应地修改它们。 |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |

### 3.2. 名称服务开关

NSS 是将属性映射到名称的服务。因此，例如，如果文件的所有者是用户 1001 ，应用程序将查询 NSS 获取 1001 的名称，它可能是 bob 或 ted 或用户的实际名称。

现在我们的用户信息保存在 LDAP 中，当查询时，我们需要告诉 NSS 在那里查找。

net/nss_ldapport 就是这么做的。它使用与 security/pam_ldap 相同的配置文件，安装后不需要任何额外的参数。剩下的就是简单地编辑/etc/nsswitch.conf 以利用该目录。只需替换以下行：

```
group: compat
passwd: compat
```

与

```
group: files ldap
passwd: files ldap
```

这将允许您将用户名映射到 UID 和 UID 映射到用户名。

恭喜！您现在应该拥有可用的 LDAP 身份验证。

### 3.3. 注意事项

不幸的是，在撰写本文时，FreeBSD 不支持使用 passwd(1) 更改用户密码。因此，大多数管理员需要自己实现解决方案。我在这里提供一些示例。请注意，如果您编写自己的密码更改脚本，需要注意一些安全问题；请参阅密码存储。

示例 6. Shell 更改密码脚本

```
#!/bin/sh

stty -echo
read -p "Old Password: " oldp; echo
read -p "New Password: " np1; echo
read -p "Retype New Password: " np2; echo
stty echo

if [ "$np1" != "$np2" ]; then
  echo "Passwords do not match."
  exit 1
fi

ldappasswd -D uid="$USER",ou=people,dc=example,dc=org 
  -w "$oldp" 
  -a "$oldp" 
  -s "$np1"
```

```
# sysctl security.bsd.see_other_uids=0
```

可以通过编写自定义程序，甚至是 Web 界面，来使用更灵活（也可能更安全）的方法。以下是一个可以更改 LDAP 密码的 Ruby 库的一部分。它既可以在命令行上使用，也可以在 Web 上使用。

更改密码的 Ruby 脚本示例 7。

```
require 'ldap'
require 'base64'
require 'digest'
require 'password' # ruby-password

ldap_server = "ldap.example.org"
luser = "uid=#{ENV['USER']},ou=people,dc=example,dc=org"

# get the new password, check it, and create a salted hash from it
def get_password
  pwd1 = Password.get("New Password: ")
  pwd2 = Password.get("Retype New Password: ")

  raise if pwd1 != pwd2
  pwd1.check # check password strength

  salt = rand.to_s.gsub(/0./, '')
  pass = pwd1.to_s
  hash = "{SSHA}"+Base64.encode64(Digest::SHA1.digest("#{pass}#{salt}")+salt).chomp!
  return hash
end

oldp = Password.get("Old Password: ")
newp = get_password

# We'll just replace it.  That we can bind proves that we either know
# the old password or are an admin.

replace = LDAP::Mod.new(LDAP::LDAP_MOD_REPLACE | LDAP::LDAP_MOD_BVALUES,
                        "userPassword",
                        [newp])

conn = LDAP::SSLConn.new(ldap_server, 389, true)
conn.set_option(LDAP::LDAP_OPT_PROTOCOL_VERSION, 3)
conn.bind(luser, oldp)
conn.modify(luser, [replace])
```

虽然不能保证没有安全漏洞（例如，密码保存在内存中），但这比简单的 sh 脚本更加清晰和灵活。

## 4. 安全考虑

现在，您的计算机（可能还有其他服务）正在对 LDAP 服务器进行身份验证，因此至少需要像正常服务器上的 /etc/master.passwd 一样对此服务器进行保护，甚至可能需要更好，因为破解或被入侵的 LDAP 服务器会破坏每个客户端服务。

请记住，本节内容并不详尽。您应该持续审查您的配置和流程以进行改进。

### 4.1. 设置属性为只读

LDAP 中的几个属性应该是只读的。如果允许用户编写，例如，用户可以将他的 uidNumber 属性更改为 0 并获得 root 访问权限！

首先， userPassword 属性不应该是全局可读的。默认情况下，任何可以连接到 LDAP 服务器的人都可以读取此属性。要禁用这一点，请在 slapd.conf 中添加以下内容：

示例 8. 隐藏密码

```
access to dn.subtree="ou=people,dc=example,dc=org"
  attrs=userPassword
  by self write
  by anonymous auth
  by * none

access to *
  by self write
  by * read
```

这将禁止读取 userPassword 属性，同时允许用户更改他们自己的密码。

另外，您需要阻止用户更改他们自己某些属性。默认情况下，用户可以更改任何属性（除了 LDAP 模式本身禁止更改的属性），例如 uidNumber 。要关闭此漏洞，请修改上述内容为

只读属性例 9

```
access to dn.subtree="ou=people,dc=example,dc=org"
  attrs=userPassword
  by self write
  by anonymous auth
  by * none

access to attrs=homeDirectory,uidNumber,gidNumber
  by * read

access to *
  by self write
  by * read
```

这将阻止用户伪装成其他用户。

### 4.2. root 帐户定义

Often the `root` or manager account for the LDAP service will be defined in the configuration file. OpenLDAP supports this, for example, and it works, but it can lead to trouble if slapd.conf is compromised. It may be better to use this only to bootstrap yourself into LDAP, and then define a `root` account there.

Even better is to define accounts that have limited permissions, and omit a `root` account entirely. For example, users that can add or remove user accounts are added to one group, but they cannot themselves change the membership of this group. Such a security policy would help mitigate the effects of a leaked password.

#### 4.2.1. Creating a Management Group

假设您希望您的 IT 部门能够更改用户的主目录，但您不希望所有人都能够添加或删除用户。这样做的方法是为这些管理员添加一个组：

例如 10. 创建一个管理组

```
dn: cn=homemanagement,dc=example,dc=org
objectClass: top
objectClass: posixGroup
cn: homemanagement
gidNumber: 121 # required for posixGroup
memberUid: uid=tuser,ou=people,dc=example,dc=org
memberUid: uid=user2,ou=people,dc=example,dc=org
```

然后在 slapd.conf 中更改权限属性：

示例 11. 用于主目录管理组的 ACL

```
access to dn.subtree="ou=people,dc=example,dc=org"
  attr=homeDirectory
  by dn="cn=homemanagement,dc=example,dc=org"
  dnattr=memberUid write
```

现在 tuser 和 user2 可以更改其他用户的主目录。

在这个示例中，我们向某些用户赋予了管理权限的子集，而不是在其他领域赋予权限。想法是很快没有单一用户帐户拥有 root 帐户的权限，但每个权限根至少被一个用户拥有。然后 root 帐户变得不必要并可以被删除。

### 4.3. 密码存储

默认情况下，OpenLDAP 将以存储其他数据的方式存储 userPassword 属性的值：明文。大多数时候它是 base64 编码的，这提供了足够的保护来防止诚实的管理员知道您的密码，但除此之外没有什么别的保护。

因此，将密码存储为更安全的格式（例如 SSHA，盐化的 SHA）是一个好主意。这是通过您用来更改用户密码的任何程序完成的。

## 附录 A：有用的辅助工具

如果您有许多用户且不想手动配置所有内容，可能有一些其他程序很有用。

security/pam_mkhomedir 是一个始终成功的 PAM 模块；其目的是为那些还没有家目录的用户创建家目录。如果您有数十个客户端服务器和数百名用户，使用此方法并设置骨架目录要比准备每个家目录容易得多。

sysutils/ldapvi 是一个使用类似 LDIF 语法编辑 LDAP 值的实用工具。目录（或目录的子部分）以 EDITOR 环境变量选择的编辑器呈现。这使得在目录中进行大规模更改变得容易，而无需编写定制工具。

security/openssh-portable 具有联系 LDAP 服务器以验证 SSH 密钥的能力。如果您有许多服务器且不想在所有服务器上复制您的公钥，这非常方便。

## 附录 B：LDAP 的 OpenSSL 证书

如果你托管两个或更多的 LDAP 服务器，你可能不想使用自签名证书，因为每个客户端都必须配置为使用每个证书。虽然这是可能的，但它远不如创建你自己的证书颁发机构，并用它签署你的服务器证书那么简单。

这里的步骤几乎没有解释正在发生什么——可以在 openssl(1) 和相关工具中找到进一步的解释。

要创建一个证书颁发机构，我们只需要一个自签名证书和密钥。步骤如下

创建证书示例 12。

```
% openssl genrsa -out root.key 1024
% openssl req -new -key root.key -out root.csr
% openssl x509 -req -days 1024 -in root.csr -signkey root.key -out root.crt
```

这将是您的根 CA 密钥和证书。您可能希望加密密钥并将其存储在一个阴凉而干燥的地方；任何能访问到它的人都可以冒充您的 LDAP 服务器之一。

接下来，使用上述的头两个步骤创建一个名为 ldap-server-one.key 的密钥和一个名为 ldap-server-one.csr 的证书签名请求。一旦您使用 root.key 签署签名请求，您将能够在您的 LDAP 服务器上使用 ldap-server-one.*。

|     | 不要忘记在生成证书签名请求时使用完全合格的域名作为“通用名称”属性；否则客户端将拒绝与您建立连接，并且诊断可能非常棘手。 |
| --- | ---------------------------------------------------------------------------------------------------------------------- |

要签署该密钥，请使用 -CA 和 -CAkey ，而不是 -signkey ：

示例 13. 作为证书颁发机构签署

```
% openssl x509 -req -days 1024 
-in ldap-server-one.csr -CA root.crt -CAkey root.key 
-out ldap-server-one.crt
```

生成的文件将是您可以在 LDAP 服务器上使用的证书。

最后，为了使客户端信任您所有的服务器，请将 root.crt（证书，而非密钥！）分发给每个客户端，并在 ldap.conf 中的 TLSCACertificateFile 指令中指定它。
