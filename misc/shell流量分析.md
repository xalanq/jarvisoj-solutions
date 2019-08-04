## 描述

分析一下shell流量，得到flag

来源：HCTF2016

[+_+.rar.977e2c637dc492fb9a7cf7595c852044](./+_+.rar.977e2c637dc492fb9a7cf7595c852044)

## 题解

`strings +_+.pcapng | grep -i -n ctf` 得到 `172:message = IV + 'flag is hctf{xxxxxxxxxxxxxxx}'`，直接文本编辑器找到相应的地方，看到一串 python 代码

```python
#!/usr/bin/env python
# coding:utf-8
__author__ = 'Aklis'

from Crypto import Random
from Crypto.Cipher import AES

import sys
import base64


def decrypt(encrypted, passphrase):
  IV = encrypted[:16]
  aes = AES.new(passphrase, AES.MODE_CBC, IV)
  return aes.decrypt(encrypted[16:])


def encrypt(message, passphrase):
  IV = message[:16]
  length = 16
  count = len(message)
  padding = length - (count % length)
  message = message + '\0' * padding
  aes = AES.new(passphrase, AES.MODE_CBC, IV)
  return aes.encrypt(message)


IV = 'YUFHJKVWEASDGQDH'

message = IV + 'flag is hctf{xxxxxxxxxxxxxxx}'


print len(message)

example = encrypt(message, 'Qq4wdrhhyEWe4qBF')
print example
example = decrypt(example, 'Qq4wdrhhyEWe4qBF') 
print example
```

执行一下得到

```
45
��h���Y�6:��r��w��su�/��>TGf�2pi����tm
flag is hctf{xxxxxxxxxxxxxxx}
```

猜测是加密了然后传了出去，我们差一个加密后的数据。用 `WireShark` 打开，慢慢看，看得出来是一个类似 ssh 的交互，远程写网站。

太多包了，好烦，看到一半不想看了，尝试直接搜。然后一波

`strings +_+.pcapng | grep -i -n flag` 得到

```
172:message = IV + 'flag is hctf{xxxxxxxxxxxxxxx}'
245:flag
252:at flag 
257:flag
```

文本编辑器打开相应位置，找到一个地方有一串 Base64 编码

```
mbZoEMrhAO0WWeugNjqNw3U6Tt2C+rwpgpbdWRZgfQI3MAh0sZ9qjnziUKkV90XhAOkIs/OXoYVw5uQDjVvgNA==
```

猜测这个就是传出去的 `message`，所以写上面的代码尝试一下

```python
example = base64.b64decode(b'mbZoEMrhAO0WWeugNjqNw3U6Tt2C+rwpgpbdWRZgfQI3MAh0sZ9qjnziUKkV90XhAOkIs/OXoYVw5uQDjVvgNA==')
example = decrypt(example, 'Qq4wdrhhyEWe4qBF') 
print example
```

然后便得到 flag

## 答案

hctf{n0w_U_w111_n0t_f1nd_me}
