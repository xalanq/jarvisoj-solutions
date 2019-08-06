## 描述

相信你已经做出了medium RSA，这题的pubkey在medium RSA的基础上我做了点手脚，继续挑战吧。

Hint1: 1.不需要爆破。2.用你的数学知识解决此题。3.难道大家都不会开根号吗？

[hardRSA.rar.b498edae4e73af8eb4567fb18117de46](./assets/hardRSA.rar.b498edae4e73af8eb4567fb18117de46)

## 题解

先看看公钥里的信息

```
$ RsaCtfTool.py --dumpkey --key pubkey.pem
[*] n: 87924348264132406875276140514499937145050893665602592992418171647042491658461
[*] e: 2
```

e 是 2，所以

c = m^e % n = m * m % n

然后 n 不是特别大，用 `yafu` 分解他看看。过了一分钟左右，分解出来了

```
p = 319576316814478949870590164193048041239
q = 275127860351348928173285174381581152299
```

但是你会惊奇的发现，phi(n) = (p - 1) * (q - 1) 显然有一个 2 的因子，所以 gcd(e, phi(n)) ≠ 1，所以求不出 e 的逆元。

看题解说是 rabin 算法 orz，见 https://ctf-wiki.github.io/ctf-wiki/crypto/asymmetric/rsa/rsa_e_attack-zh/#rsa-rabin

```python
def exgcd(a, b):
    if not b:
        return a, 1, 0
    d, y, x = exgcd(b, a % b)
    return d, x, y - a // b * x

def inv(a, p):
    d, x, y = exgcd(a, p)
    return (x + p) % p

def fpow(a, b, c):
    a = a % c
    r = 1
    if b < 0:
        a = inv(a, c)
        b = -b
    while b:
        if b & 1:
            r = r * a % c
        a = a * a % c
        b = b >> 1
    return r

p = 319576316814478949870590164193048041239
q = 275127860351348928173285174381581152299
assert(p % 4 == 3 and q % 4 == 3)
n = p * q
cipher = int(open('flag.enc','rb').read().hex(), 16)
inv_p = inv(p, q)
inv_q = inv(q, p)

mp = fpow(cipher, (p + 1) // 4, p)
mq = fpow(cipher, (q + 1) // 4, q)

a = (inv_p * p * mq + inv_q * q * mp) % n
b = n - a
c = ((inv_p * p * mq - inv_q * q * mp) % n + n) % n
d = n - c

for i in (a, b, c, d):
    try:
        s = format(i, 'x')
        if len(s) % 2 != 0:
            s = '0' + s
        hx = bytes.fromhex(s)
        print(hx)
    except:
        pass
```

## 答案

PCTF{sp3ci4l_rsa}
