## 描述

有人在内网发起了大量扫描，而且扫描次数不止一次，请你从capture日志分析一下对方第4次发起扫描时什么时候开始的，请提交你发现包编号的sha256值(小写)。

Hint1: 请提交PCTF{包编号的sha256}

[capture.rar.520fff452096bc407fab4567ecfb6b86](./assets/capture.rar.520fff452096bc407fab4567ecfb6b86)

## 题解

以为是文本文件用 `gedit` 差点没卡死虚拟机。。。`vim` 看了一眼全是二进制一样的东西，还是先 `binwalk` 一下，得到

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Libpcap capture file, little-endian, version 2.4, Ethernet, snaplen: 1514
7594653       0x73E29D        mcrypt 2.2 encrypted data, algorithm: blowfish-448, mode: CBC, keymode: 8bit
```

查了一下，是 `pcap` 文件，可以用 `WireSharp` 打开。打开后看到一片红黑交错的包，虽然我不知道颜色是啥意思，但是我知道 `ACK`、`SVN` 这种是 TCP 的 3 次握手，所以查一下就知道哪个是发出来的了。

所以第 4 个 `SVN` 就是我们要的包名。

试了 3 次全错了。

我是傻逼。

我有点不太懂。。。

看了题解，说是第四次 ICMP 协议的那个。。

我再看了下那些 TCP 包，发现他们验证的端口都不一样。。。。哦。。原来是在扫哪些端口开放。。。

所以题目里 “发起扫描” 应该是指对方第几次执行 `nmap` 之类的命令？

在 `filter` 一栏打 `icmp`，过滤出来，找第四个 `source` 不同的（即换了四个 IP 或者代理了 4 次），编号是 155989。

## 答案

PCTF{0be2407512cc2a40bfb570464757fd56cd0a1d33f0bf3824dfed4f0119133c12}
