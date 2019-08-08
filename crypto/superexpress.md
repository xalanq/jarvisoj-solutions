## 描述

题目来源：TWCTF2016

[superexpress.7z.b25a070afa73bdb3a2e425f8a68498ee](./assets/superexpress.7z.b25a070afa73bdb3a2e425f8a68498ee)

## 题解

给出的加密式子是这个

```python
for c in flag:
    c = ord(c)
    for a, b in zip(key[0:n], key[n:2*n]):
        c = (ord(a) * c + ord(b)) % 251
    encrypted += '%02x' % c
```

很显然，对于一个字符 c，将第 3 行代码的循环展开，可知加密后的得到的字符 d 满足 d = (pc + q) % 251，其中 p 和 q 是个常数（key 确定的情况下）

而我们已知了部分密文，因此我们爆破 p 和 q 就好，然后验证就好。由于爆破 p 和爆破 inv(p) 和 -q 本质是一样的，所以直接爆破等式 c = (ac + b) % 251 的式子就可以了。

MD 被坑了，代码里那个 `CENSORED` 是 tm 假的，得删掉。

```python
encrypted = open('encrypted', 'r').read().strip()
data = [int(encrypted[i:i+2], 16) for i in range(0, len(encrypted), 2)]
flag = 'TWCTF{***********************}'
for a in range(251):
    for b in range(251):
        ok = 1
        for i in range(len(flag)):
            if flag[i] != '*' and chr((data[i] * a + b) % 251) != flag[i]:
                ok = 0
        if ok:
            res = ''
            for i in range(len(flag)):
                if flag[i] == '*':
                    res += chr((data[i] * a + b) % 251)
                else:
                    res += flag[i]
            print(res)
```

## 答案

TWCTF{Faster_Than_Shinkansen!}
