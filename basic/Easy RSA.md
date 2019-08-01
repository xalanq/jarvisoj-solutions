## 描述

还记得veryeasy RSA吗？是不是不难？那继续来看看这题吧，这题也不难。

已知一段RSA加密的信息为：0xdc2eeeb2782c且已知加密所用的公钥：

(N=322831561921859 e = 23)

请解密出明文，提交时请将数字转化为ascii码提交

比如你解出的明文是0x6162，那么请提交字符串ab

提交格式:PCTF{明文字符串}

## 题解

先用 factor 分解质因数

```
$ factor 322831561921859
322831561921859: 13574881 23781539
```

再用 [veryeasyRSA](./veryeasyRSA.md) 那题的解法

```python
def exgcd(a, b):
    if not b:
        return a, 1, 0
    d, y, x = exgcd(b, a % b)
    y -= a // b * x
    return d, x, y

def get_inverse(a, p):
    d, x, y = exgcd(a, p)
    return (x + p) % p

e = 23
phi_n = (13574881 - 1) * (23781539 - 1)
d = get_inverse(e, phi_n)
```

解得 d = 42108459725927

然后再写个解密

```python
n = 322831561921859
d = 42108459725927
data = 0xdc2eeeb2782c

def fpow(a, b, c):
    a = a % c
    r = 1
    while b:
        if b & 1:
            r = r * a % c
        a = a * a % c
        b = b >> 1
    return r

ans = fpow(data, d, n)
hx = bytes.fromhex(format(ans, 'x'))
print(hx.decode('utf-8'))
```

## 答案

PCTF{3a5Y}
