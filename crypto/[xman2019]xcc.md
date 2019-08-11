## 描述

nc 47.97.215.88 10001

[xcc.py.629eddb9e7000bc66c2c7b23daa41f8b](./assets/xcc.py.629eddb9e7000bc66c2c7b23daa41f8b)

## 题解

AES 加密。构造明文，收到密文，需要求 IV，有十次机会。

我了解了一下 IV 的原理，然后做了一个尝试

```python
from Crypto.Cipher import AES
import os

def aes_cbc_dec(key, iv, c):
    handler=AES.new(key, AES.MODE_CBC, iv)
    return handler.decrypt(c)

key = os.urandom(16)
iv = '1234567890123456'
s = 'a' * 16

x = aes_cbc_dec(key, iv, s)
y = aes_cbc_dec(key, iv, s)
z = ''
for i in range(len(x)):
    z += chr(ord(x[i]) ^ ord(y[i]))

print(iv)
print(z)
```

然后输出一致~

那么要是我们知道了 key，我们只需随便构造一个密文字符串，然后通过服务器得到明文，然后在本地将 IV 置为 0，然后进行解密得到另一个明文，然后两个明文取亦或就好~

就是不知道 key 是多少。。。总不可能暴力枚举 key 吧

想了好久不会做，题解也找不到。。

想到三个亦或等于 0 的明文经过加密后得到的密文会不会也是亦或等于 0 呢？于是写了个这个来尝试

```python
from Crypto.Cipher import AES
import os
import random
from bitarray import bitarray

def rnd():
    a = [bitarray(), bitarray(), bitarray()]
    for i in range(3):
        a[i].append(0)
    for i in range(7):
        while True:
            x = random.randint(0, 2)
            y = random.randint(0, 2)
            if x != y:
                break
        a[x].append(True)
        a[y].append(True)
        a[3-x-y].append(False)
    return a[0].tobytes(), a[1].tobytes(), a[2].tobytes()

def xr(x, y, z):
    t = ''
    for i in range(16):
        t += chr(ord(x[i]) ^ ord(y[i]) ^ ord(z[i]))
    return t

def p(x):
    print(' '.join(['{:03d}'.format(ord(c)) for c in x]))

def aes_cbc(key, iv, m):
    handler = AES.new(key, AES.MODE_CBC, iv)
    return handler.encrypt(m)

key = os.urandom(16)
iv = '1234567890123456'

a = [''] * 3
for i in range(16):
    x, y, z = rnd()
    a[0] += x
    a[1] += y
    a[2] += z
p(a[0])
p(a[1])
p(a[2])
p(xr(a[0], a[1], a[2]))

b = [''] * 3
for i in range(3):
    b[i] = aes_cbc(key, iv, a[i])

p(b[0])
p(b[1])
p(b[2])
p(xr(b[0], b[1], b[2]))
```

没得卵用。。。一个字符串经过多次经过解密会不会回到 IV 呢？尝试了一下，也不会。。

老天啊，给我找到份题解吧！

先搁着！

来了来了，找到题解了：https://github.com/ashutosh1206/Crypton/blob/master/Block-Cipher/CBC-IV-Detection/README.md

![](./assets/xcc.svg)

假设密文有三个块（如上图），CBC 的解密算法是这样的

```
p1 = D(c1) xor iv
p2 = D(c2) xor c1
p3 = D(c3) xor c2
```

那么我们构造一下这三个密文块，使得

```
p1' = D(c1) xor iv
p2' = D("\x00"*blocksize) xor c1
p3' = D(c3 = c1) xor "\x00"*blocksize = D(c1)
```

那么 `iv = p1' xor p3'`

```python
a = 'a' * 16
print a.encode('hex')
p1 = raw_input('input p1(hex):').strip().decode('hex')
b = a + '\x00' * 16 + a
print b.encode('hex')
p3 = raw_input('input p3(hex):').strip().decode('hex')

t = ''
for i in range(16):
    t += chr(ord(p1[i]) ^ ord(p3[32 + i]))

print t
```

## 答案

iv_is_danger_!!!
