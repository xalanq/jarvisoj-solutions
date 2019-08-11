## 描述

nc 47.97.215.88 12300

## 题解

给出的信息大概是这

```
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

sha = input().strip()[1:]
skr = input().strip()[1:]

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
                exit(0)
```

日，还有，好像是刚开始，第一关！

### 第一关

```
[+]Generating challenge 1
[+]n=0xccb42e1da27b1c1c1047a7377ea3bfe9bd85b50b753f58b2e5fe28144dd281ee9940ffc752b9fccde6bff54f90a67de0856239f6dd69f4467bf712551c9cea4d9e4f6e6d8648da48866cfca8845008376438b576227a44ac969e4d12b2f0445a6197874b432995e055b1d4aa09406159fbb051c58d55acf6f1afbbd472a45b75L
[+]e=3
[+]m=random.getrandbits(512)
[+]c=pow(m,e,n)=0x89f265e09b53cbc83d32461dc366c9a9004ccfc9a9e0c9ab26277f0e06d02508d6dfb059425430666ae66b0a21d53c3876086079a80c520ede768269a9055958d2c5f7ce6784a4a9ffc98d96dd5875b55ee26d6952b31fb207cc984a56a5af4052946dcc9d74d2dd77f2f4967ce2e190e469d9e0580b7a26f9aa01b9a780a031L
[+]((m>>72)<<72)=0xa6717a7ee57e329b717a29f2a9fc503641bf481d5c24198fe2f9c15dc3ddee11a184c46b0065b54fa332aebfed130d7d44da249ec51d27000000000000000000L
[-]long_to_bytes(m).encode('hex')=
```

Known High Bits Message Attack

```python
from sage.all import *
n = 0xccb42e1da27b1c1c1047a7377ea3bfe9bd85b50b753f58b2e5fe28144dd281ee9940ffc752b9fccde6bff54f90a67de0856239f6dd69f4467bf712551c9cea4d9e4f6e6d8648da48866cfca8845008376438b576227a44ac969e4d12b2f0445a6197874b432995e055b1d4aa09406159fbb051c58d55acf6f1afbbd472a45b75
e = 3
c = 0x89f265e09b53cbc83d32461dc366c9a9004ccfc9a9e0c9ab26277f0e06d02508d6dfb059425430666ae66b0a21d53c3876086079a80c520ede768269a9055958d2c5f7ce6784a4a9ffc98d96dd5875b55ee26d6952b31fb207cc984a56a5af4052946dcc9d74d2dd77f2f4967ce2e190e469d9e0580b7a26f9aa01b9a780a031
m = 0xa6717a7ee57e329b717a29f2a9fc503641bf481d5c24198fe2f9c15dc3ddee11a184c46b0065b54fa332aebfed130d7d44da249ec51d27000000000000000000
k_bits = 72
F.<x> = PolynomialRing(Zmod(n)); 
f = (x + m)^e - c
x0 = f.small_roots(X=2^k_bits, beta=1)[0]
print "result m:", (int(x0) + m).hex()
```

得到 a6717a7ee57e329b717a29f2a9fc503641bf481d5c24198fe2f9c15dc3ddee11a184c46b0065b54fa332aebfed130d7d44da249ec51d27e4ead4a08036e8c72c

### 第二关

```
[+]Generating challenge 2
[+]n=0x2757328be3c648d2fae0c454f91aa82af78eb83318fac0311ada9bb9a64f30457650ea0e975946f7259eb45821f15528ac24dea1bde15bf8fc5a3e76787e0ac57c74e167d8144861a7608078f6693877dc9c59b53b7f8f6e34c1ce27a3bef79a33df6b04ee170ddf67116e8fe67e3a59a1cba216a41ca23b4ef6c214b5831e1L
[+]e=65537
[+]m=random.getrandbits(512)
[+]c=pow(m,e,n)=0x6ee16b34e2a5523e58e6a63fb44db5f696156246c7ebe27b9af29cd199d2de56dbb77f0e1057a831fb41bd1993c8f18f97de9cc71f9b68518e3817d12f3449dc9aa9eeb8c66113a46d07b5a01326bf93b7a4195d11654155f4a4a316c289fdcf32ad6521f2ee540010babfd8f444fbf3c064d621c9708524520c972b179bb2L
[+]((p>>128)<<128)=0x362d02c9bf3434042ddfaf08229b9f2bc869d040e94b54ac00ea01bc080b101fc23692f7eac472063664d8fc9f6cfa400000000000000000000000000000000L
[-]long_to_bytes(m).encode('hex')=
```

Factoring with High Bits Known

和 [[61dctf]cry](./[61dctf]cry.md) 一模一样

```python
import time

def matrix_overview(BB, bound):
    for ii in range(BB.dimensions()[0]):
        a = ('%02d ' % ii)
        for jj in range(BB.dimensions()[1]):
            a += '0' if BB[ii,jj] == 0 else 'X'
            a += ' '
        if BB[ii, ii] >= bound:
            a += '~'
        print a

def coppersmith_howgrave_univariate(pol, modulus, beta, mm, tt, XX):
    """
    Coppersmith revisited by Howgrave-Graham
    
    finds a solution if:
    * b|modulus, b >= modulus^beta , 0 < beta <= 1
    * |x| < XX
    """
  
    dd = pol.degree()
    nn = dd * mm + tt

    if not 0 < beta <= 1:
        raise ValueError("beta should belongs in (0, 1]")

    if not pol.is_monic():
        raise ArithmeticError("Polynomial must be monic.")

    polZ = pol.change_ring(ZZ)
    x = polZ.parent().gen()

    gg = []
    for ii in range(mm):
        for jj in range(dd):
            gg.append((x * XX)**jj * modulus**(mm - ii) * polZ(x * XX)**ii)
    for ii in range(tt):
        gg.append((x * XX)**ii * polZ(x * XX)**mm)
    
    BB = Matrix(ZZ, nn)

    for ii in range(nn):
        for jj in range(ii+1):
            BB[ii, jj] = gg[ii][jj]

    BB = BB.LLL()

    new_pol = 0
    for ii in range(nn):
        new_pol += x**ii * BB[0, ii] / XX**ii

    potential_roots = new_pol.roots()
    print "potential roots:", potential_roots

    roots = []
    for root in potential_roots:
        if root[0].is_integer():
            result = polZ(ZZ(root[0]))
            if gcd(modulus, result) >= modulus^beta:
                roots.append(ZZ(root[0]))

    return roots

n = 0x2757328be3c648d2fae0c454f91aa82af78eb83318fac0311ada9bb9a64f30457650ea0e975946f7259eb45821f15528ac24dea1bde15bf8fc5a3e76787e0ac57c74e167d8144861a7608078f6693877dc9c59b53b7f8f6e34c1ce27a3bef79a33df6b04ee170ddf67116e8fe67e3a59a1cba216a41ca23b4ef6c214b5831e1
e = 65537
k_bits = 128
q_high = 0x362d02c9bf3434042ddfaf08229b9f2bc869d040e94b54ac00ea01bc080b101fc23692f7eac472063664d8fc9f6cfa400000000000000000000000000000000
cipher = 0x6ee16b34e2a5523e58e6a63fb44db5f696156246c7ebe27b9af29cd199d2de56dbb77f0e1057a831fb41bd1993c8f18f97de9cc71f9b68518e3817d12f3449dc9aa9eeb8c66113a46d07b5a01326bf93b7a4195d11654155f4a4a316c289fdcf32ad6521f2ee540010babfd8f444fbf3c064d621c9708524520c972b179bb2

qbar = q_high
F.<x> = PolynomialRing(Zmod(n), implementation='NTL'); 
pol = x - qbar
dd = pol.degree()

# PLAY WITH THOSE:
beta = 0.3                             # we should have q >= N^beta
epsilon = beta / 7                     # <= beta/7
mm = ceil(beta**2 / (dd * epsilon))    # optimized
tt = floor(dd * mm * ((1/beta) - 1))   # optimized
XX = ceil(n**((beta**2/dd) - epsilon)) # we should have |diff| < X

# Coppersmith
start_time = time.time()
roots = coppersmith_howgrave_univariate(pol, n, beta, mm, tt, XX)

# output
print "\n# Solutions"
print "we found:", roots
print("in: %s seconds " % (time.time() - start_time))

if roots:
    q = qbar - int(roots[0])
    assert n % q == 0
    p = n / int(q)
    phin = (p - 1) * (q - 1)
    d = inverse_mod(e, phin)
    print "d: ", d
    m = pow(cipher, d, n)
    print hex(int(m))[2:-1]
```

得到 c5fe6513d948337a7500e011aa8f15e89b9f6d4f0ff1ad8e59b224cd845dcbf08499cde779db958c36ee9a9c064ebdf8581389712cec9a51d838998ed0302e79

### 第三关

```
[+]Generating challenge 3
[+]n=0x50c6c6602d0d821cba20f5a5e1499b2ecb75221c9ebcee91337bcf5c8bbc9fe76312079dbaf6a1758604b1510114bed90ddeb556790e62297418290e4a5d1fcb103fc859ca95dbc20c7065ff7ca4f76570eb1da8e89d8be79f7af03df9df1681a984a25a9f344a14105a68a95e7ea565982821204074eb49ec265233ef2f1d69L
[+]e=3
[+]m=random.getrandbits(512)
[+]c=pow(m,e,n)=0xbb3a42e4a0601e36fe3280ee26997d8a7c0fc6a359d27685620ef1199d151b9d94e5c778e4ec8e68685b19af3f0d8935e1cd8a327c5d9b66cfbe06bac36b0a8c5c4f75461846066f62ce3d074355ddebfa36f724881abbd41131e74b31bfacd512c4cb5dabfada212635a146933ecee5685121cff1bf351e4daed48ef084406L
[+]d=invmod(e,(p-1)*(q-1))
[+]d&((1<<512)-1)=0x9412a067ed028cb0779dd075b178c2852a37ab9baa318405003c08706e455aadade03309b65e7a2e0f0e3ec991e92beeafd89ecdcedef9ef03321d5c9ce2f9abL
[-]long_to_bytes(m).encode('hex')=
```

Partial Key Exposure Attack

```python
def partial_p(p0, kbits, n):
    PR.<x> = PolynomialRing(Zmod(n))
    nbits = n.nbits()
    f = 2^kbits*x + p0
    f = f.monic()
    roots = f.small_roots(X=2^(nbits//2-kbits), beta=0.3)  # factor >= n^0.3
    if roots:
        x0 = roots[0]
        p = gcd(2^kbits*x0 + p0, n)
        return ZZ(p)

def find_p(d0, kbits, e, n):
    X = var('X')
    for k in range(1, e+1):
        results = solve_mod([e*d0*X - k*X*(n-X+1) + k*n == X], 2^kbits)
        for x in results:
            p0 = ZZ(x[0])
            p = partial_p(p0, kbits, n)
            if p:
                return p

n = 0x50c6c6602d0d821cba20f5a5e1499b2ecb75221c9ebcee91337bcf5c8bbc9fe76312079dbaf6a1758604b1510114bed90ddeb556790e62297418290e4a5d1fcb103fc859ca95dbc20c7065ff7ca4f76570eb1da8e89d8be79f7af03df9df1681a984a25a9f344a14105a68a95e7ea565982821204074eb49ec265233ef2f1d69
e = 3
d0 = 0x9412a067ed028cb0779dd075b178c2852a37ab9baa318405003c08706e455aadade03309b65e7a2e0f0e3ec991e92beeafd89ecdcedef9ef03321d5c9ce2f9ab
cipher = 0xbb3a42e4a0601e36fe3280ee26997d8a7c0fc6a359d27685620ef1199d151b9d94e5c778e4ec8e68685b19af3f0d8935e1cd8a327c5d9b66cfbe06bac36b0a8c5c4f75461846066f62ce3d074355ddebfa36f724881abbd41131e74b31bfacd512c4cb5dabfada212635a146933ecee5685121cff1bf351e4daed48ef084406
nbits = n.nbits()
kbits = floor(nbits*0.5)
print "bits:", nbits, "kbits:", kbits 

p = find_p(d0, kbits, e, n)
q = n//p
d = inverse_mod(e, (p-1)*(q-1))

print "p: %d" % p
print "q: %d" % q
print "d:", d.hex()
print "m:", hex(int(pow(cipher, d, n)))[2:-1]
```

得到 4035f38e64bea51e507624dec9f96dcccfda6abcf3938daf031c3ea4e8968ffdf7315449359aa2563eeaa84d0d21fb157ed0717d29957295cbc52a0c7707d87e

### 第四关

```
[+]Generating challenge 4
[+]e=3
[+]m=random.getrandbits(512)
[+]n1=0x67bef7b0609dd792178f354691968428a1ab7cc14f668e37a9843a6311c1c3f2344442b19f5e252eae10b5c646c52a625b4814b616bb005f1d218a6493a1f6707c12b568278089ae2e95621748fb094f73f8980ec5be9eda4d14c736d77e22ca81a2be8098d574309153263ffd0b726c9b2989e40ea6714088f9589ce109b2d9L
[+]c1=pow(m,e,n1)=0x3e312eb5aea257e92c6e5a0e08c6706ed3f7de72648ae1083b8203db3ba650c94e99ecd3513dbffa0d220ac4cb642514b2650f32dc046ce5809544c4367b83ab1d2fadd242df50ace20e11d06943b9985d6cf129202775875e1a168df147e8bda62aba7733b835d1e952423ef828e5e99588485a29cef756a24961a87c8638a0L
[+]n2=0x215205ceffed07d0f4c605bcdccba86663f05dd81e28bd9f55d7ee0e9d13834fe053845bcb06a2879798090ef24ef7d8b1a003952251cd1c888cd6c34bcaf24d3fad34418c04dfa5705c63ef8de56dd557f99e3f1dd3b1a91a7b320cf65016d4440367ba0667d3e515469334d48d796f8eeef58c98cfd46404f62d0fc473167L
[+]c2=pow(m,e,n2)=0x54e62ef24a0e3f20560ddd3963580aa059ab82e521d4f0187c2db9f84630f901cb2cffc800cb9a0ff12e64f2c743ac26d56247930b703d2a04f4a965b6d80e8666e6c69238168c706d5652e9005dfa3054006d69eba55ed5ebfaa259456f2b4db481cd0aae8b5f9b2e031d1d500ebbf8d911ecbb1eff605e924f54c6b09cceL
[+]n3=0x52d82623ad8be77f752810b752df43eb79e029c0f473f57107a6fd1728ae41c651fb1a1eb25735eb57ff6da081bff5e72d036203fa0ec1c043cfdd55266ed55c38013fc18d4890552ca22f532aa7e17437711b1b895e55b799d9fc4893809ec8610234ab0afce657770ce725647d7634954c4ddc1f91100a51f59292def090b1L
[+]c3=pow(m,e,n3)=0x3394a11064099e4d3be242b2a367c6c39c5eedfcfcde183d9bca298ae5e2dd2bc27bf22e1dfcdb6abe8e59fd27cd903026c38298d45ec26bd1bf262baf6d90cc7669a9d9ecde6ce5433b507b0a7ef6247649d2c6eaba6f1d334b4363bb441da34508fa6b43e71c333f8741f65ea690cd1c558bef7da12c00c499f42213d71391L
```

中国剩余定理可以计算出 m^3 的值，然后再开个三次方根就好

```python
from libnum import *
n1 = 0x67bef7b0609dd792178f354691968428a1ab7cc14f668e37a9843a6311c1c3f2344442b19f5e252eae10b5c646c52a625b4814b616bb005f1d218a6493a1f6707c12b568278089ae2e95621748fb094f73f8980ec5be9eda4d14c736d77e22ca81a2be8098d574309153263ffd0b726c9b2989e40ea6714088f9589ce109b2d9
c1 = 0x3e312eb5aea257e92c6e5a0e08c6706ed3f7de72648ae1083b8203db3ba650c94e99ecd3513dbffa0d220ac4cb642514b2650f32dc046ce5809544c4367b83ab1d2fadd242df50ace20e11d06943b9985d6cf129202775875e1a168df147e8bda62aba7733b835d1e952423ef828e5e99588485a29cef756a24961a87c8638a0
n2 = 0x215205ceffed07d0f4c605bcdccba86663f05dd81e28bd9f55d7ee0e9d13834fe053845bcb06a2879798090ef24ef7d8b1a003952251cd1c888cd6c34bcaf24d3fad34418c04dfa5705c63ef8de56dd557f99e3f1dd3b1a91a7b320cf65016d4440367ba0667d3e515469334d48d796f8eeef58c98cfd46404f62d0fc473167
c2 = 0x54e62ef24a0e3f20560ddd3963580aa059ab82e521d4f0187c2db9f84630f901cb2cffc800cb9a0ff12e64f2c743ac26d56247930b703d2a04f4a965b6d80e8666e6c69238168c706d5652e9005dfa3054006d69eba55ed5ebfaa259456f2b4db481cd0aae8b5f9b2e031d1d500ebbf8d911ecbb1eff605e924f54c6b09cce
n3 = 0x52d82623ad8be77f752810b752df43eb79e029c0f473f57107a6fd1728ae41c651fb1a1eb25735eb57ff6da081bff5e72d036203fa0ec1c043cfdd55266ed55c38013fc18d4890552ca22f532aa7e17437711b1b895e55b799d9fc4893809ec8610234ab0afce657770ce725647d7634954c4ddc1f91100a51f59292def090b1
c3 = 0x3394a11064099e4d3be242b2a367c6c39c5eedfcfcde183d9bca298ae5e2dd2bc27bf22e1dfcdb6abe8e59fd27cd903026c38298d45ec26bd1bf262baf6d90cc7669a9d9ecde6ce5433b507b0a7ef6247649d2c6eaba6f1d334b4363bb441da34508fa6b43e71c333f8741f65ea690cd1c558bef7da12c00c499f42213d71391

c = solve_crt([c1, c2, c3], [n1, n2, n3])
m = nroot(c, 3)
print(hex(m)[2:])
```

得到 d79d43e4f3c2fb64def2af71e4a6d921f2709e5720f6134f87d62c79cbf0f193e8f2c990fb6abc75981af99f3625dab309ff3412656879a6772fe7e4c0c90ecf

### 第五关

```
[+]Generating challenge 5
[+]n=0x3d87091c028f3aeca1936daf9d7dc3e6331bb70dead002207f76ac508ce8d8c64195a91442898f87d38320e60665723ca3a86d203f1ef6fb3873950636fa020dd10435129f63b1cf8173b4891cb54c47e7cb95ca7301d9bd7d8f7343e3786392d722dcbeae20bb46496a90e4936cad52cd3520947dccc2ce5a63facf9ac73933L
[+]e=3
[+]m=random.getrandbits(512)
[+]c=pow(m,e,n)=0x824bbc8399838a26b72b880eee19b90fb6277092d36d3fc4182b41ba8847755c988f05f8a928ea0ed819e1071e256038ac6c683d548ee0f944b0cf359d9fb12a8d3409f50e6ca5b09df9f08e3eb46cb7d995e0eaa1e5b5b953426a645e063a80543ee1d8533bce0ea09a3aef3e3e8f7a64bc164e6125afa8b891dc6416b71f4L
[+]x=pow(m+1,e,n)=0x30bf367cf47a6c27cab5ba7b5918e926183cf980e8447cf1928e29b84ac1e142548833c18be0b92ab73b3698d6d9e623a7fcab00debcb73d7b3bc3df3b287f6e4977bf1db0ad5d67acaf596a7b089957d18ce65ea3d125e319f8515325c9817f06ffff0770bb4217e4ea73f0db05e6b0b653a8abb1df065a2425dd36798597f5L
```

(m+1)^3 - m^3 = 3m^2 + 3m + 1

所以我们要解方程 3m^2 + 3m + 1 + c - x = in

```python
from libnum import *
n = 0x3d87091c028f3aeca1936daf9d7dc3e6331bb70dead002207f76ac508ce8d8c64195a91442898f87d38320e60665723ca3a86d203f1ef6fb3873950636fa020dd10435129f63b1cf8173b4891cb54c47e7cb95ca7301d9bd7d8f7343e3786392d722dcbeae20bb46496a90e4936cad52cd3520947dccc2ce5a63facf9ac73933
e = 3
c1 = 0x824bbc8399838a26b72b880eee19b90fb6277092d36d3fc4182b41ba8847755c988f05f8a928ea0ed819e1071e256038ac6c683d548ee0f944b0cf359d9fb12a8d3409f50e6ca5b09df9f08e3eb46cb7d995e0eaa1e5b5b953426a645e063a80543ee1d8533bce0ea09a3aef3e3e8f7a64bc164e6125afa8b891dc6416b71f4
c2 = 0x30bf367cf47a6c27cab5ba7b5918e926183cf980e8447cf1928e29b84ac1e142548833c18be0b92ab73b3698d6d9e623a7fcab00debcb73d7b3bc3df3b287f6e4977bf1db0ad5d67acaf596a7b089957d18ce65ea3d125e319f8515325c9817f06ffff0770bb4217e4ea73f0db05e6b0b653a8abb1df065a2425dd36798597f5
for i in range(1000000):
    d = 9 - 12 * (1 + c1 - c2 - i * n)
    if d >= 0:
        r = nroot(d, 2)
        if r * r == d and (r - 3) % 6 == 0:
            m = (r - 3) // 6
            print(hex(m)[2:])
            exit(0)
```

得到 ec937f11f399739045529c9d3f91482bde899154c3ec79c543881dabac49ce5c7ac456677bf6e8f23aa96bd8e7a57089c1223458a8aedeb9e34c15e8a4c13d79

### 第六关

```
[+]Generating challenge 6
[+]n=0xbadd260d14ea665b62e7d2e634f20a6382ac369cd44017305b69cf3a2694667ee651acded7085e0757d169b090f29f3f86fec255746674ffa8a6a3e1c9e1861003eb39f82cf74d84cc18e345f60865f998b33fc182a1a4ffa71f5ae48a1b5cb4c5f154b0997dc9b001e441815ce59c6c825f064fdca678858758dc2cebbc4d27L
[+]d=random.getrandbits(1024*0.270)
[+]e=invmod(d,phin)
[+]hex(e)=0x11722b54dd6f3ad9ce81da6f6ecb0acaf2cbc3885841d08b32abc0672d1a7293f9856db8f9407dc05f6f373a2d9246752a7cc7b1b6923f1827adfaeefc811e6e5989cce9f00897cfc1fc57987cce4862b5343bc8e91ddf2bd9e23aea9316a69f28f407cfe324d546a7dde13eb0bd052f694aefe8ec0f5298800277dbab4a33bbL
[+]m=random.getrandbits(512)
[+]c=pow(m,e,n)=0xe3505f41ec936cf6bd8ae344bfec85746dc7d87a5943b3a7136482dd7b980f68f52c887585d1c7ca099310c4da2f70d4d5345d3641428797030177da6cc0d41e7b28d0abce694157c611697df8d0add3d900c00f778ac3428f341f47ecc4d868c6c5de0724b0c3403296d84f26736aa66f7905d498fa1862ca59e97f8f866cL
```

d < n^0.292，所以知道这是 Boneh and Durfee attack

```
RsaCtfTool.py -n 131220266260257726784563231632524259192287425907086322240287919206747189418737213895961710667681482673722441200160183176293978060339414789036626722383395900282828294720181753868486347157723063536765531909832527500306344776277799499540975525817191918248937022909390496214986596994678029680483503994082294517031 -e 12250979346409384956125275671247973080604847642674548888849819998762519358016137217088961459400450576457774414141280533854852114652995729671199080526581735941182072594417599294988588904592344540135791955381825156414235149257022130505765538228100578450610911533988352629868056263764730724464210431860114338747 --uncipher 623536275773848797625345143423756942738800555689469442646156337235573818950302815020869750021452422700360662007989420533592928113237355881759597338980518907512212388027308240701592412909951718177311040739922920814188244062815113169310648979277815399276763821181565481549531207726241880079411474519898949228 --attack boneh_durfee --verbose
```

（注意，要稍微修改一下 RsaCtfTool.py 使其支持其目录下的 `boneh_durfee.sage` 文件，这样才能调用我上面的这个命令）

得到

```
b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00k;\xb0\xcd\xc7*\x7f,\xe8\x99\x02\xe1\x9d\xb0\xfb,\x05\x14\xc7ht\xb2\xcaA\x13\xb8nm\xc1(\xd4L\xc8Y(=\xb4\xca\x8b\x0b]\x9e\xe3P2\xae\xc8\xcc\x8b\xb9n\x8c\x11Ty\x15\xfc\x9e\xf0Z\xa2\xd7+('
```

也就是 6b3bb0cdc72a7f2ce89902e19db0fb2c0514c76874b2ca4113b86e6dc128d44cc859283db4ca8b0b5d9ee35032aec8cc8bb96e8c11547915fc9ef05aa2d72b28

## 总结

```python
from pwn import *
import hashlib
import struct
import random

con = remote('47.97.215.88', 12300)

def cha0():
    print '=========== challenge 0 ==========='
    t = con.recvline()
    sha = con.recvline().split('=')[1].strip()
    skr = con.recvline().split('=')[1].strip()
    con.recvuntil('=')
    print 'sha1(sha):', sha
    print 'skr[:5]:  ', skr

    def rg():
        a = list(range(256))
        random.shuffle(a)
        return a

    for i in rg():
        for j in rg():
            for k in rg():
                t = skr + format(i, '02x') + format(j, '02x') + format(k, '02x')
                if hashlib.sha256(t.decode('hex')).hexdigest() == sha:
                    print 'skr:      ', t
                    con.send(t + '\n')
                    return

answer = [
'a6717a7ee57e329b717a29f2a9fc503641bf481d5c24198fe2f9c15dc3ddee11a184c46b0065b54fa332aebfed130d7d44da249ec51d27e4ead4a08036e8c72c',
'c5fe6513d948337a7500e011aa8f15e89b9f6d4f0ff1ad8e59b224cd845dcbf08499cde779db958c36ee9a9c064ebdf8581389712cec9a51d838998ed0302e79',
'4035f38e64bea51e507624dec9f96dcccfda6abcf3938daf031c3ea4e8968ffdf7315449359aa2563eeaa84d0d21fb157ed0717d29957295cbc52a0c7707d87e',
'd79d43e4f3c2fb64def2af71e4a6d921f2709e5720f6134f87d62c79cbf0f193e8f2c990fb6abc75981af99f3625dab309ff3412656879a6772fe7e4c0c90ecf',
'ec937f11f399739045529c9d3f91482bde899154c3ec79c543881dabac49ce5c7ac456677bf6e8f23aa96bd8e7a57089c1223458a8aedeb9e34c15e8a4c13d79',
'6b3bb0cdc72a7f2ce89902e19db0fb2c0514c76874b2ca4113b86e6dc128d44cc859283db4ca8b0b5d9ee35032aec8cc8bb96e8c11547915fc9ef05aa2d72b28',
]

def cha(i, ans):
    print '=========== challenge %d ===========' % i
    con.recvuntil("long_to_bytes(m).encode('hex')=")
    print 'ans:', ans
    con.send(ans + '\n')

def wait():
    print con.recvall()

cha0()
for i in range(6):
    cha(i + 1, answer[i])
wait()
```

## 答案

flag{qwb_2020_welcome}
