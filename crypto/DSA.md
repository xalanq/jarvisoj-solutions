## 描述

DSA是基于整数有限域离散对数难题的，其安全性与RSA相比差不多。DSA的一个重要特点是两个素数公开，这样，当使用别人的p和q时，即使不知道私钥x，你也能确认它们是否是随机产生的，还是作了手脚。

可以使用openssl方便地进行dsa签名和验证。

签名与验证：

openssl dgst -sha1 -sign dsa_private.pem -out sign.bin message.txt

openssl sha1 -verify dsa_public.pem -signature sign.bin message.txt

本题的攻击方法曾被用于PS3的破解，答案格式：CTF{x}(x为私钥，请提交十进制格式)

[dsa.rar.fca5b1bd311ecbf50cad05f31ba995c4](./assets/dsa.rar.fca5b1bd311ecbf50cad05f31ba995c4)

## 题解

看到本题，一脸懵逼，不会做啊。。

DSA 的原理可见 https://ctf-wiki.github.io/ctf-wiki/crypto/signature/dsa-zh/

攻击方法上面的链接给了两种，一种是已知 k，另一种是 k 共享。本题的话是 k 共享攻击，解出密钥 x。

根据 r = (g^k mod p) mod q，可知若 k 一样，则 r 一样。然后再根据两个不同的 s 值，便可以解出 x，具体推导可见上面链接。

在本题，我们先输出一下各个文件的验证情况与 r、s

（注意，把 pycrypto 给删了吧，用 pycryptodome；同时注意题目告诉我们用 sha1 而不是 sha256

```python
from Crypto.PublicKey import DSA
from Crypto.Signature import DSS
from Crypto.Util.asn1 import DerSequence
from Crypto.Hash import SHA1

pubkey_file = 'dsa_public.pem'
files = [
  ['./packet1/message1', './packet1/sign1.bin'],
  ['./packet2/message2', './packet2/sign2.bin'],
  ['./packet3/message3', './packet3/sign3.bin'],
  ['./packet4/message4', './packet4/sign4.bin'],
]

pubkey = DSA.import_key(open(pubkey_file, 'r').read())

y = pubkey.y
g = pubkey.g
p = pubkey.p
q = pubkey.q

print('y =', y)
print('g =', g)
print('p =', p)
print('q =', q)

verifier = DSS.new(pubkey, 'fips-186-3', 'der')

for f in files:
    try:
        sha = SHA1.new(open(f[0], 'rb').read())
        signature = open(f[1], 'rb').read()
        verifier.verify(sha, signature)
        print('=======================================')
        print(f, ' verification is OK')
        der_seq = DerSequence().decode(signature, strict=True)
        print('sha1 =', sha.hexdigest())
        print('r =', int(der_seq[0]))
        print('s =', int(der_seq[1]))
    except:
        print(f, 'verification is not OK')
        continue
```

输出如下

```
y = 48966667837013414801031811810642280772647236378744040304912885467993823167141281755249027785686488654582548005766785141294865650644124270468685164456792976944411071341740863520743350743985636739430430553599637295250998469649774667076175577773243724165204009991562726052681112720456820018391695334506416243823
g = 53955759259506195629370221955882028602129565196710167480649056988330906117971891319360466002581132965581793497788773734702624458318101767983345305577358509640139391448262692595149152168335401138516115001646783546740429454038931813085439969488499666706724464185937932560157850845046285161405028860099711467897
p = 135072277349986231918767318777496371552998523256306836680207886328250469140725070958493165166375742776776188095739453689162118072119637932393028705045785456135605269009498175908369054264845986650254461451778502410559449967848738173219539609196568432769555817217073171835795353828043869976253611672766967842971
q = 768204286206312924745826772404361572053995803069
=======================================
['./packet1/message1', './packet1/sign1.bin']  verification is OK
sha1 = 7fc1806f61a1a06346a4a1b784dfbcc8906d413d
r = 738437995981556045245595063149489742966556435465
s = 127640912786532160080001678238425750876521035522
=======================================
['./packet2/message2', './packet2/sign2.bin']  verification is OK
sha1 = 46ea25f6c5621195fe29e49027cccf10bcb643bf
r = 552209889428382347558845903868077675026123086463
s = 352721313857222439707115609826294110683053823413
=======================================
['./packet3/message3', './packet3/sign3.bin']  verification is OK
sha1 = c188b855c50a1b8247a8d1faf5f5ec3a8983c0ce
r = 459949603688030721373430799631292914769067962828
s = 279284157876018828861359417225115972081115359101
=======================================
['./packet4/message4', './packet4/sign4.bin']  verification is OK
sha1 = a54e8c02fa296b42b67ecda5afc1742cd2d59666
r = 459949603688030721373430799631292914769067962828
s = 537021354262452057613101170898297952892696215843
```

可以看到，3 和 4 的 r 是一样的。于是便能进行破解了

```python
from Crypto.PublicKey import DSA
from Crypto.Util.asn1 import DerSequence
from Crypto.Hash import SHA1
import gmpy2

pubkey_file = 'dsa_public.pem'
file_1 = ['./packet3/message3', './packet3/sign3.bin']
file_2 = ['./packet4/message4', './packet4/sign4.bin']

pubkey = DSA.import_key(open(pubkey_file, 'r').read())

y = pubkey.y
g = pubkey.g
p = pubkey.p
q = pubkey.q

print('y =', y)
print('g =', g)
print('p =', p)
print('q =', q)

def get_rs(signature):
    der_seq = DerSequence().decode(signature, strict=True)
    return int(der_seq[0]), int(der_seq[1])

hm1 = int(SHA1.new(open(file_1[0], 'rb').read()).hexdigest(), 16)
hm2 = int(SHA1.new(open(file_2[0], 'rb').read()).hexdigest(), 16)
r, s1 = get_rs(open(file_1[1], 'rb').read())
_, s2 = get_rs(open(file_2[1], 'rb').read())

print('r =', r)
print('hm1 =', hm1)
print('s1 =', s1)
print('hm2 =', hm1)
print('s2 =', s2)

print('cracking')

ds = s2 - s1
dhm = hm2 - hm1
k = gmpy2.mul(dhm, gmpy2.invert(ds, q))
k = gmpy2.f_mod(k, q)
tmp = gmpy2.mul(k, s1) - hm1
x = tmp * gmpy2.invert(r, q)
x = gmpy2.f_mod(x, q)
print('x =', x)
```

运行便得到 x

## 答案

CTF{520793588153805320783422521615148687785086070744}
