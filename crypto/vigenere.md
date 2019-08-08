## 描述

题目来源：TWCTF2016

[vigenere_300.zip.f79ac4bffe76f0fa1ad6afa315393840](./assets/vigenere_300.zip.f79ac4bffe76f0fa1ad6afa315393840)

## 题解

简单的移位加密，但问题在于秘钥是随机的，长度也挺长。

唯一的突破口在 Base64，好吧，不会，看题解。

学到了。

题目里用到的加密是 Vigenere 密码，就是明文和秘钥同时移动相同距离，然后查表加密（表是由某一行或某一列循环生成的）。

详细可以参考 https://ctf-wiki.github.io/ctf-wiki/crypto/classical/polyalphabetic-zh/#vigenere

破解 Vigenere 密码有一个叫 Kasiski 测试法的算法，这个算法是用来求秘钥的长度的。我在这里简要摘抄一下上面链接的例子

卡西斯基试验是基于类似 the 这样的常用单词有可能被同样的密钥字母进行加密，从而在密文中重复出现。例如，明文中不同的 CRYPTO 可能被密钥 ABCDEF 加密成不同的密文：

```
密钥：ABCDEF AB CDEFA BCD EFABCDEFABCD
明文：CRYPTO IS SHORT FOR CRYPTOGRAPHY
密文：CSASXT IT UKSWT GQU GWYQVRKWAQJB
```

此时明文中重复的元素在密文中并不重复。然而，如果密钥相同的话，密文自然就相同了（使用密钥 ABCD）：

```
密钥：ABCDAB CD ABCDA BCD ABCDABCDABCD
明文：CRYPTO IS SHORT FOR CRYPTOGRAPHY
密文：CSASTP KV SIQUT GQU CSASTPIUAQJB
```

此时卡西斯基试验就能产生效果。对于更长的段落此方法更为有效，因为通常密文中重复的片段会更多。如通过下面的密文就能破译出密钥的长度：

```
密文：DYDUXRMHTVDVNQDQNWDYDUXRMHARTJGWNQD
```

其中，两个 DYDUXRMH 的出现相隔了 18 个字母。因此，可以假定密钥的长度是 18 的约数，即长度为 18、9、6、3 或 2。而两个 NQD 则相距 20 个字母，意味着密钥长度应为 20、10、5、4 或 2。取两者的交集，那么基本确定密钥长度为 2。（但这并不是绝对正确的，只是概率比较大而已）

所以在本题里，我们可以找一下长度 >= 3 的重复密文段。

```python
import math

def kasiski(s, l):
    res = 0
    for i in range(len(s) - l):
        w = s[i:i + l]
        j = s[i + l:].find(w)
        if j != -1:
            res = math.gcd(res, l + j)
    return res

s = open('encrypted.txt', 'r').read().strip()
print(kasiski(s, 3)) 
```

得到 12，那么本题的密钥长度可能为 6 或者 12。如何快速检验密钥是否合法呢？准确来说，由于密钥属于 ascii 里的正常字符，一旦 Base64 转换回来出现了不正常的字符，那么密钥就不可能正确。在这里，正常的 ascii 字符我们可以理解为 32 <= c <= 126 （且包括换行符 10 与 13）的字符（查表可知）

我们再来了解一些 Base64 的原理，其实就是将 6 个 bit 对应到 64 个字符，也就是每 3 个字节就替换成 4 个字节的可见字符（见 https://ctf-wiki.github.io/ctf-wiki/misc/encode/computer-zh/#base ）。要是 4 个字符的密钥正确，那么相应的 3 个字节一定为可见字符。

因此我们将密钥分成 3 份，每份 4 个字符，然后依次来暴力每一份的字符。对于每一份，1 个字符不能确定啥，2 个字符能确定明文的 1 个字符为可见字符，3 个字符可以确定明文的 2 个字符为可见字符，4 个字符可以确定 3 个。

```python
from base64 import b64encode, b64decode

chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/'
encrypted = open('encrypted.txt', 'r').read().strip()

def shift(char, key, rev = False):
    if not char in chars:
        return char
    if rev:
        return chars[(chars.index(char) - chars.index(key)) % len(chars)]
    else:
        return chars[(chars.index(char) + chars.index(key)) % len(chars)]

def decrypt(encrypted, key):
    encrypted = ''.join([shift(encrypted[i], key[i % len(key)], True) for i in range(len(encrypted))])
    return b64decode(encrypted)

def crack(m, key_words):
    print('len of password =', m)

    is_valid_char = lambda s: all([c == 10 or c == 13 or (32 <= c and c <= 126) for c in s])
    
    def is_valid_block(pwd, num, block, skip):
        start = 3 * block
        plain = decrypt(encrypted, pwd)
        for i in range(start, len(plain), skip):
            if not is_valid_char(plain[i:i+num]):
                return False
        return True
    
    blocks = (m + 3) // 4
    skip = 3 * blocks
    candidate = []
    for k in range(blocks):
        candidate.append([])
        for a in chars:
            for b in chars:
                if not is_valid_block(a + b + 'aa', 1, k, skip):
                    continue
                for c in chars:
                    if not is_valid_block(a + b + c + 'a', 2, k, skip):
                        continue
                    for d in chars:
                        if not is_valid_block(a + b + c + d, 3, k, skip):
                            continue
                        candidate[k].append(a + b + c + d)
    print('Candidate: ')
    print(candidate)
    print('============\nresult: ')
    pwd_list = []

    def gen(i, pwd):
        if i == len(candidate):
            pwd_list.append(pwd)
            return
        for c in candidate[i]:
            gen(i + 1, pwd + c)

    gen(0, '')

    for pwd in pwd_list:
        plain = decrypt(encrypted, pwd).decode('ascii')
        if all([key in plain for key in key_words]):
            print('=================')
            print(pwd)
            print(plain)

# crack(6, [])
crack(12, ['TWCTF{', 'the flag'])
```

逐渐尝试的去修改 `key_words` 来减少可能性，最终得到

```
len of password = 12
Candidate: 
[['shA1', 'shA2', 'shA3', 'shA4', 'shA5', 'shA6'], ['I8Hn', 'I8Hq', 'I8HE', 'I8HH', 'I8HS', 'I8HT', 'I8HU', 'I8H1', 'I8H4', 'I8H5', 'I8H8', 'I8H+', 'I8H/'], ['XLFR', 'XLFS', 'XLFT', 'XLFV', 'XLFW', 'XLFX', 'XLFY', 'XLF5']]
============
result: 
=================
shA6I8HUXLFY
SKK is a Japanese Input Method developed by Sato Masahiko. Original SKK targets Emacs. However, there are various SKK programs that works other systems such as SKKFEP(for Windows), AquaSKK(for MacOS X) and eskk(for vim).
OK, the flag is TWCTF{C14ss1caL CiPhEr iS v3ry fun}.
```

## 答案

TWCTF{C14ss1caL CiPhEr iS v3ry fun}
