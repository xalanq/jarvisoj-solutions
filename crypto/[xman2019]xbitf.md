## 描述

nc 47.97.215.88 10000

[xbitf.py.dfc0187cc423361f81729efe174aed4a](./assets/xbitf.py.dfc0187cc423361f81729efe174aed4a)

## 题解

链接一下会出现这个

```
$ nc 47.97.215.88 10000
session=96978945518d1e66;admin=0;checksum=576c0fc5ad8799a11a7acb4a580226f535fb48318b49461d27ebb3b182590690
```

再看了下代码，一个 AES 加密验证 `admin == 1`，感觉不太可做啊，AES 的密钥和 IV 都是 随机的。

但是我尝试提交了下，发现，这玩意 tm 会把报错信息给爆出来？

```
$ nc 47.97.215.88 10000
session=96978945518d1e66;admin=0;checksum=576c0fc5ad8799a11a7acb4a580226f535fb48318b49461d27ebb3b182590690
token:session=96978945518d1e66;admin=1;    
Traceback (most recent call last):
  File "/root/xbitf/xbitf.py", line 37, in <module>
    c_rcv=token_rcv.split("checksum=")[1].strip()
IndexError: list index out of range
```

所以我猜可能是想办法让报错信息把 flag 给爆出来。

想了一会。。好吧不行。。

查了下 AES，已知明文和密文，是爆破不出密钥的 orz

发现交互会进行十次，每次都会把解密的信息给输出，所以是不是要构造一下密文然后去爆破出密钥？

搜了下，有一个叫 `CBC 比特翻转` 的攻击方法，就是用亦或翻转某一个字符，比如这个例子：

```
session=96978945518d1e66;admin=0
```

我们要修改的是 `admin=0` 里的 `0` 变为 `1`，那么我们可以将密文的某个位置，亦或上 `ord('0') ^ ord('1')`，就能实现密文解码后变成这个位置上变为 `1`，但是我们应该修改的是密文的哪个位置呢？我们可以通过写一份代码来验证

```python
from Crypto.Cipher import AES
import os

def aes_cbc(key, iv, m):
    handler = AES.new(key, AES.MODE_CBC, iv)
    return handler.encrypt(m.encode()).hex()

def aes_cbc_dec(key, iv, c):
    handler=AES.new(key, AES.MODE_CBC, iv)
    return handler.decrypt(bytes.fromhex(c))

key = os.urandom(16)
iv = os.urandom(16)
token = 'session=96978945518d1e66;admin=0'
checksum = aes_cbc(key, iv, token)
print('token    =', token)
print('checksum =', checksum)
for i in range(len(token)):
    t = bytearray(bytes.fromhex(checksum))
    t[i] = t[i] ^ ord('0') ^ ord('1')
    new_checksum = t.hex()
    print('{:02d}       ='.format(i), aes_cbc_dec(key, iv, new_checksum))
```

得到 

```
token    = session=96978945518d1e66;admin=0
checksum = 6535360f64b9d5a60099aaa82df64348fa519e0ff9e853b8035b51b0810b9e8e
00       = b'\xb3\xc1\xef\x9a\x0c\x7fr\x9e1\xc3\x94\xf66v\xb8"418d1e66;admin=0'
01       = b')e\x96\xe2S\xcc>\x84%\xcbc\rt\xe5\xe1\x88508d1e66;admin=0'
02       = b')4\x9eu\x9c\xb3\xc3\x7f\xb1\x0e\xac\x9cgI\xea\x9e519d1e66;admin=0'
03       = b'sw;,4\xe9\\U3\x17\xa9\xe1\x96\x83\xb9\x0e518e1e66;admin=0'
04       = b'&}\xc6\xf7)|\x17\xce\xa4\x8a\xb6\x8a\xb1\xed\x01x518d0e66;admin=0'
05       = b"\x06\xea.\xacMjU\xb1'\x8b\xe6\xca\x19u\x81\x03518d1d66;admin=0"
06       = b'O^!\xc3\x139K\x19:\x9eq\x97\xb0\xe8\x88\xe0518d1e76;admin=0'
07       = b'\xec8,\xfc\x8d\xac\xb9\xec\xb3\xbb\x91\xb4\xdb\x0b\xce\xf0518d1e67;admin=0'
08       = b'a\xc5?\x94\xc9\xd9\x18u\x92\xd8\xe7\x8d\xf3-\xf4\xed518d1e66:admin=0'
09       = b'\xe3f\x1dC\xed\x80\xa5\xcb\xdaE\x18f\xc9\xa8\xd1\x86518d1e66;`dmin=0'
10       = b'\xe2\xdb\xdd\xd27H\x9d\xa7T\xadU\xf1\xa1\t\x9d[518d1e66;aemin=0'
11       = b'p2\xc5&\x16}/\x80\xff\xf4"\xb8\x9b\xaa\xf0P518d1e66;adlin=0'
12       = b'\xff\x10\xfem!\xd5#\xa1\xd5F\xa5\xfa\xb7\xb4\x1a\x8c518d1e66;admhn=0'
13       = b'\xb1\xd0\xa2Ob\xa9\xaf\xa0\x00\xf1\xcfwc\x8c\xa7\xc5518d1e66;admio=0'
14       = b';\xca\xf5l\x05\x83\x82<A\x9f7\x1ai\x1d\xcb\xad518d1e66;admin<0'
15       = b'J\xff;:\xddM\xe8\xd0-\x8b\xaf\x91\rl\xf5Q518d1e66;admin=1'
16       = b'session=96978945\xd1N\xa5Vs\xa1\xaf\x1e\x9d\xca\x9e\xc1S\xef\x89\x84'
17       = b'session=96978945ND\xde\xf1\\\xceu\xcfGI+\x8e\x9c\xbc\xd8\x0f'
18       = b'session=96978945\xdb\x9e\t\x8b\xd8]\x8c<\xc6\xf2N\x95o\xce\x16\xb6'
19       = b'session=96978945\xcd\x85\xf3\x86g\xdb\xbf\x8a\x8d\xce\xd4\xd6!\x90\xfa\xb2'
20       = b'session=96978945H\xbf\xb1\xb2\xbb\x01\xb1lDX\xa7\x18b\xa46"'
21       = b"session=96978945\xd2\xe5'3\xfe\xcf y\xfeWB\xa4\x9d\xa1\xa63"
22       = b'session=96978945\x1a\xf7\xaf\x05$\x9e\x82l\xd5\xcd\x9dEbQc+'
23       = b'session=969789458QKr65\t\xf4\xec\x01z\x7f\xf0\xdbb\x1d'
24       = b'session=96978945\x89],\x11$\xe8\x8c\xae\t\xce\xc5\x17R\xab\x8d\x0c'
25       = b'session=96978945tYN\x0e2\x90T\xcc9Th\xbb\xe3\xdc\xad '
26       = b'session=96978945\xd4Fe#\x9cD\xa6u\x00/w\x94Q\x0e\xbd@'
27       = b'session=96978945\xbb\xf6R\xbf\\\xf7\xb2K]\x1b\xb5\xa3\xfa\n\xc8h'
28       = b'session=96978945_\xc3\x03\xb2\xff\x9bl\x98\x17\t\x14\xa8TJ\xae3'
29       = b'session=96978945\xec\xee\x96\xb2N\x91v?\xd7\x1c,\xd5\xf2fb\xdf'
30       = b'session=96978945c~\xb1\x1brLk\xfa#w#\xa4,\xb5\xb0\x8c'
31       = b'session=96978945|v_\xf4\xfe3k\x15)\xc3:B\xe8X\xf2\xb9'
```

可以发现在 i = 15 的时候修改成功了。所以这道题就直接修改第 15 位即可

```python
text = input().strip()
session = text.split('admin=0')[0]
checksum = text.split('checksum=')[1]
t = bytearray(bytes.fromhex(checksum))
t[15] = t[15] ^ ord('0') ^ ord('1')
new_checksum = t.hex()
print(session + 'admin=1;checksum=' + new_checksum)
```

## 答案

flag{xman_bit_flip_112233}
