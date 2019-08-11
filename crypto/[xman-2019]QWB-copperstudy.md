## 描述

nc 47.97.215.88 12300

## 题解

给出的信息大概是这

```
nc 47.97.215.88 12300
[+]proof: skr=os.urandom(8)
[+]hashlib.sha256(skr).hexdigest()=a176d7f9062d5540f69b3842fd4d38ce4c01d59d3f1fa95b898b5f78af27c639
[+]skr[0:5].encode('hex')=70ba5bad07
[-]skr.encode('hex')=
```

而且过一段时间会自动断开。

想都不用想，直接暴力。它已经告诉了我们 5 位，我们再爆破 3 位即可。

```python
import hashlib
import struct
import random

sha = input().strip()
skr = input().strip()

def rg():
    a = list(range(256))
    random.shuffle(a)
    return a

for i in rg():
    for j in rg():
        for k in rg():
            t = skr + format(i, '02x') + format(j, '02x') + format(k, '02x')
            if hashlib.sha256(bytes.fromhex(t)).hexdigest() == sha:
                print(t)
```

日，还有

```
[++++++++++++++++]proof completed[++++++++++++++++]
[+]Generating challenge 1
[+]n=0xccb42e1da27b1c1c1047a7377ea3bfe9bd85b50b753f58b2e5fe28144dd281ee9940ffc752b9fccde6bff54f90a67de0856239f6dd69f4467bf712551c9cea4d9e4f6e6d8648da48866cfca8845008376438b576227a44ac969e4d12b2f0445a6197874b432995e055b1d4aa09406159fbb051c58d55acf6f1afbbd472a45b75L
[+]e=3
[+]m=random.getrandbits(512)
[+]c=pow(m,e,n)=0x89f265e09b53cbc83d32461dc366c9a9004ccfc9a9e0c9ab26277f0e06d02508d6dfb059425430666ae66b0a21d53c3876086079a80c520ede768269a9055958d2c5f7ce6784a4a9ffc98d96dd5875b55ee26d6952b31fb207cc984a56a5af4052946dcc9d74d2dd77f2f4967ce2e190e469d9e0580b7a26f9aa01b9a780a031L
[+]((m>>72)<<72)=0xa6717a7ee57e329b717a29f2a9fc503641bf481d5c24198fe2f9c15dc3ddee11a184c46b0065b54fa332aebfed130d7d44da249ec51d27000000000000000000L
[-]long_to_bytes(m).encode('hex')=
```

Least Significant Bit Oracle Attack：https://github.com/ashutosh1206/Crypton/blob/master/RSA-encryption/Attack-LSBit-Oracle/README.md

## 答案
