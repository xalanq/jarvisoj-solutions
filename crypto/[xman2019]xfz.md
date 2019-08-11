## 描述

[xfz.7z.704e85021952fe7593f270ec411c3b0e](./assets/xfz.7z.704e85021952fe7593f270ec411c3b0e)

## 题解

```python
def round(M,K):
    L=M[0:27]
    R=M[27:54]
    new_l=R
    new_r=xor(xor(R,L),K)
    return new_l+new_r
```

的逆操作为

```python
def r_round(M,K):
    L=M[0:27]
    R=M[27:54]
    new_l=xor(xor(R,L),K)
    new_r=L
    return new_l+new_r
```

而

```python
def fez(m,K):
    for i in K:
        m=round(m,i)
    return m
```

的逆操作为

```python
def r_fez(m,K):
    for i in K[::-1]:
        m=r_round(m,i)
    return m
```

题目给了我们 `test`、`fez(test, K)` 和 `fez(flag, K)`，我们要想办法将 `K` 给求出来。

我感觉，`fez` 里的 K 可以分奇偶来做，因为 m 交换两次顺序又回来了，所以会不会奇数、偶数位置的 K 全部亦或在一起变成两个 K 效果是一样的？

尝试一下

```python
def ffez(m,K):
    k1 = 0
    k2 = 0
    for i in range(len(K)):
        k1 ^= ord(k)
    for k in K[1::2]:
        k2 ^= ord(k)
    m = round(m,k1)
    m = round(m,k2)
    if len(K) & 1:
        m = round(m, 0)
    return m
```

完全不对。

冷静一下，仔细想想。

暴力分解一下加密流程

|  L  |  R  |
|:---:|:---:|
|  x  |  y  |
|  y  |  x^y^k<sub>0</sub>  |
| x^y^k<sub>0</sub> | x^k<sub>0</sub>^k<sub>1</sub>|
| x^k<sub>0</sub>^k<sub>1</sub> | y^k<sub>1</sub>^k<sub>2</sub>|
| y^k<sub>1</sub>^k<sub>2</sub> | x^y^k<sub>0</sub>^k<sub>2</sub>^k<sub>3</sub> |
| x^y^k<sub>0</sub>^k<sub>2</sub>^k<sub>3</sub> | x^k<sub>0</sub>^k<sub>1</sub>^k<sub>3</sub>^k<sub>4</sub> |
| x^k<sub>0</sub>^k<sub>1</sub>^k<sub>3</sub>^k<sub>4</sub> | y^k<sub>1</sub>^k<sub>2</sub>^k<sub>4</sub>^k<sub>5</sub> |
| y^k<sub>1</sub>^k<sub>2</sub>^k<sub>4</sub>^k<sub>5</sub> | x^y^k<sub>0</sub>^k<sub>2</sub>^k<sub>3</sub>^k<sub>5</sub>^k<sub>6</sub> |

所以既然我们知道了 `test` 和 `fez(test, K)`（也就是第一行和最后一行），那么很简单了啊！！！马上能得出 k<sub>1</sub>^k<sub>2</sub>^k<sub>4</sub>^k<sub>5</sub> 和 k<sub>0</sub>^k<sub>2</sub>^k<sub>3</sub>^k<sub>5</sub>^k<sub>6</sub>，再套到 flag 里就好！

```python
def xor(a,b):
    assert len(a) == len(b)
    c = bytearray()
    for i in range(len(a)):
        c.append(a[i] ^ b[i])
    return bytes(c)

f = open('fez.log', 'r')
test = bytes.fromhex(f.readline().strip())
fez_test = bytes.fromhex(f.readline().strip())
fez_flag = bytes.fromhex(f.readline().strip())
x = test[0:27]
y = test[27:54]
ka = xor(fez_test[0:27], y)
kb = xor(fez_test[27:54], xor(x, y))
fy = xor(fez_flag[0:27], ka)
fx = xor(fy, xor(fez_flag[27:54], kb))
print(fx + fy)
```


## 答案

