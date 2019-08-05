## 描述

分析压缩包中的数据包文件并获取flag。flag为32位小写md5。

题目来源：CFF2016

[findwebshell.rar.96e24e913b817b7503f85fd36e0a4f17](./assets/findwebshell.rar.96e24e913b817b7503f85fd36e0a4f17)

## 题解

太简单了吧这题。直接 `strings findwebshell.pcapng | grep -i -n flag` 看到最后一行有一个十六进制串，猜测是 Base64，然后一解码，得到一个网址

https://dn.jarvisoj.com/challengefiles/AbTzA2YteFjGhPWCftraouVD3B684a9A.jpg

然后打开二维码一解析，就得到 flag 了。

## 答案

1542ae716e47576e1f3e36326a23e72e
