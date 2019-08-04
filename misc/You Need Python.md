## 描述

人生苦短我用Python。

题目：[you_need_python.zip.74d515955b9aa607b488a48437591a14](./assets/you_need_python.zip.74d515955b9aa607b488a48437591a14)

## 题解

print 了一下没啥意义，将 `marshal.loads` 括号内的内容二进制写到一个文件里，应该是 `.pyc`，尝试运行，发现报错

```
RuntimeError: Bad magic number in .pyc file
```

查了一下，应该是编译版本问题，具体看 

- https://stackoverflow.com/questions/514371/whats-the-bad-magic-number-error
- https://shankaraman.wordpress.com/tag/how-to-fix-runtimeerror-bad-magic-number-in-pyc-file/

但是我用 `010 Editor` 尝试改一下，发现还是没用。

再翻了一下 ctf-wiki，看到了 https://ctf-wiki.github.io/ctf-wiki/misc/other/pyc-zh/

> 最开始 4 个字节是一个 Maigc int, 标识此 pyc 的版本信息  
  接下来四个字节还是个 int, 是 pyc 产生的时间

这个文件一样都没有，那就尝试直接加上去？

```python
import imp, struct
data = bytearray(imp.get_magic())
data += struct.pack('<i', 0)
data += a # a 就是 marshal.loads() 括号内包裹的内容

with open('what.pyc', 'wb') as f:
    f.write(data)
```

然后便能执行 `python what.pyc` 了。然后用 `uncompyle6` 反编译成源代码，得到

```python
# uncompyle6 version 3.3.5
# Python bytecode 2.7 (62211)
# Decompiled from: Python 2.7.16 (default, Apr  6 2019, 01:42:57) 
# [GCC 8.3.0]
import hashlib

def sha1(string):
    return hashlib.sha1(string).hexdigest()


def calc(strSHA1):
    r = 0
    for i in strSHA1:
        r += int('0x%s' % i, 16)

    return r


def encrypt(plain, key):
    keySHA1 = sha1(key)
    intSHA1 = calc(keySHA1)
    r = []
    for i in range(len(plain)):
        r.append(ord(plain[i]) + int('0x%s' % keySHA1[(i % 40)], 16) - intSHA1)
        intSHA1 = calc(sha1(plain[:i + 1])[:20] + sha1(str(intSHA1))[:20])

    return ('').join(map(lambda x: str(x), r))


if __name__ == '__main__':
    key = raw_input('[*] Please input key:')
    plain = raw_input('[*] Please input flag:')
    encryptText = encrypt(plain, key)
    cipherText = '-185-147-211-221-164-217-188-169-205-174-211-225-191-234-148-199-198-253-175-157-222-135-240-229-201-154-178-187-244-183-212-222-164'
    if encryptText == cipherText:
        print '[>] Congratulations! Flag is: %s' % plain
        exit()
    else:
        print '[!] Key or flag is wrong, try again:)'
        exit()
```

好吧，接着我们来破解给的 key 文件，根据文件名提示，搜索一下 rfc4042，学到了一手 utf-9....wdnmd

```python
> import utf9
> a = open('key_is_here_but_do_you_know_rfc4042', 'r').read()
> print(utf9.utf9decode(a))
_____*((__//__+___+______-____%____)**((___%(___-_))+________+(___%___+_____+_______%__+______-(______//(_____%___)))))+__*(((________/__)+___%__+_______-(________//____))**(_*(_____+_____)+_______+_________%___))+________*(((_________//__+________%__)+(_______-_))**((___+_______)+_________-(______//__)))+_______*((___+_________-(______//___-_______%__%_))**(_____+_____+_____))+__*(__+_________-(___//___-_________%_____%__))**(_________-____+_______)+(___+_______)**(________%___%__+_____+______)+(_____-__)*((____//____-_____%____%_)+_________)**(_____-(_______//_______+_________%___)+______)+(_____+(_________%_______)*__+_)**_________+_______*(((_________%_______)*__+_______-(________//________))**_______)+(________/__)*(((____-_+_______)*(______+____))**___)+___*((__+_________-_)**_____)+___*(((___+_______-______/___+__-_________%_____%__)*(___-_+________/__+_________%_____))**__)+(_//_)*(((________%___%__+_____+_____)%______)+_______-_)**___+_____*((______/(_____%___))+_______)*((_________%_______)*__+_____+_)+___//___+_________+_________/___
```

这。。。。啥玩意。。有毒吧。。完全没头绪。看题解吧

题解说这是个下划线的算式 orz，下划线长度代表数字 orz

```python
import utf9
a = open('key_is_here_but_do_you_know_rfc4042', 'r').read()
_ = 1
__ = 2
___ = 3
____ = 4
_____ = 5
______ = 6
_______ = 7
________ = 8
_________ = 9
__________ = 0
print(hex(eval(utf9.utf9decode(a)))[2:].decode('hex'))
```

得到 key 为 `I_4m-k3y`

现在来看加密算法。过于简单，暴力枚举每一位就好。

```python
import hashlib

def sha1(string):
    return hashlib.sha1(string).hexdigest()

def calc(strSHA1):
    r = 0
    for i in strSHA1:
        r += int('0x%s' % i, 16)
    return r

def encrypt(plain, key):
    keySHA1 = sha1(key)
    intSHA1 = calc(keySHA1)
    r = []
    for i in range(len(plain)):
        r.append(ord(plain[i]) + int('0x%s' % keySHA1[(i % 40)], 16) - intSHA1)
        intSHA1 = calc(sha1(plain[:i + 1])[:20] + sha1(str(intSHA1))[:20])
    return ('').join(map(lambda x: str(x), r))

key = 'I_4m-k3y'
cipherText = '-185-147-211-221-164-217-188-169-205-174-211-225-191-234-148-199-198-253-175-157-222-135-240-229-201-154-178-187-244-183-212-222-164'
cipher = []
for i in range(0, len(cipherText), 4):
    cipher.append(cipherText[i:i+4])

flag = ''
keySHA1 = sha1(key)
intSHA1 = calc(keySHA1)
for i in range(len(cipher)):
    k = ''
    for j in range(32, 127):
        if int(cipher[i]) == j + int('0x%s' % keySHA1[(i % 40)], 16) - intSHA1:
            k = chr(j)
            break
    flag += k
    intSHA1 = calc(sha1(flag)[:20] + sha1(str(intSHA1))[:20])

print(flag)
```

运行得到 flag

## 答案

flag{Lif3_i5_5h0r7_U_n33d_Py7h0n}
