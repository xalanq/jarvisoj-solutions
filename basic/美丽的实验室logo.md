## 描述

出题人丢下个logo就走了，大家自己看着办吧

[logo.jpg.8244d3d060e9806accc508ec689fabfb](./assets/logo.jpg.8244d3d060e9806accc508ec689fabfb)

## 题解

先 strings 了下，看不出啥。

然后丢去 010 Editor，发现有一个 unknownPadding 的多余数据，然后发现这段数据的开头和整张 jpg 图片的开头一致，所以猜测是两张 jpg 拼在了一起。

将多余数据丢到一个新的文件，得到一张新图片，里面有答案。

## 答案

PCTF{You_are_R3ally_Car3ful}
