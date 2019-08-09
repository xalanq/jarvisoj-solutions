## 描述

五层密码，好复杂

来源：sunnyelf

[Complicated Crypto.7z.800733a9251cab311bb5f34f15386008](./assets/Complicated%20Crypto.7z.800733a9251cab311bb5f34f15386008)

## 题解

拿到文件，是一个加密的 7z，看到 3 个文件这么小（以及还给了文件名提示），一眼就知道是 CRC32 破解小文件。

参考之前做过的题 [神秘的压缩包](./神秘的压缩包.md)，可以轻松过掉第一关：

```
$ 7z l -slt Complicated\ Crypto.7z.800733a9251cab311bb5f34f15386008 | grep Path
Path = Complicated Crypto.7z.800733a9251cab311bb5f34f15386008
Path = pwd1.txt
Path = pwd2.txt
Path = pwd3.txt
Path = CRC32 Collision.7z
7z l -slt Complicated\ Crypto.7z.800733a9251cab311bb5f34f15386008
$ 7z l -slt Complicated\ Crypto.7z.800733a9251cab311bb5f34f15386008 | grep CRC
CRC = 7C2DF918
CRC = A58A1926
CRC = 4DAD5967
Path = CRC32 Collision.7z
CRC = 06B072C5
```

将以下内容保存到 `crc32.txt`

```
7C2DF918:00000000
A58A1926:00000000
4DAD5967:00000000
```

开始爆破

```
$ hashcat -m 11500 -a 3 --keep-guessing crc32.txt ?a?a?a?a?a?a -o key.txt
```

过了几十分钟，才爆破完

好 tm 的多。。。写一份脚本生成所有可能秘钥：

```python
f = open('key.txt', 'r')

a = [
    '7C2DF918'.lower(),
    'A58A1926'.lower(),
    '4DAD5967'.lower(),
]

mp = dict()
for k in a:
    mp[k] = []

for l in f.readlines():
    data = l.strip().split(':')
    crc = data[0]
    raw = data[-1]
    if len(raw) != 6:
        raw = bytes.fromhex(raw[5:-1]).decode()
    mp[crc].append(raw)

tot = []

def gen(i, passwd):
    if i == len(a):
        tot.append(passwd)
        return
    for c in mp[a[i]]:
        gen(i + 1, passwd + c)

gen(0, '')
open('wordlist.txt', 'w').write('\n'.join(tot))
```

然后大概翻了一些，肉眼查看了下，根据 CTF 的命名习惯和本题的意义，大致确定：

```
7C2DF918：_CRC32
A58A1926：_i5_n0
4DAD5967：t_s4f3
```

合在一起就是 `_CRC32_i5_n0t_s4f3`。试了一下，解压成功~得到 `CRC32 Collision.7z`，里边有几个文件

```
ciphertext.txt  'Find password.7z'   keys.txt   tips.txt
```

tips 的内容是

```
你知道维吉尼亚密码吗？
我们给了keys.txt，唯一的密钥就在其中，那么解密ciphertext.txt里的密文吧！
解压密码就在明文里，祝你好运！
```

先上这个网站 https://www.guballa.de/vigenere-solver 破解了下，得到一个不完整的内容 

```
the getenere cilger is a method of thensating alphamagic text bu tsing a series of schbycent caesar nechers basac on the letters ou u mashord it is a sticle form ob oolyalphabetic hodonttution so plofword is vefenere cipher fucha
```

我们可以确定前面一句话的几个单词 `cipher is a method of`，然后再来写个代码爆破一下

```python
chars = 'abcdefghijklmnopqrstuvwxyz'
key_word = 'cipher is a method of'

def shift(char, key, rev = False):
    if not char in chars:
        return char
    if rev:
        return chars[(chars.index(char) - chars.index(key)) % len(chars)]
    else:
        return chars[(chars.index(char) + chars.index(key)) % len(chars)]

def decrypt(text, key):
    r = ''
    i = 0
    for c in text:
        if c == ' ':
            r += ' '
            continue
        r += shift(c, key[i % len(key)], True)
        i += 1
    return r

cipher = open('ciphertext.txt', 'r').read()

for l in open('keys.txt', 'r').readlines():
    key = l.strip().lower()
    text = decrypt(cipher, key)
    if key_word in text:
        print(key.upper())
        print(text)
```

得到

```
the vigenere cipher is a method of encrypting alphabetic text by using a series of different caesar ciphers based on the letters of a keyword it is a simple form of polyalphabetic substitution so password is vigenere cipher funny
```

密码是 `vigenere cipher funny`。解压后得到新的一关：

```
恭喜!

现在我们遇到一个问题,我们有一个zip文件,但我们不知道完整的解压密码。
幸好我们知道解压密码的一部分sha1值。
你能帮我们找到的密码吗?

不完整的密码："*7*5-*4*3?"  *代表可打印字符

不完整的sha1："619c20c*a4de755*9be9a8b*b7cbfa5*e8b4365*"  *代表可打印字符

人生苦短，我用Python。
```

直接暴力没话说

```python
import hashlib, random
cp = '619c20c*a4de755*9be9a8b*b7cbfa5*e8b4365*'.replace('*', '')
rg1 = list(range(32, 127))
random.shuffle(rg1)
rg2 = list(rg1)
random.shuffle(rg2)
rg3 = list(rg2)
random.shuffle(rg3)
rg4 = list(rg3)
random.shuffle(rg4)
cnt = 0
tot = len(rg1)
for a in rg1:
    cnt += 1
    print(cnt, '/', tot)
    for b in rg2:
        for c in rg3:
            for d in rg4:
                p = '{}7{}5-{}4{}3?'.format(chr(a), chr(b), chr(c), chr(d))
                s = hashlib.sha1(p.encode()).hexdigest()
                if s[0:7] + s[8:15] + s[16:23] + s[24:31] + s[32:39] == cp:
                    print(p)
                    print(s)
                    exit(0)
```

不一会，得到 `I7~5-s4F3?`。解压又到了新的一关，有一个加密的 7z 压缩包（头也加密了的那种）和一个文本：

```
Hello World ;-)
MD5校验真的安全吗？
有没有两个不同的程序MD5却相同呢？
如果有的话另一个程序输出是什么呢？
解压密码为单行输出结果。
```

真的没看懂这是啥意思。。。两个相同 MD5 不同内容的程序多了去了。。。你要我找哪个？？？

网上搜了一下这两句话，找到了这篇文章 https://blog.csdn.net/xiaofei0859/article/details/54924123 ，里面给了两个程序。。。一个 Hello World，一个 Good Bye。

可能要找的就是这个程序 http://www.win.tue.nl/hashclash/SoftIntCodeSign/GoodbyeWorld-colliding.exe ，下载并运行，输出了 `Goodbye World :-(`，尝试了下，密码正确。

最后一关得到一个 RSA 的公钥和密文，先看看公钥的内容

```
$ RsaCtfTool.py --dumpkey --key rsa_public_key.pem 
[*] n: 460657813884289609896372056585544172485318117026246263899744329237492701820627219556007788200590119136173895989001382151536006853823326382892363143604314518686388786002989248800814861248595075326277099645338694977097459168530898776007293695728101976069423971696524237755227187061418202849911479124793990722597
[*] e: 354611102441307572056572181827925899198345350228753730931089393275463916544456626894245415096107834465778409532373187125318554614722599301791528916212839368121066035541008808261534500586023652767712271625785204280964688004680328300124849680477105302519377370092578107827116821391826210972320377614967547827619
```

这。。。过分了，这么大的 e 我头一回见。。。还是和 n 一样长，有毒吧。。那 d 会不会就很小。。

不管了，用 `RsaCtfTool.py` 先跑一波。woc 秒出

```
$ RsaCtfTool.py --publickey rsa_public_key.pem --uncipherfile flag.enc
[+] Clear text : b'\x00\x02\xb3\xf3\xc6W8\xb5\x81S/cwr\xe8\xd3\xb5Cf\xe4\xe5w\x81h\t\x82\x8cd\x85D}\xec7\xec!\xe4;\x89\xb3w\xa4Uf\xf5\xd9%\xcb\x96\x85\x10\x11B\x9a<"QS\x05\x84\x80{\xb1.\x82\xcc\x1c\xf6\x87z@\x91\x9e\xf6h\xe7\xa1\x8f\x96\x9d%&\xa4\xcd\xf0\'\x16J\xf4!\x9c\'h8!Y\xa1o(H\xea}\x00flag{W0rld_Of_Crypt0gr@phy}'
```

后来查了题解，发现这是 Wiener's attack，学到了 orz

## 答案

flag{W0rld_Of_Crypt0gr@phy}
