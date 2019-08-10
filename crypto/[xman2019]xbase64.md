## 描述

[encode.py.93ddf015f10b30bcd5e9d29fc6381aef](./assets/encode.py.93ddf015f10b30bcd5e9d29fc6381aef)

## 题解

只是把标准的 Base64 的符号对应表翻转了一下，简单翻转回来即可。

```python
base64_table = ['=','A', 'B', 'C', 'D','E','F','G','H','I','J','K','L','M','N','O','P','Q','R','S','T','U','V','W','X','Y','Z',
                'a', 'b', 'c', 'd','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z',
                '0', '1', '2', '3','4','5','6','7','8','9',
                '+', '/'][::-1]
a = 'mZOemISXmpOTkKCHkp6Rgv=='
b = ''
for c in a:
    t = 63 - base64_table.index(c)
    if t < 0:
        t = 64
    b += base64_table[t]

print(b)
```

## 答案

flag{hello_xman}
