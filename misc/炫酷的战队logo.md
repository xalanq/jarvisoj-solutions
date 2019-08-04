## 描述

欣赏过了实验室logo，有人觉得我们战队logo直接盗图比较丑，于是我就重新设计了一个，大家再欣赏下？

[phrack.bmp.197c0ac62c8128bc4405a27eca3021b6](./assets/phrack.bmp.197c0ac62c8128bc4405a27eca3021b6)

## 题解

```
$ strings phrack.bmp | grep -i -n flag
$ strings phrack.bmp | grep -i -n ctf
$ file phrack.bmp 
phrack.bmp: data
$ binwalk

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
1143301       0x117205        Zlib compressed data, compressed

$ unzip phrack.bmp
Archive:  phrack.bmp
  End-of-central-directory signature not found.  Either this file is not
  a zipfile, or it constitutes one disk of a multi-part archive.  In the
  latter case the central directory and zipfile comment will be found on
  the last disk(s) of this archive.
unzip:  cannot find zipfile directory in one of phrack.bmp or
        phrack.bmp.zip, and cannot find phrack.bmp.ZIP, period.

$ zip -FF phrack.bmp --out gg.zip
Fix archive (-FF) - salvage what can
	zip warning: Missing end (EOCDR) signature - either this archive
                     is not readable or the end is damaged
Is this a single-disk archive?  (y/n): y
  Assuming single-disk archive
Scanning for entries...
	zip warning: zip file empty

```

懵逼。

用 `010 Editor` 看了下，发现大量重复的字符，我感觉好像字符画的样子？直接 `cat` 出来，发现没换行一样，啥也看不出来。

用 `python` 以文本形式读进来，然后输出了下长度是 1145146，分解一下质因子 `$ factor 1145146` 发现只有 2 572573，看起来不是。

懵逼。

查题解。

好吧，我脑洞太大了，傻逼一个。

其实这题正解是，二进制查看文件后，发现头部没有 BMP 文件的特征 `424D`，将其补全保存。然后就能看到图片了。

然后再 `binwalk` 一下，得到

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PC bitmap, Windows 3.x format,, 868 x 439 x 24
1143301       0x117205        Zlib compressed data, compressed
```

可能有多余的数据，毕竟 BMP 没得 Zlib 压缩，看到文件的末尾是 `IEND`，即 png 文件的结尾，故应该是一张 PNG 黏在后面了。

在 `010 Editor` 搜索 png 文件的开头特征 `89 50 4E 47 0D 0A 1A 0A` 得到位置 1171AA，于是用 `dd` 将其分离出来

```
$ dd if=phrack.bmp of=gg.png skip=$((0x1171AA)) bs=1
```

（这里 `$((0x1171AA))` 可以将其转成十进制）

解压出来打不开，文件损坏，`kali` 的图片浏览器提示我们 `IHDR` 里的 `CRC` 校验出错了，用 `010 Editor` 打开发现 `height = 0`，所以打不开。

这是典型的删除宽度（或高度）来隐藏的操作，详细可见

https://ctf-wiki.github.io/ctf-wiki/misc/picture/png-zh/#ihdr

爆破就好

```
import os
import binascii
import struct

misc = open("gg.png","rb").read()

for i in range(1024):
    data = misc[12:20] + struct.pack('>i', i) + misc[24:29]
    crc32 = binascii.crc32(data) & 0xffffffff
    if crc32 == 0xF37A5E12:
        print(i)
```

然后居然啥都没爆出来？？？？？？？

MD，看了题解发现，宽其实也有问题，那个 256 是假的！所以要爆破宽和高

```
import os
import binascii
import struct

misc = open("gg.png","rb").read()

for i in range(1024):
    for j in range(1024):
        data = misc[12:16] + struct.pack('>i', i) + struct.pack('>i', j) + misc[24:29]
        crc32 = binascii.crc32(data) & 0xffffffff
        if crc32 == 0xF37A5E12:
            print(i, j)
```

然后爆出来是 450 x 450，改一下便得到了 flag

都怪我没有去了解过这些文件的特征

## 答案

PCTF{CrC32_i5_Useful_iN_pNG}
