# 桥接过滤器

<details open="" data-immersive-translate-walked="d632301c-1d24-4ef7-9c3c-c2b69fd8947e"><summary data-immersive-translate-walked="d632301c-1d24-4ef7-9c3c-c2b69fd8947e" data-immersive-translate-paragraph="1"><font class="notranslate immersive-translate-target-wrapper" data-immersive-translate-translation-element-mark="1" lang="zh-CN"><font class="notranslate" data-immersive-translate-translation-element-mark="1"> </font><font class="notranslate immersive-translate-target-translation-theme-none immersive-translate-target-translation-inline-wrapper-theme-none immersive-translate-target-translation-inline-wrapper" data-immersive-translate-translation-element-mark="1"><font class="notranslate immersive-translate-target-inner immersive-translate-target-translation-theme-none-inner" data-immersive-translate-translation-element-mark="1">商标</font></font></font></summary>

FreeBSD 是 FreeBSD 基金会的注册商标。

3Com 和 HomeConnect 是 3Com 公司的注册商标。

Intel、Celeron、Centrino、Core、EtherExpress、i386、i486、Itanium、Pentium 和 Xeon 是 Intel 公司或其子公司在美国和其他国家的商标或注册商标。

许多制造商和卖家用来区分其产品的名称被声明为商标。如果这些名称出现在本文档中，并且 FreeBSD 项目知道该商标声明，这些名称后面已加上 “™” 或 “®” 符号。

</details>

 摘要

通常情况下，将一个物理网络（如以太网）划分为两个独立段而无需创建子网，然后使用路由器将它们连接起来是很有用的。以这种方式连接两个网络的设备称为桥接器。拥有两个网络接口的 FreeBSD 系统足以充当桥接器。

桥接器通过扫描连接到其每个网络接口的设备的 MAC 层（以太网地址）地址，然后仅在源和目的地位于不同段时才转发两个网络之间的流量。从许多方面看，桥接器类似于一个只有两个端口的以太网交换机。

---

## 1. 为什么要使用过滤式桥接器？

越来越频繁地，多亏了宽带互联网连接成本的降低（xDSL），以及可用 IPv4 地址的减少，许多公司全天候连接到互联网，并且只有很少的 IP 地址（有时甚至不是 2 的幂）。在这些情况下，通常希望有一个防火墙来过滤从互联网进出的流量，但基于路由器的数据包过滤解决方案可能不适用，要么是由于子网划分问题，要么是路由器是连接供应商（ISP）所有的，要么是因为它不支持这些功能。在这些情况下，强烈建议使用过滤桥。

基于桥接的防火墙可以在 xDSL 路由器和您的以太网集线器/交换机之间进行配置和插入，而无需任何 IP 编号问题。

## 2. 安装方法

将桥接功能添加到 FreeBSD 系统中并不困难。自 4.5 版本以来，可以将这些功能作为模块加载，而无需重新构建内核，极大地简化了过程。在接下来的小节中，我将解释两种安装方式。

>不要同时遵循两套说明：一种过程排斥另一种。根据您的需求和能力选择最佳选项。

在继续之前，请确保至少有两张支持混杂模式的以太网卡，能够在接收和发送时发送任何地址的以太网数据包，而不仅仅是它们自己的地址。此外，为了获得良好的吞吐量，网卡应为 PCI 总线主控卡。最佳选择仍然是 Intel EtherExpress™ Pro，其次是 3Com® 3c9xx 系列。为了简化防火墙配置，拥有两张不同制造商的卡（使用不同驱动程序）可能很有用，以清楚区分哪个接口连接到路由器，哪个连接到内部网络。

### 2.1. 内核配置

所以您已经决定使用较旧但经过良好测试的安装方法。首先，您必须将以下行添加到内核配置文件中：

```
options BRIDGE
options IPFIREWALL
options IPFIREWALL_VERBOSE
```

第一行是用于编译桥接支持，第二行是用于防火墙，第三行是用于防火墙的日志功能。

现在需要构建并安装新内核。您可以在 FreeBSD 手册的构建和安装自定义内核部分找到详细的说明。

### 2.2. 模块加载

如果您选择使用新的更简单的安装方法，现在唯一要做的就是将以下行添加到 /boot/loader.conf 文件中：

```
bridge_load="YES"
```

在系统启动过程中，桥接模块 bridge.ko 将与内核一起加载。不需要为 ipfw.ko 模块添加类似的行，因为在执行以下部分中的步骤后，它将自动加载。

## 3. 最终准备

在重新启动以加载新内核或所需模块之前（根据先前选择的安装方法），您需要对 /etc/rc.conf 配置文件进行一些更改。防火墙的默认规则是拒绝所有 IP 数据包。最初，我们将设置一个 open 防火墙，以验证其在不涉及数据包过滤方面的问题下的操作（如果您打算远程执行此过程，则此配置将避免您与网络隔离）。请将以下行放入 /etc/rc.conf：

```
firewall_enable="YES"
firewall_type="open"
firewall_quiet="YES"
firewall_logging="YES"
```

第一行将启用防火墙（如果未在内核中编译，则会加载模块 ipfw.ko），第二行将在 open 模式下设置它（如/etc/rc.firewall 中所述），第三行将不显示规则加载，第四行将启用日志支持。

关于网络接口的配置，最常用的方式是为其中一个网络卡指定一个 IP，但即使两个接口都没有配置 IP，桥也会正常工作。在最后一种情况（无 IP）中，桥机仍将更隐匿，无法从网络访问：要对其进行配置，必须从控制台登录或通过与桥隔离的第三个网络接口进行登录。有时，在系统启动过程中，有些程序需要网络访问，比如域名解析：在这种情况下，需要为外部接口（连接到互联网的接口，DNS 服务器所在的位置）分配一个 IP，因为桥将在启动过程结束时激活。这意味着在/etc/rc.conf 文件的 ifconfig 部分中必须提到 fxp0 接口（在我们的案例中），而不是 xl0 接口。为两个网络卡分配 IP 并没有太多意义，除非在启动过程中，应用程序需要访问两个以太网段上的服务。

还有一件重要的事情需要知道。在以太网上运行 IP 时，实际上使用了两种以太网协议: 一种是 IP，另一种是 ARP。 ARP 将主机的 IP 地址转换为其以太网地址（MAC 层）。为了允许桥接两个主机之间的通信，必须使桥接将转发 ARP 数据包。这种协议不包括在 IP 层中，因为它仅在 IP 在以太网上运行时存在。FreeBSD 防火墙仅在 IP 层上进行过滤，因此所有非 IP 数据包（包括 ARP）将被转发，而不会被过滤，即使防火墙配置为不允许任何东西。

现在是时候重新启动系统并像以前一样使用它：桥梁和防火墙会有一些新消息，但桥梁不会被激活，防火墙处于 open 模式，不会阻止任何操作。

如果有任何问题，您应该现在解决它们，然后继续。

## 4. 启用桥梁

到这一步，为了启用桥接，您需要执行以下命令（要机智地替换两个网络接口 fxp0 和 xl0 的名称为您自己的名称）：

```
# sysctl net.link.ether.bridge.config=fxp0:0,xl0:0
# sysctl net.link.ether.bridge.ipfw=1
# sysctl net.link.ether.bridge.enable=1
```

第一行指定应由桥接激活的接口，第二行将在桥接上启用防火墙，最后一行将启用桥接。

到这一步，您应该能够在两组主机之间插入机器，而不会影响它们之间的任何通信能力。如果是这样，请将这些行的 net.link.ether.bridge.<em>[blah]</em>=<em>[blah]</em> 部分添加到 /etc/sysctl.conf 文件中，以便它们在启动时执行。

## 5. 配置防火墙

现在是时候创建您自己的带有自定义防火墙规则的文件，以保护内部网络。这样做会有一些复杂性，因为并非所有防火墙功能都适用于桥接包。此外，正在转发的数据包与接收到本地机器的数据包之间存在差异。一般来说，传入的数据包只经过一次防火墙过滤，而不是像通常情况下那样两次；事实上，它们只在接收时进行过滤，因此使用 out 或 xmit 的规则永远不会匹配。个人而言，我使用的是 in via ，这是一种较旧的语法，但阅读起来有意义。另一个限制是，您仅限于使用 pass 或 drop 命令来过滤通过桥接的数据包。像 divert 、 forward 或 reject 这样复杂的选项是不可用的。这些选项仍然可以使用，但仅限于与桥接机器本身的流量（如果它有 IP 地址）。

FreeBSD 4.0 中的新功能是状态过滤的概念。这对于 UDP 流量来说是一个重大改进，通常是请求发送出去，然后紧接着以相同的 IP 地址和 port 数字（当然，源和目标颠倒）的响应。对于没有状态保持的防火墙来说，几乎没有办法将这种流量处理为单个会话。但是，对于可以“记住”出站 UDP 数据包并在接下来的几分钟内允许响应的防火墙，处理 UDP 服务是轻而易举的。以下示例显示了如何做到这一点。可以对 TCP 数据包执行相同的操作。这使您可以避免一些拒绝服务攻击和其他恶意技巧，但通常会使您的状态表迅速增长。

让我们来看一个示例设置。首先请注意，在 /etc/rc.firewall 的顶部已经有针对环回接口 lo0 的标准规则，所以我们不再需要关心它们。定制规则应放在一个单独的文件中（例如 /etc/rc.firewall.local），并在系统启动时通过修改我们定义的 open 防火墙的 /etc/rc.conf 行加载它们：

```
firewall_type="/etc/rc.firewall.local"
```

>你必须指定完整路径，否则它将不会被加载，有隔离网络的风险。 

对于我们的示例，假设有一个 fxp0 接口连接到外部（互联网），一个 xl0 接口连接到内部（局域网）。桥接机器的 IP 是 1.2.3.4 （你的 ISP 不可能给你一个这样的地址，但对于我们的示例来说是好的）。

```
# Things that we have kept state on before get to go through in a hurry
add check-state

# Throw away RFC 1918 networks
add drop all from 10.0.0.0/8 to any in via fxp0
add drop all from 172.16.0.0/12 to any in via fxp0
add drop all from 192.168.0.0/16 to any in via fxp0

# Allow the bridge machine to say anything it wants
# (if the machine is IP-less do not include these rows)
add pass tcp from 1.2.3.4 to any setup keep-state
add pass udp from 1.2.3.4 to any keep-state
add pass ip from 1.2.3.4 to any

# Allow the inside hosts to say anything they want
add pass tcp from any to any in via xl0 setup keep-state
add pass udp from any to any in via xl0 keep-state
add pass ip from any to any in via xl0

# TCP section
# Allow SSH
add pass tcp from any to any 22 in via fxp0 setup keep-state
# Allow SMTP only towards the mail server
add pass tcp from any to relay 25 in via fxp0 setup keep-state
# Allow zone transfers only by the secondary name server [dns2.nic.it]
add pass tcp from 193.205.245.8 to ns 53 in via fxp0 setup keep-state
# Pass ident probes.  It is better than waiting for them to timeout
add pass tcp from any to any 113 in via fxp0 setup keep-state
# Pass the "quarantine" range
add pass tcp from any to any 49152-65535 in via fxp0 setup keep-state

# UDP section
# Allow DNS only towards the name server
add pass udp from any to ns 53 in via fxp0 keep-state
# Pass the "quarantine" range
add pass udp from any to any 49152-65535 in via fxp0 keep-state

# ICMP section
# Pass 'ping'
add pass icmp from any to any icmptypes 8 keep-state
# Pass error messages generated by 'traceroute'
add pass icmp from any to any icmptypes 3
add pass icmp from any to any icmptypes 11

# Everything else is suspect
add drop log all from any to any
```

那些曾经设置防火墙的人可能会注意到一些缺失的东西。特别是，没有防欺骗规则，事实上我们没有添加：

```
add deny all from 1.2.3.4/8 to any in via fxp0
```

也就是说，丢弃那些从外部来的声称是从我们网络来的数据包。这是您通常会做的事情，以确保没有人试图逃避数据包过滤器，通过生成看起来像来自内部的恶意数据包。问题在于，外部接口上至少有一个主机您不想忽略：路由器。但通常，ISP 在其路由器上进行反欺骗，所以我们不需要那么麻烦。

最后一条规则似乎是默认规则的精确副本，也就是，不允许任何未经明确允许的东西通过。但有一个区别：所有可疑流量将被记录。

传递 SMTP 和 DNS 流量到邮件服务器和名称服务器有两条规则。显然，整个规则集应该根据个人口味进行调整，这只是一个具体示例（规则格式在 ipfw(8) 手册中描述得很准确）。请注意，为了让 "relay" 和 "ns" 起作用，在启用桥接之前必须使名称服务查找工作正常。这是一个确保您在正确的网络卡上设置 IP 的示例。另外，您可以指定 IP 地址而不是主机名（如果该机器没有 IP 地址，则是必需的）。

习惯设置防火墙的人可能也习惯于为 ident 数据包（TCP port 113）设置 reset 或 forward 规则。不幸的是，在桥接中是不适用这个选项的，所以最好的方法就是直接将它们传递给目的地。只要目的地机器没有运行 ident 守护程序，这就相对无害。另一种选择是在 port 113 上断开连接，这会给像 IRC 这样的服务创建一些问题（ident 探测必须超时）。

您可能注意到的另一件有点奇怪的事情是有一个规则让桥接机器通信，另一个是为内部主机。请记住，这是因为两组流量将通过内核和数据包过滤器以不同的路径进行传输。内部网络将通过桥接，而本地机器将使用正常的 IP 栈进行通信。因此，两个规则用于处理不同情况。 in via fxp0 规则适用于两条路径。一般来说，如果在过滤器中使用 in via 规则，那么您需要为本地生成的数据包进行例外处理，因为它们并不是通过我们的任何接口进入的。

## 6. 赞助者

本文的许多部分都取自一篇关于桥梁的旧文，由 Nick Sayer 编辑过，并进行了更新和改编。两个启发来自 Steve Peterson 撰写的一篇关于桥梁的介绍。

特别感谢 Luigi Rizzo 在 FreeBSD 中实现桥梁代码以及他为我解答所有相关问题所付出的时间。

感谢汤姆·罗兹(Tom Rhodes)也审查了我从意大利语（本文的原始语言）翻译成英语的工作。

