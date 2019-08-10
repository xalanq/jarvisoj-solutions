## 描述

nc pwn2.jarvisoj.com 9891

[cry.py.2e3eb0f65795f3a581ea904dbd069e3d](./assets/cry.py.2e3eb0f65795f3a581ea904dbd069e3d)

## 题解

RSA

看了下代码，意思是服务器告诉我们 n（随机，n = p * q） 和 e（固定为 65537），然后我们能发送一串信息 m 过去，服务器会告诉我们 m^d mod n 是多少，还会告诉我们 p 的 180 个位（高位到低，p 是 2^1024 范围内的质数），然后将 Flag 用 AES 加密，而 AES 的密钥和 IV 都是随机的，会以 t^e mod n 的方式告诉我们。所以我们的目标就是构造这个 m，从而达到解出 AES 密钥的目的。

思考了一下无果。不过这题特殊的地方在于给了我们 p 的高 180 位，应该是某种分解算法吧，尝试去 ctf-wiki 看一下。于是找到了这个

- https://github.com/ashutosh1206/Crypton/blob/master/RSA-encryption/Attack-Coppersmith/README.md
- https://ctf-wiki.github.io/ctf-wiki/crypto/asymmetric/rsa/rsa_coppersmith_attack-zh/#factoring-with-high-bits-known

代码参考 https://github.com/mimoo/RSA-and-LLL-attacks/blob/master/coppersmith.sage 

用 sage 运行以下代码。

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

n = 0x132473c1176ffc27858e8547b7c40898b8346132fd00082d0f66016554a445adb710a57b6a6479ea50847ff7cd5eb6be316b789d97ce0fcf778dbec7a55e83e655bbbbacc913b86b195a364adc6bee77e656e62b95b73071e64d133a9b82352171567a449905f3b343250cd1fe9c4a43eaebd892051e48b2b8b4ca7ac2d50aa64283ba60ac23d51864e50d584b30375d22e8c490ee189453bdcec863939ffd9ce7b79f28c441395421336662fa23c72bb3117288d160fcfa90e138f5937534f7d44854ccf0e5f77764540f2490e6de8c039b7fee2013e4bb64489824a850d73e9e8fe9add4bcea057a8e5b91c8f8872081f583b38856edd6c5db628f56cbb00d
e = 0x10001
q_bits = 1024
q_high = 0x935b3a366ef1c4d321074e49f5c1ecf2cc2652d0939c6c5f7488779a997342691217232eb91d716198a4fb38d42c4f74f4a905784ee6cb6f763b77b3889c95d3fe03d06e45a95876d0ccdcd87632a0714af44ce4b1f6f27eb215
ciphers = [0x71cc2eb875fd7f6d60261b4bc609c28b0823b2c210d59ed52b91e1614354543e08ae60b7546b141aa77fc75ce0b82239bdc2d278fd8f4d8fe41e64bdc83827ac533ff9cde519253dfa74f43a452b87a1f2aa7f9d2fa0fbf69297c254509b06eed9b3f24bd34c7e8d035052c7a428427e3b61464ef16c446cdad18ece05a5595cee098d49c4fa2ab5b4c65230af1b715a14cb32af70c315c0a8ca1b59bbffd83889703bf55a4f45bc42a73d999b00aedbd192f3d190d6901a208b36694fd71634d0e1eb0e93d94386b54dbaeb4457c80b6685d369a9356e41f53d95888db902d161b053770c78ff68e94431ba223439d4ec5c767f728df7f04b06e5ad67cd99b, 0x1025f23d1d603a320ef3b07cb43bf23f458754181b0b45d487af494a4e91b373f9d4f57d0dd0fd2056973402b139b80550ee3323e6929740e1351724500a98d8f84e88490966910f1a7105301a1ea633f2763545bee344565ab4846379d2466d375ed4b9f1dd3314a6dc46617ef6745b1fc178e57efcf47e78a21aad5512c7e54deaa5cf50303d2b3f82d0b4064f24fa6b9bb7b3ecdaa1b215eecf80e7d7b05e148851f96172d83ecf80935f47b837df8db19cdd184b992c0e960596010495baab09da67e0c1ac693753fb3eaca40896687d931feaec3f3ffe57e87857a642e4170fa85125298faa51cf5c21af0d34cb352e759f7637520d6e454d04f3fae6ef]

k_bits = q_bits - q_high.nbits()
qbar = q_high << k_bits
F.<x> = PolynomialRing(Zmod(n), implementation='NTL'); 
pol = x - qbar
dd = pol.degree()

# PLAY WITH THOSE:
beta = 0.5                             # we should have q >= N^beta
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
    for i in range(len(ciphers)):
        ck = pow(ciphers[i], d, n)
        ck = hex(int(ck))[:-1]
        print i, "#:", ck
```

得到

```
we found: [-3215605411470335614512358680663993005467668193344248309425939682814221921072110680581175557]
in: 0.0713548660278 seconds 
d:  1207274396250126695692824893859133679383496043690725057284318538280805449085460980153524445573009384034398687776515214182897962726049402830798675645220902145466188846619221262060463474294919677860380422135925208108827088206529077259714460269755417844664907144796565234565909197600997919450703688136998138100307460886693643469061917267926656918164946085196102954743080305293558526731647046353008646186305625780333977988927191641429631539029512134039854469661653861946049433844954350568519247394833541392037402630868384036469547739669700978454461585728013204382049517739851017221174073898392876791845136169894018817473
0 #: 0x9aef0dc9a26600703b78ce240de74637
1 #: 0xb9a0f07babf68ce286683a79643359aa
```

再解密即可

```python
import base64
from Crypto.Cipher import AES
flag = 'raAwkwJytHB8qKLgLhySEpMSTjnqVPbllkfS+kmGRxA='
key = '0x9aef0dc9a26600703b78ce240de74637'[2:]
iv = '0xb9a0f07babf68ce286683a79643359aa'[2:]
cipher = AES.new(bytes.fromhex(key), AES.MODE_CBC, bytes.fromhex(iv))
flag = cipher.decrypt(base64.b64decode(flag))
print(flag)
```

## 答案

flag{ok_this_is_a_flag_rsa}
