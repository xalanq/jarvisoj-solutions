## 描述

已知椭圆曲线加密Ep(a,b)参数为

p = 15424654874903

a = 16546484

b = 4548674875

G(6478678675,5636379357093)

私钥为

k = 546768

求公钥K(x,y)

提示：K=kG

提交格式XUSTCTF{x+y}(注意，大括号里面是x和y加起来求和，不是用加号连接)

注：题目来源XUSTCTF2016

## 题解

网上找了几篇文章学习了下，但是讲得最清楚的其实是这一篇

https://www.jianshu.com/p/e41bc1eb1d81

简要总结一下

y² = (x³ + ax + b) % p

定义取反操作 -A(x, y) = A(x, -y)

定义相加 R(Xr, Yr) = P(Xp, Yp) + Q(Xq, Yq) 操作为

- 保证 P ≠ -Q
- 若 P = Q，λ = (Yq - Yp) / (Xq - Xp)
- 若 P ≠ Q，λ = (3Xp² + a) / (2Yp)
- Xr = (λ² - Xp - Xq)
- Yr = (λ(Xp - Xr) - Yp)

生成质数 p、随机起点 G(x, y) 和随机整数私钥 k，然后将上述计算全部放到模 p 意义下的整数域上进行。

- 计算公钥 K = kG，发送 {G, K}
- 公钥加密：选择随机数 r，将消息 M 生成密文 C = {rG, M + rK}，发送 C
- 私钥解密：M + rK - k(rG) = M + r(kG) - k(rG) = M

注意：

- 上面的大小写 k 和 K
- 零点不是 (0, 0)

然后就可以做了，为了加深印象，我自己手写了一份代码（求逆可以直接用快速幂是因为 p 是质数，由欧拉定理（费马小定理）得知）

```python
P = 15424654874903
A = 16546484
B = 4548674875
G = (6478678675, 5636379357093)
k = 546768

def fpow(a, b, c):
    r = 1
    while b:
        if b & 1:
            r = r * a % c
        a = a * a % c
        b = b >> 1
    return r

def inv(a, p):
    return fpow(a, p - 2, p)

class Point:
    def __init__(self, x, y):
        self.x = (x % P + P) % P
        self.y = (y % P + P) % P
    
    def __add__(self, other):
        assert(not(self.x == other.x and self.y == (P - other.y) % P))
        if self.x != other.x:
            lbd = (other.y - self.y + P) * inv(other.x - self.x + P, P) % P
        else:
            lbd = (3 * self.x * self.x + A) * inv(2 * self.y, P) % P
        x = (lbd * lbd - self.x - other.x + P + P) % P
        y = (lbd * (self.x - x + P) - self.y + P) % P
        return Point(x, y)
    
    def __neg__(self):
        return Point(self.x, (P - self.y) % P)
    
    def __mul__(self, num):
        assert(isinstance(num, int))
        r = None
        a = self
        while num:
            if num & 1:
                if r == None:
                    r = a
                else:
                    r = r + a
            a = a + a
            num = num >> 1
        return r
    
    def __str__(self):
        return "({}, {})".format(self.x, self.y)

K = Point(G[0], G[1]) * k
print(K)
print(K.x + K.y)
```

## 答案

XUSTCTF{19477226185390}
