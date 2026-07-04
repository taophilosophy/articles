# FreeBSD 对闰秒的支持

- 原文：[FreeBSD Support for Leap Seconds](https://docs.freebsd.org/en/articles/leap-seconds/)

## 1. 介绍

*闰秒*是在一年中的特定时刻对 UTC 进行的一秒钟调整，用以将原子时间尺度与地球自转的变化同步。本文介绍了 FreeBSD 如何处理闰秒。

截至本文写作时，下一个闰秒将在 2015 年 6 月 30 日 23:59:60 UTC 发生。这个闰秒将在北美、南美和亚太地区的工作日内发生。

闰秒由 [IERS](https://www.iers.org/IERS/EN/Home/home_node.html) 在 [公告 C](https://datacenter.iers.org/data/latestVersion/16_BULLETIN_C16.txt) 中宣布。

标准的闰秒行为在 [RFC 7164](https://datatracker.ietf.org/doc/html/rfc7164#section-3) 中有描述。另请参考 [time2posix(3)](https://man.freebsd.org/cgi/man.cgi?query=time2posix&sektion=3&format=html)。

## 2. FreeBSD 默认的闰秒处理

处理闰秒的最简单方法是使用 FreeBSD 默认的 POSIX 时间规则，结合 [NTP](https://docs.freebsd.org/en/books/handbook/#network-ntp)。当 [ntpd(8)](https://man.freebsd.org/cgi/man.cgi?query=ntpd&sektion=8&format=html) 运行时，且系统时间与正确处理闰秒的上游 NTP 服务器同步，闰秒将导致系统时间自动重复当天的最后一秒。无需其他调整。

如果上游 NTP 服务器没有正确处理闰秒， [ntpd(8)](https://man.freebsd.org/cgi/man.cgi?query=ntpd&sektion=8&format=html) 会在出错的上游服务器注意到并自行调整后，将系统时间调整一秒。

如果没有使用 NTP，在闰秒过去后需要手动调整系统时钟。

## 3. 注意事项

闰秒是在全球范围内同一时刻插入的：UTC 午夜。在日本是上午中段，在太平洋地区是中午，在美洲是傍晚，而在欧洲是晚上。

我们相信并期望，FreeBSD 在提供正确且稳定的 NTP 服务时，会在这次闰秒时按设计正常工作，就像之前的闰秒一样。

然而，我们要提醒的是，几乎没有应用程序曾经向内核询问过关于闰秒的信息。我们的经验是，按照设计，闰秒本质上是对闰秒前一秒的重播，这出乎大多数应用程序开发人员的意料。

其他操作系统和其他计算机可能像 FreeBSD 一样处理闰秒，也可能不会，没有正确和稳定的 NTP 服务的系统根本无法了解闰秒。

计算机因闰秒而崩溃并非罕见，经验表明，很大一部分公共 NTP 服务器可能会错误地处理并宣布闰秒。

请尽量确保闰秒事件不会造成严重问题。

## 4. 测试

可以测试是否会使用闰秒。由于 NTP 的性质，测试在闰秒前最多 24 小时内可能有效。一些主要的参考时钟源只会在事件发生前一小时宣布闰秒。查询 NTP 守护进程：

```sh
% ntpq -c 'rv 0 leap'
```

包含 `leap_add_sec` 的输出表示正确支持闰秒。在距闰秒发生还有超过 24 小时的时候，或闰秒过后，将显示 `leap_none`。

## 5. 结论

实际上，闰秒通常不会在 FreeBSD 上造成问题。我们希望这篇概述能帮助澄清预期情况，并让闰秒事件更顺利地进行。
