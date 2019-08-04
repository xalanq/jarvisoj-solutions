## 描述

听说神盾局的网络被日穿之后，有人从里面挖出来一个神秘的文件，咋一看也没什么，可是这可是class10保密等级的哦，里面一定暗藏玄机，你能发现其中暗藏的玄机吗？

[class10.1c40ca6a83c607f424c23402abe53981](./assets/class10.1c40ca6a83c607f424c23402abe53981)

## 题解

```
$ binwalk class10 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
110           0x6E            Zlib compressed data, compressed
1000073       0xF4289         Zlib compressed data, default compression
```

`010 Editor` 打开直接看到了 `IHDR`，说明是 png 文件，但是前面那个开头 `89 50 4E 47 0D 0A 1A 0A` 被抹掉了，我们加回去。

直接看到 Flag

PCTF{SHIELD_Class_10_C4n_You_find_the_fl4g?}

然后输入进去发现错了。。。。md，修改后，再看一次

```
binwalk class10 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 1366 x 768, 8-bit/color RGBA, non-interlaced
110           0x6E            Zlib compressed data, compressed
1000073       0xF4289         Zlib compressed data, default compression
```

PNG 只会最多一个 Zlib，说明有一个有问题，`zsteg` 分析一下

```
$ zsteg class10.png 
[?] 174 bytes of extra data after zlib stream
extradata:0         .. file: zlib compressed data
    00000000: 78 9c 6d 92 0b 1a c3 20  08 83 af 94 dc ff 72 5b  |x.m.... ......r[|
    00000010: 95 3c d8 aa 7e b6 8a c8  4f 04 38 8d df 41 e2 19  |.<..~...O.8..A..|
    00000020: a7 4f 23 99 ad 7b 60 26  1e 1f 9c 85 dc 29 df 18  |.O#..{`&.....)..|
    00000030: 33 bd 18 27 62 7c fb a2  63 10 89 63 7a b9 7a c1  |3..'b|..c..cz.z.|
    00000040: 36 1f bd 65 5e c8 90 a5  19 83 88 09 96 8b ee 62  |6..e^..........b|
    00000050: 22 eb a7 42 46 b0 09 8b  9f 1c 74 aa be 30 7b 29  |"..BF.....t..0{)|
    00000060: 68 54 91 dc e4 d1 a1 93  c3 5f 7b 53 a6 6f 2a 7c  |hT......._{S.o*||
    00000070: 8d ce be 70 58 52 28 47  03 c0 d5 d0 84 5d 31 52  |...pXR(G.....]1R|
    00000080: a3 9e 00 21 76 29 55 f5  e9 fc 2e 36 9a db d7 39  |...!v)U....6...9|
    00000090: 9d 91 04 f1 b2 da 86 2d  29 2c dd f6 dc 6a 34 bf  |.......-),...j4.|
    000000a0: ab 73 ab ed 17 77 8d 3c  f3 07 fe 74 9f 40        |.s...w.<...t.@  |
imagedata           .. file: VAX-order 68K Blit (standalone) executable
b1,rgba,lsb,xy      .. text: "yywwwwwy"
b1,abgr,msb,xy      .. text: "wyyywyyy"
b2,b,lsb,xy         .. file: floppy image data (IBM SaveDskF)
b2,rgba,lsb,xy      .. text: ["c" repeated 15 times]
b3,r,lsb,xy         .. file: MPEG ADTS, layer II, v1, 224 kbps, Monaural
b4,r,lsb,xy         .. text: "wwfvwvwvx"
b4,r,msb,xy         .. text: "fjjjfjjj"
b4,g,lsb,xy         .. text: "1\"#\"#2$2233333TDDECVFUUUedUVfWVVffeUwvwgggwgggvwxw"
b4,g,msb,xy         .. text: "UYUUUUUUUYUU"
b4,b,lsb,xy         .. file: SoftQuad DESC or font file binary
b4,b,msb,xy         .. file: VISX image file
[6]   Done                    stegsolve.jar
```

出来了贼多东西，所以我猜这里面肯定塞了个压缩包进去。。

折腾了好久，解压出来的那些东西根本开不了。。。看题解吧。。

wdnmd，这是啥玩意，我的三观，啊，啊，，，QAQ

用 `binwalk -e` 把那俩 `zlib` 的给解压出来，发现其中有一个是文本文件，里面全是 0 和 1，然后长度是 841 个字符。。。。。factor 一下发现是 29 x 29。。。。然后就是个二维码。

吐血。

```python
from PIL import Image

sz = 29
im = Image.new('L', (sz, sz))
data = open('F4289', 'r').read()
for i in range(sz):
    for j in range(sz):
        if data[i * sz + j] == '0':
            im.putpixel([i, j], 0)
        else:
            im.putpixel([i, j], 255)

im.show()
```

运行得到二维码，扫一下得到 flag

## 答案

PCTF{e32f2543fd5e246272eb7d15cc72a8ec}
