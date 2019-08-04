## 描述

![](./assets/Flag.png)

## 题解

依次尝试 `strings` 和 `binwalk`，然后 `binwalk` 得出

```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 664 x 586, 8-bit/color RGB, non-interlaced
41            0x29            Zlib compressed data, default compression
```

一个压缩包。有很多种方法分离开来，可以参考 https://hackfun.org/2017/01/12/CTF中图片隐藏文件分离方法总结/ 

好吧，不是压缩包，Zlib 只是压缩了 png 图片的数据罢了，我不太信，也不想看题解，所以把这段 `Zlib compressed data, default compression` 拿去搜索了下，找到了一篇文章

https://blog.rootshell.be/2015/04/29/hack-in-paris-challenge-wrap-up/

我按他的尝试方法尝试了下，和我差不多，所以猜测是用 StegSolve 工具来解决。由于尝试 LSB 太多组合方案，故放弃。

在 https://github.com/zardus/ctf-tools 里搜 png，找到了个工具 https://github.com/zed-0xff/zsteg ，然后一试，卧槽，出来了

```
$ zsteg 42011487927629132.png 
imagedata           .. text: "KK<220\r\r"
b1,rgb,lsb,xy       .. file: Zip archive data, at least v2.0 to extract
b1,bgr,msb,xy       .. text: "saZ$S:'6"
b3,b,lsb,xy         .. text: "#?/(9Rk;"
b3,rgb,lsb,xy       .. text: "~G#\rwW:U"
b4,r,lsb,xy         .. text: "Ewe##333#\"#"
b4,r,msb,xy         .. text: ";UUUUUUU"
b4,g,lsb,xy         .. text: "yffgfTS22"
b4,g,msb,xy         .. text: "gF87rqw@Bw"
b4,b,lsb,xy         .. text: "ffffvvgwfvfw"
b4,rgb,msb,xy       .. text: "Dsr@3%\"7"
b4,bgr,msb,xy       .. text: "vCp2C\"5'"
```

然后 `zsteg -E b1,rgb,lsb,xy 42011487927629132.png > a.zip` 解压得到一个压缩包，里面有个文件 `1`。解压后发现是 ELF 文件，执行得到 Flag。

看了别人的题解，和我做的差不多。

## 答案

hctf{dd0gf4c3tok3yb0ard4g41n~~~}
