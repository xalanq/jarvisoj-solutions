## 描述

出题人硬盘上找到一个神秘的压缩包，里面有个word文档，可是好像加密了呢~让我们一起分析一下吧！

[word.zip.a5465b18cb5d7d617c861dee463fe58b](./assets/word.zip.a5465b18cb5d7d617c861dee463fe58b)

## 题解

打开发现 bug occur，也就是文件有问题，猜测是伪加密的 zip。用 010 Editor 打开，使用 ZIP 的模板，然后找到的 deFlag ，也就是加密方式，发现是 9，将其改成 0。

然后解压得到一个 word 文件，发现被藏起来了。再将 word 给解压一下，在 media 文件夹里找到了第二张图片，得到答案。

## 答案

PCTF{You_Know_moR3_4boUt_woRd}
