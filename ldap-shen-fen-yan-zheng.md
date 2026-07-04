# LDAP 身份验证

- 原文：[LDAP Authentication](https://docs.freebsd.org/en/articles/ldap-auth/)


## 摘要

本文旨在为在 FreeBSD 上配置用于身份验证的 LDAP 服务器（主要是 OpenLDAP 服务器）提供指南。这在需要多个服务器使用相同用户账户的场景下非常有用，例如替代 NIS 的情况。

## 1. 前言

本文旨在帮助读者获得足够的 LDAP 知识，从而能够配置一台 LDAP 服务器。本文还将尝试解释如何使用 [net/nss_ldap](https://cgit.freebsd.org/ports/tree/net/nss_ldap/) 与 [security/pam_ldap](https://cgit.freebsd.org/ports/tree/security/pam_ldap/) 配置客户端服务，使其与 LDAP 服务器协同工作。

阅读完毕后，读者应能够配置并部署一台可以托管 LDAP 目录的 FreeBSD 服务器，并能够配置并部署一台可以通过 LDAP 目录进行身份验证的 FreeBSD 服务器。

本文并不旨在详尽讨论配置 LDAP 或其它相关服务时的安全性、稳健性或最佳实践。尽管作者尽力保持内容正确，但安全性问题仅在一般范围内予以处理。本文应视为理论基础，任何实际部署都应伴随仔细的需求分析。

## 2. 配置 LDAP

LDAP 是“轻量目录访问协议”（Lightweight Directory Access Protocol）的缩写，是 X.500 目录访问协议的子集。其最新规范见于 [RFC4510](http://www.ietf.org/rfc/rfc4510.txt) 及相关文档。它本质上是数据库，其主要用途是读取而非写入。

本文所使用的 LDAP 服务器为 [OpenLDAP](http://www.openldap.org/)。尽管本文的原理部分适用于多种服务器，但具体的管理操作是特定于 OpenLDAP 的。Ports 中提供多个版本的服务器，例如 [net/openldap26-server](https://cgit.freebsd.org/ports/tree/net/openldap26-server/)。客户端服务器需要安装对应的 [net/openldap26-client](https://cgit.freebsd.org/ports/tree/net/openldap26-client/) 库。

LDAP 服务基本上有两个方面需要配置：第一是设置服务器以正确接收连接，第二是向服务器的目录添加条目，以便 FreeBSD 工具能够与之交互。

### 2.1. 设置服务器以接收连接

>**注意**
>
>本节内容特定于 OpenLDAP。如果使用其它服务器，请查阅相应文档。

#### 2.1.1. 安装 OpenLDAP

首先，安装 OpenLDAP：

**示例 1. 安装 OpenLDAP**

```sh
# cd /usr/ports/net/openldap26-server
# make install clean
```

这将安装 `slapd` 与 `slurpd` 可执行文件，以及所需的 OpenLDAP 库。

#### 2.1.2. 配置 OpenLDAP

接下来我们需要配置 OpenLDAP。

你应当要求 LDAP 连接使用加密，否则用户的密码将以明文传输，这是不安全的。我们将使用的工具支持两种非常相似的加密方式：SSL 与 TLS。

TLS 是“传输层安全协议”（Transportation Layer Security）的缩写。使用 TLS 的服务通常监听与非加密服务相同的端口；例如支持 TLS 的 SMTP 服务监听 25 端口，LDAP 服务监听 389 端口。

SSL 是“安全套接字层”（Secure Sockets Layer）的缩写，使用 SSL 的服务不会与其非加密版本监听相同端口。例如 SMTPS 使用 465（而非 25），HTTPS 使用 443，LDAPS 使用 636。

SSL 与 TLS 使用不同端口的原因在于：TLS 连接以明文开始，在接收到 `STARTTLS` 指令后切换为加密；而 SSL 连接从一开始就是加密的。除此之外，两者并无本质区别。

>**注意**
>
>我们将配置 OpenLDAP 使用 TLS，因为 SSL 已被弃用。

OpenLDAP 安装完成后，在 **/usr/local/etc/openldap/slapd.conf** 中添加以下配置参数以启用 TLS：

```ini
security ssf=128

TLSCertificateFile /path/to/your/cert.crt
TLSCertificateKeyFile /path/to/your/cert.key
TLSCACertificateFile /path/to/your/cacert.crt
```

其中，`ssf=128` 表示 OpenLDAP 要求所有连接（包括查询与更新）使用 128 位加密。该参数可根据站点的安全需求调整，但通常无需降低强度，因为大多数 LDAP 客户端库都支持强加密。

**cert.crt**、**cert.key** 与 **cacert.crt** 文件是为了让客户端能够认证 *你* 是合法的 LDAP 服务器。如果你仅仅想要一个能运行的服务器，可以使用 OpenSSL 创建自签名证书：

**示例 2. 生成 RSA 密钥**

```sh
% openssl genrsa -out cert.key 1024
Generating RSA private key, 1024 bit long modulus
....................++++++
...++++++
e is 65537 (0x10001)

% openssl req -new -key cert.key -out cert.csr
```

此时会提示你输入一些值。可以输入任意内容，但“Common Name” 字段必须填写 OpenLDAP 服务器的完整主机名。在本文与示例中，该服务器为 *server.example.org*。此值设置错误会导致客户端连接失败，常成为令人沮丧的问题源头，因此务必仔细确认。

最后，需要对证书请求进行签名：

**示例 3. 自签证书**

```sh
% openssl x509 -req -in cert.csr -days 365 -signkey cert.key -out cert.crt
Signature ok
subject=/C=AU/ST=Some-State/O=Internet Widgits Pty Ltd
Getting Private key
```

这将创建可用于 **slapd.conf** 中配置项的自签名证书，其中 **cert.crt** 与 **cacert.crt** 可为同一文件。如果你计划使用多个 OpenLDAP 服务器（例如通过 `slurpd` 进行复制），请参阅 [LDAP 中的 OpenSSL 证书](https://docs.freebsd.org/en/articles/ldap-auth/#ssl-ca) 以生成 CA 密钥，并使用该密钥为每台服务器签发证书。

完成后，在 **/etc/rc.conf** 中加入以下内容：

```sh
slapd_enable="YES"
```

然后运行 `/usr/local/etc/rc.d/slapd start`。这将启动 OpenLDAP。使用以下命令确认其监听 389 端口：

```sh
% sockstat -4 -p 389
ldap     slapd      3261  7  tcp4   *:389                 *:*
```

#### 2.1.3. 配置客户端

为 OpenLDAP 库安装 [net/openldap26-client](https://cgit.freebsd.org/ports/tree/net/openldap26-client/) Port。客户端始终会使用 OpenLDAP 库，因为目前 [security/pam_ldap](https://cgit.freebsd.org/ports/tree/security/pam_ldap/) 与 [net/nss_ldap](https://cgit.freebsd.org/ports/tree/net/nss_ldap/) 只支持 OpenLDAP。

OpenLDAP 库的配置文件为 **/usr/local/etc/openldap/ldap.conf**。编辑该文件，加入如下内容：

```sh
base dc=example,dc=org
uri ldap://server.example.org/
ssl start_tls
tls_cacert /path/to/your/cacert.crt
```

>**注意**
>
>客户端必须能够访问 **cacert.crt** 文件，否则将无法建立连接。

>**注意**
>
>系统中有两个名为 **ldap.conf** 的文件。第一个是上述用于 OpenLDAP 库的文件，用于定义如何连接服务器；第二个是 **/usr/local/etc/ldap.conf**，用于 `pam_ldap`。

此时，你应可在客户端运行 `ldapsearch -Z`；其中 `-Z` 表示“使用 TLS”。若遇到错误，说明配置存在问题，最常见的是证书问题。可使用 [openssl(1)](https://man.freebsd.org/cgi/man.cgi?query=openssl&sektion=1&format=html) 的 `s_client` 与 `s_server` 工具确认证书是否配置正确并已签名。


### 2.2. 数据库中的条目

对 LDAP 目录的认证通常是通过尝试以连接用户的身份绑定到目录来完成的。这是通过在目录上建立“简单”绑定并提供用户名来实现的。如果存在条目的 `uid` 等于该用户名，并且该条目的 `userPassword` 属性与提供的密码匹配，那么绑定就会成功。

我们首先需要弄清楚用户在目录中的位置。

我们数据库的基本条目是 `dc=example,dc=org`。大多数客户端默认期望用户位于 `ou=people,base` 这样的路径下，因此我们也采用这个结构。不过要注意，这一点是可以配置的。

因此，`people` 这个组织单位的 ldif 条目应如下所示：

```sh
dn: ou=people,dc=example,dc=org
objectClass: top
objectClass: organizationalUnit
ou: people
```

所有用户都将作为这个组织单位的子条目被创建。

你需要考虑你的用户将属于哪个对象类。大多数工具默认使用 `person`，如果你只想提供认证条目，这样就足够了。但如果你还想在 LDAP 数据库中存储用户信息，那么你很可能需要使用 `inetOrgPerson`，它包含许多有用的属性。无论选择哪一种，都需要在 **slapd.conf** 中加载相关的 schema。

在本例中我们使用 `person` 对象类。如果你使用 `inetOrgPerson`，步骤基本相同，但需要额外提供 `sn` 属性。

为了添加名为 `tuser` 的测试用户，ldif 文件应如下：

```sh
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

我从 10000 开始为 LDAP 用户分配 UID，以避免与系统账户冲突；你可以根据需要配置任何小于 65536 的数字。

我们还需要创建组条目。它们和用户条目一样可以配置，但我们这里使用默认配置如下：

```sh
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

你可以使用 `slapadd` 或 `ldapadd` 将这些条目所在的文件导入数据库。你也可以使用 [sysutils/ldapvi](https://cgit.freebsd.org/ports/tree/sysutils/ldapvi/) 工具。

客户端上的 `ldapsearch` 工具现在应该可以返回这些条目。如果可以，说明你的数据库已经正确配置，可以作为 LDAP 认证服务器使用。



## 3. 客户端配置

客户端应该已经从 [配置客户端](https://docs.freebsd.org/en/articles/ldap-auth/#ldap-connect-client) 中安装了 OpenLDAP 库，但如果你要配置多台客户端机器，那么每台机器都需要安装 [net/openldap26-client](https://cgit.freebsd.org/ports/tree/net/openldap26-client/)。

FreeBSD 需要安装两个 Port 才能对 LDAP 服务器进行认证：[security/pam_ldap](https://cgit.freebsd.org/ports/tree/security/pam_ldap/) 和 [net/nss_ldap](https://cgit.freebsd.org/ports/tree/net/nss_ldap/)。

### 3.1. 认证

[security/pam_ldap](https://cgit.freebsd.org/ports/tree/security/pam_ldap/) 通过 **/usr/local/etc/ldap.conf** 文件进行配置。

>**注意**
>
>这个文件与 OpenLDAP 库函数的配置文件 **/usr/local/etc/openldap/ldap.conf** 是 *不同的文件*；不过它接受许多相同的选项；事实上，它是后者的超集。下文中提到的 **ldap.conf** 均指 **/usr/local/etc/ldap.conf**。

因此，我们需要将原先配置文件 **openldap/ldap.conf** 中的所有参数复制到新的 **ldap.conf** 中。完成此操作后，我们需要告知 [security/pam_ldap](https://cgit.freebsd.org/ports/tree/security/pam_ldap/) 要在目录服务器上查找哪些内容。

我们通过 `uid` 属性来识别用户。要配置这一点（尽管它是默认值），请在 **ldap.conf** 中设置 `pam_login_attribute` 指令：

**示例 4. 设置 `pam_login_attribute`**

```sh
pam_login_attribute uid
```

设置之后，[security/pam_ldap](https://cgit.freebsd.org/ports/tree/security/pam_ldap/) 将在 `base` 所指定的整个 LDAP 目录中搜索 `uid=username` 这一值。如果只找到一个匹配条目，它将尝试以该用户绑定，并使用提供的密码进行验证。若绑定成功，则允许访问；否则验证失败。

如果用户的 shell 不在 **/etc/shells** 中，将无法登录。这一点在 LDAP 服务器为用户设置 Bash shell 时尤为重要。FreeBSD 的默认安装并不包含 Bash。当通过包或 Port 安装 Bash 时，它位于 **/usr/local/bin/bash**。需要确认服务器上的 shell 路径设置正确：

```sh
% getent passwd username
```

如果输出的最后一列显示为 `/bin/bash`，有两种解决方式：一种是将 LDAP 服务器上的用户条目改为 **/usr/local/bin/bash**；另一种是在 LDAP 客户端机器上创建符号链接：

```sh
# ln -s /usr/local/bin/bash /bin/bash
```

同时请确保 **/etc/shells** 包含 `/usr/local/bin/bash` 和 `/bin/bash` 两个条目。这样用户就可以使用 Bash 作为 shell 登录系统。


#### 3.1.1. PAM

PAM 是“可插拔认证模块”（Pluggable Authentication Modules）的缩写，是 FreeBSD 用于认证大多数会话的机制。要告诉 FreeBSD 我们希望使用 LDAP 服务器，需要向合适的 PAM 文件添加一行配置。

通常情况下，如果你希望通过 SSH 使用 LDAP 认证，则应修改 **/etc/pam.d/sshd** 文件（同时记得在 **/etc/ssh/sshd_config** 中设置相关选项，否则 SSH 不会使用 PAM）。

要启用 PAM 认证，请添加如下行：

```sh
auth  sufficient  /usr/local/lib/pam_ldap.so  no_warn
```

该行在文件中的具体位置及第四列中所使用的选项决定了认证机制的具体行为；参见 [pam(d)](https://man.freebsd.org/cgi/man.cgi?query=pam&sektion=d&format=html)。

通过此配置，你应能成功通过 LDAP 目录认证用户。PAM 会使用你的凭据尝试绑定，如果成功，则允许 SSH 访问。

然而，允许 *每个* 目录中的用户登录 *每台* 客户端机器并不是个好主意。按当前配置，只要用户在 LDAP 中有条目，就能登录任意机器。幸运的是，有几种方法可以限制用户访问。

**ldap.conf** 支持 `pam_groupdn` 指令；每个连接到该机器的账户都必须属于指定的组。例如，如果你写入以下内容：

```sh
pam_groupdn cn=servername,ou=accessgroups,dc=example,dc=org
```

到 **ldap.conf** 中，那么只有该组的成员才能登录。不过需要注意以下几点：

该组的成员由一个或多个 `memberUid` 属性指定，而且每个属性必须为成员的完整可分辨名称（DN）。因此 `memberUid: someuser` 是无效的，必须写为：

```sh
memberUid: uid=someuser,ou=people,dc=example,dc=org
```

此外，该指令并不会在认证阶段（auth）检查，而是在账号管理阶段（account）检查，因此你还需要在 PAM 文件中添加一行 `account` 配置。这将导致 *每个* 用户都必须被列入该组，这并非总是我们想要的行为。为了避免阻止不在 LDAP 中的用户，应启用 `ignore_unknown_user` 选项。最后，建议设置 `ignore_authinfo_unavail`，以防 LDAP 服务器不可用时导致无法登录所有机器。

你的 **pam.d/sshd** 文件可能最终如下所示：

**示例 5. pam.d/sshd 示例**

```ini
auth            required        pam_nologin.so          no_warn
auth            sufficient      pam_opie.so             no_warn no_fake_prompts
auth            requisite       pam_opieaccess.so       no_warn allow_local
auth            sufficient      /usr/local/lib/pam_ldap.so      no_warn
auth            required        pam_unix.so             no_warn try_first_pass

account         required        pam_login_access.so
account         required        /usr/local/lib/pam_ldap.so      no_warn ignore_authinfo_unavail ignore_unknown_user
```

>**注意**
>
>因为我们是专门向 **pam.d/sshd** 添加这些行，因此这些配置只对 SSH 会话生效。LDAP 用户将无法在控制台登录。要改变这一行为，请检查 **/etc/pam.d** 中的其他文件并进行相应修改。


### 3.2. 名称服务切换（Name Service Switch）

NSS 是将属性映射为名称的服务。例如，如果文件归属用户 `1001`，某个应用程序就会通过 NSS 查询 `1001` 对应的用户名，可能返回的是 `bob`、`ted` 或者其他名称。

现在我们的用户信息存储在 LDAP 中了，因此我们需要告诉 NSS，在被查询时应从那里查找。

[net/nss_ldap](https://cgit.freebsd.org/ports/tree/net/nss_ldap/) Port 提供了这个功能。它使用与 [security/pam_ldap](https://cgit.freebsd.org/ports/tree/security/pam_ldap/) 相同的配置文件，安装后通常不需要额外的参数。接下来我们只需编辑 **/etc/nsswitch.conf**，以启用目录服务支持。只需将如下几行：

```ini
group: compat
passwd: compat
```

替换为：

```ini
group: files ldap
passwd: files ldap
```

这将使你能够将用户名映射到 UID，或将 UID 映射为用户名。

恭喜！你现在应该拥有了可用的 LDAP 认证功能。


### 3.3. 注意事项（Caveats）

不幸的是，截至本文写作时，FreeBSD 尚不支持通过 [passwd(1)](https://man.freebsd.org/cgi/man.cgi?query=passwd&sektion=1&format=html) 命令更改用户密码。因此，多数管理员需要自己实现解决方案。下面提供了一些示例。注意：如果你打算自行编写密码更改脚本，有一些安全问题需要注意；请参阅 [Password Storage](https://docs.freebsd.org/en/articles/ldap-auth/#security-passwd)。

**示例 6. 用于更改密码的 Shell 脚本**

```sh
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

ldappasswd -D uid="$USER",ou=people,dc=example,dc=org \
  -w "$oldp" \
  -a "$oldp" \
  -s "$np1"
```

>**小心**
>
>该脚本几乎不进行任何错误检查，更重要的是它对密码的存储非常草率。如果你确实打算使用此类方法，至少应调整 `security.bsd.see_other_uids` 这个 sysctl 参数：
>
>```sh
># sysctl security.bsd.see_other_uids=0
>```

一种更灵活（也可能更安全）的方法是编写自定义程序，甚至是 Web 接口。以下是 Ruby 库的部分内容，它可以更改 LDAP 密码，既可用于命令行，也可用于 Web。

**示例 7. 用于更改密码的 Ruby 脚本**

```ruby
require 'ldap'
require 'base64'
require 'digest'
require 'password' # ruby-password

ldap_server = "ldap.example.org"
luser = "uid=#{ENV['USER']},ou=people,dc=example,dc=org"

# 获取新密码、进行检查并生成带盐的哈希值
def get_password
  pwd1 = Password.get("New Password: ")
  pwd2 = Password.get("Retype New Password: ")

  raise if pwd1 != pwd2
  pwd1.check # 检查密码强度

  salt = rand.to_s.gsub(/0\./, '')
  pass = pwd1.to_s
  hash = "{SSHA}"+Base64.encode64(Digest::SHA1.digest("#{pass}#{salt}")+salt).chomp!
  return hash
end

oldp = Password.get("Old Password: ")
newp = get_password

# 我们将直接替换旧密码。能够成功绑定说明我们要么知道旧密码，要么有管理员权限。

replace = LDAP::Mod.new(LDAP::LDAP_MOD_REPLACE | LDAP::LDAP_MOD_BVALUES,
                        "userPassword",
                        [newp])

conn = LDAP::SSLConn.new(ldap_server, 389, true)
conn.set_option(LDAP::LDAP_OPT_PROTOCOL_VERSION, 3)
conn.bind(luser, oldp)
conn.modify(luser, [replace])
```

尽管它不能保证绝对安全（比如密码仍保存在内存中），但相较于简单的 `sh` 脚本，这种方法更清晰、也更灵活。

## 4. 安全注意事项（Security Considerations）

现在你的机器（甚至可能包括其他服务）已经开始依赖 LDAP 服务器进行身份验证，这台服务器的安全性至少应与常规服务器上的 **/etc/master.passwd** 文件持平，甚至更高。因为一旦 LDAP 服务器被破坏或攻破，所有客户端服务也将随之瘫痪。

请注意，本节并不穷尽所有内容。你应持续审查自己的配置和操作流程，以便改进。

---

### 4.1. 设置只读属性（Setting Attributes Read-only）

LDAP 中的多个属性应设为只读。如果允许用户自行写入，比如用户可以将其 `uidNumber` 改为 `0`，这将获得 `root` 权限！

首先，`userPassword` 属性不应向任何人公开读取。默认情况下，任何能连接 LDAP 服务器的人都可以读取此属性。为了禁止这一点，请在 **slapd.conf** 中添加如下内容：

**示例 8. 隐藏密码**

```ini
access to dn.subtree="ou=people,dc=example,dc=org"
  attrs=userPassword
  by self write
  by anonymous auth
  by * none

access to *
  by self write
  by * read
```

这将禁止读取 `userPassword` 属性，但仍允许用户更改自己的密码。

此外，你还需要防止用户更改某些自身的属性。默认情况下，用户可以修改任何属性（除非 LDAP 架构自身禁止修改），例如 `uidNumber`。为堵住这一安全漏洞，可以将上述配置修改为：

**示例 9. 设为只读的属性**

```ini
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

这样就能阻止用户冒充其他用户。


### 4.2. `root` 账户定义

LDAP 服务的 `root` 或管理账户通常会直接写在配置文件中。以 OpenLDAP 为例，它支持这种做法，也确实可行，但如果 **slapd.conf** 被泄露，就会带来麻烦。更好的做法是只在最初引导 LDAP 系统时使用这个账户，之后在 LDAP 数据库中创建 `root` 账户。

进一步的改进是完全不使用 `root` 账户，而是创建权限受限的账户。例如：被授权添加或删除用户账户的用户属于一个特定的组，但他们自己却不能修改该组的成员。这类安全策略有助于降低密码泄露带来的风险。

#### 4.2.1. 创建管理组（Creating a Management Group）

假设你希望 IT 部门的成员能够修改用户的 home 目录，但不希望他们所有人都能够添加或删除用户。实现这一目标的方法是为这些管理员添加组：

**示例 10. 创建管理组**

```sh
dn: cn=homemanagement,dc=example,dc=org
objectClass: top
objectClass: posixGroup
cn: homemanagement
gidNumber: 121 # posixGroup 所需
memberUid: uid=tuser,ou=people,dc=example,dc=org
memberUid: uid=user2,ou=people,dc=example,dc=org
```

然后在 **slapd.conf** 中更改权限属性：

**示例 11. home 目录管理组的 ACL**

```sh
access to dn.subtree="ou=people,dc=example,dc=org"
  attr=homeDirectory
  by dn="cn=homemanagement,dc=example,dc=org"
  dnattr=memberUid write
```

现在，`tuser` 和 `user2` 就可以更改其他用户的 home 目录了。

在这个例子中，我们将部分管理权限赋予某些用户，而没有授予他们在其他领域的权限。其思想是：最终不再有单个用户账户拥有 `root` 账户的权限，但 `root` 曾拥有的每一项权限，都至少有一个用户拥有。如此一来，`root` 账户就变得不再必要，可以移除。



### 4.3. 密码存储（Password Storage）

默认情况下，OpenLDAP 会将 `userPassword` 属性的值以与其他数据相同的方式存储：明文存储。大多数时候，它只是被 base64 编码，这种保护措施仅仅足以让诚实的管理员看不到你的密码，而几乎无法抵御其他威胁。

因此，更好的做法是将密码以更安全的格式存储，比如 SSHA（加盐的 SHA）。这通常由你用来修改用户密码的程序负责完成。



## 附录 A：有用工具（Useful Aids）

有一些额外的程序可能会对你有所帮助，尤其当你拥有大量用户而又不希望手动配置一切时。

[security/pam_mkhomedir](https://cgit.freebsd.org/ports/tree/security/pam_mkhomedir/) 是一个 PAM 模块，它总是会“成功”；它的作用是为没有 home 目录的用户自动创建 home 目录。如果你拥有几十台客户端服务器和上百名用户，使用它并设置 skeleton 目录将远比为每个用户手动准备 home 目录来得容易。

[sysutils/ldapvi](https://cgit.freebsd.org/ports/tree/sysutils/ldapvi/) 是一个非常优秀的工具，它允许你用类 LDIF 的语法编辑 LDAP 条目。目录（或其中某一子集）会以你通过 `EDITOR` 环境变量指定的编辑器打开。这让你无需编写自定义工具，也能轻松批量修改目录内容。

[security/openssh-portable](https://cgit.freebsd.org/ports/tree/security/openssh-portable/) 支持通过 LDAP 服务器验证 SSH 密钥。如果你管理着许多服务器，并希望避免将公钥复制到所有机器上，这个功能非常有用。

## 附录 B：用于 LDAP 的 OpenSSL 证书（OpenSSL Certificates for LDAP）

如果你在托管两个或更多的 LDAP 服务器，你很可能不希望使用自签名证书，因为这会要求每个客户端都为每个证书进行配置。虽然这并非不可能，但远不如创建你自己的证书颁发机构（CA）并使用它为你的服务器证书签名来得简单。

以下操作步骤将直接展示过程，不会详细解释其原理——你可以参考 [openssl(1)](https://man.freebsd.org/cgi/man.cgi?query=openssl&sektion=1&format=html) 及相关文档以获得进一步解释。

要创建证书颁发机构，我们只需要自签名证书和对应密钥。具体步骤如下：

**示例 12. 创建证书**

```sh
% openssl genrsa -out root.key 1024
% openssl req -new -key root.key -out root.csr
% openssl x509 -req -days 1024 -in root.csr -signkey root.key -out root.crt
```

这将生成你的根 CA 密钥和证书。你应该对该密钥进行加密，并将其保存在阴凉干燥的安全地点；任何获得该密钥的人都可以伪装成你的 LDAP 服务器。

接下来，使用上述前两步创建密钥 **ldap-server-one.key** 和证书签名请求 **ldap-server-one.csr**。使用 **root.key** 对签名请求进行签名后，你就可以在 LDAP 服务器上使用 `ldap-server-one.*` 文件了。

>**注意**
>
>生成证书签名请求时，不要忘记将“Common Name”属性设置为完全限定域名（FQDN）；否则客户端会拒绝与你的连接，而且诊断这种问题非常困难。

要对密钥进行签名，请使用 `-CA` 和 `-CAkey` 参数，而不是 `-signkey`：

**示例 13. 以证书颁发机构身份签名**

```sh
% openssl x509 -req -days 1024 \
-in ldap-server-one.csr -CA root.crt -CAkey root.key \
-out ldap-server-one.crt
```

生成的文件将是你可以在 LDAP 服务器上使用的证书。

最后，为了让客户端信任你的所有服务器，请将 **root.crt**（注意是 *证书*，不是密钥！）分发给每个客户端，并在 **ldap.conf** 文件中通过 `TLSCACertificateFile` 指令指定它。
