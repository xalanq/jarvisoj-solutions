## 描述

分析压缩包中的数据包文件并获取flag。flag为32位大写md5。

题目来源：CFF2016

[struts2.rar.5b541c4760bb277b0853ec59e8726e86](./assets/struts2.rar.5b541c4760bb277b0853ec59e8726e86)

## 题解

真的，这种题怎么称得上 500 分的？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？？

`strings struts2.pcapng | grep -i -n flag` 直接就出来了啊，一点技术含量没有 OK？

## 答案

E3274BE5C857FB42AB72D786E281B4B8
