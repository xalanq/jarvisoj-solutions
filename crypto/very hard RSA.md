## 描述

前几题因为N太小，都被你攻破了，出题人这次来了个RSA4096，是否接受挑战就看你了。

[veryhardRSA.rar.2a89300bd46d028a14e2b5752733fe98](./assets/veryhardRSA.rar.2a89300bd46d028a14e2b5752733fe98)

## 题解

看一下给我们的 py 文件可以知道，同一个 n，互质的 e1, e2。共模攻击。和 [[xman2019]xgm](./[xman2019]xgm.md) 这题一样。

```python
import sys
sys.setrecursionlimit(10000000)

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

n = 0x00b0bee5e3e9e5a7e8d00b493355c618fc8c7d7d03b82e409951c182f398dee3104580e7ba70d383ae5311475656e8a964d380cb157f48c951adfa65db0b122ca40e42fa709189b719a4f0d746e2f6069baf11cebd650f14b93c977352fd13b1eea6d6e1da775502abff89d3a8b3615fd0db49b88a976bc20568489284e181f6f11e270891c8ef80017bad238e363039a458470f1749101bc29949d3a4f4038d463938851579c7525a69984f15b5667f34209b70eb261136947fa123e549dfff00601883afd936fe411e006e4e93d1a00b0fea541bbfc8c5186cb6220503a94b2413110d640c77ea54ba3220fc8f4cc6ce77151e29b3e06578c478bd1bebe04589ef9a197f6f806db8b3ecd826cad24f5324ccdec6e8fead2c2150068602c8dcdc59402ccac9424b790048ccdd9327068095efa010b7f196c74ba8c37b128f9e1411751633f78b7b9e56f71f77a1b4daad3fc54b5e7ef935d9a72fb176759765522b4bbc02e314d5c06b64d5054b7b096c601236e6ccf45b5e611c805d335dbab0c35d226cc208d8ce4736ba39a0354426fae006c7fe52d5267dcfb9c3884f51fddfdf4a9794bcfe0e1557113749e6c8ef421dba263aff68739ce00ed80fd0022ef92d3488f76deb62bdef7bea6026f22a1d25aa2a92d124414a8021fe0c174b9803e6bb5fad75e186a946a17280770f1243f4387446ccceb2222a965cc30b3929
e1 = 17
e2 = 65537
c1 = int(open('flag.enc1','rb').read().hex(), 16)
c2 = int(open('flag.enc2','rb').read().hex(), 16)
_, s1, s2 = exgcd(e1, e2)

m = fpow(c1, s1, n) * fpow(c2, s2, n) % n

hx = bytes.fromhex(format(m, 'x'))
print(hx)
```

## 答案

PCTF{M4st3r_oF_Number_Th3ory}
