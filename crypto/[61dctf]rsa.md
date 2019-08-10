## 描述

[rsa.zip.66080e40bb4a92fd4358d7236cda7c91](./assets/rsa.zip.66080e40bb4a92fd4358d7236cda7c91)

## 题解

看了下代码，就是两个人用 RSA 进行通讯，一堆混淆视听的代码。总的来说 `send.bak` 文件是用 Bob 的公钥 (e, N1) 加密得到的密文，直接破解这个就好。

```
e = 57019382772166837488750427040015464490863175936396575556290232679668259683514801918298546850349718538222635977304361506235546587039055174992606638351023434322917005735910505716857963030159773221874370814472973319422853023944637626372539074831161749650232791256091117301145054914651557523038888181457290682237

N1 = 68253958478963934124300215757460723273691925280596309221863375905836387105252457670797482776100691909463871547306272253629697109054703333176926409544379734932863965587299902358818026560280727724874176607409038276642286734734838152097800253007709025880304435661500771500046972983240470823245360867996258239121
```

好吧，wiener 算法破解不出来。。

仔细阅读 `keygen.sage` 发现 n1 和 n2 是有某种关系的，感觉可能是要分析一下然后把两个质因子给捣鼓出来。

好吧，不会做。。。找题解。。。找了超久发现这是 0ctf-2017-final 的题，题解在这 https://elliptic-shiho.github.io/ctf-writeups/#!ctf/2017/0CTF%20Finals/cr1000-AuthenticationSecrecy/README.md

这道题特别的难，这个问题叫 Dual RSA，参数 e, d, N1, N2 满足 ed = 1 (mod φ(N1)), ed = 1 (mod φ(N2))，然后只给出 e, N1, N2。

解法为论文 Liqiang Peng, Lei Hu, Yao Lu, Jun Xu and Zhangjie Huang. 2016. "Cryptanalysis of Dual RSA"

链接给的代码是 sage 的，必须要装 sagemath 才能运行，具体安装教程：

- 去官网下载二进制版本
- `sage -i giacpy_sage`
- `sudo apt install libssl1.0.0`

我尝试翻译成原生 Py3 的。。然后到一半发现太难了。。。

```python
from sage.all import *
import math
import itertools
import alice_public_key
import bob_public_key

# display matrix picture with 0 and X
# references: https://github.com/mimoo/RSA-and-LLL-attacks/blob/master/boneh_durfee.sage
def matrix_overview(BB):
    for ii in range(BB.dimensions()[0]):
        a = ('%02d ' % ii)
        for jj in range(BB.dimensions()[1]):
            a += ' ' if BB[ii,jj] == 0 else 'X'
            if BB.dimensions()[0] < 60:
                a += ' '
        print a

def dual_rsa_liqiang_et_al(e, n1, n2, delta, mm, tt):
    '''
    Attack to Dual RSA: Liqiang et al.'s attack implementation

    References:
        [1] Liqiang Peng, Lei Hu, Yao Lu, Jun Xu and Zhangjie Huang. 2016. "Cryptanalysis of Dual RSA"
    '''
    N = (n1+n2)/2
    A = ZZ(floor(N^0.5))

    _XX = ZZ(floor(N^delta))
    _YY = ZZ(floor(N^0.5))
    _ZZ = ZZ(floor(N^(delta - 1./4)))
    _UU = _XX * _YY + 1

    # Find a "good" basis satisfying d = a1 * l'11 + a2 * l'21
    M = Matrix(ZZ, [[A, e], [0, n1]])
    B = M.LLL()
    l11, l12 = B[0]
    l21, l22 = B[1]
    l_11 = ZZ(l11 / A)
    l_21 = ZZ(l21 / A)

    modulo = e * l_21
    F = Zmod(modulo)

    PR = PolynomialRing(F, 'u, x, y, z')
    u, x, y, z = PR.gens()

    PK = PolynomialRing(ZZ, 'uk, xk, yk, zk')
    uk, xk, yk, zk = PK.gens()

    # For transform xy to u-1 (unravelled linearlization)
    PQ = PK.quo(xk*yk+1-uk)

    f = PK(x*(n2 + y) - e*l_11*z + 1)

    fbar = PQ(f).lift()

    # Polynomial construction
    gijk = {}
    for k in xrange(0, mm + 1):
        for i in xrange(0, mm-k + 1):
            for j in xrange(0, mm-k-i + 1):
                gijk[i, j, k] = PQ(xk^i * zk^j * PK(fbar) ^ k * modulo^(mm-k)).lift()

    hjkl = {}
    for j in xrange(1, tt + 1):
        for k in xrange(floor(mm / tt) * j, mm + 1):
            for l in xrange(0, k + 1):
                hjkl[j, k, l] = PQ(yk^j * zk^(k-l) * PK(fbar) ^ l * modulo^(mm-l)).lift()

    monomials = []
    for k in gijk.keys():
        monomials += gijk[k].monomials()
    for k in hjkl.keys():
        monomials += hjkl[k].monomials()

    monomials = sorted(set(monomials))[::-1]
    assert len(monomials) == len(gijk) + len(hjkl) # square matrix?
    dim = len(monomials)

    # Create lattice from polynmial g_{ijk} and h_{jkl}
    M = Matrix(ZZ, dim)
    row = 0
    for k in gijk.keys():
        for i, monomial in enumerate(monomials):
            M[row, i] = gijk[k].monomial_coefficient(monomial) * monomial.subs(uk=_UU, xk=_XX, yk=_YY, zk=_ZZ)
        row += 1
    for k in hjkl.keys():
        for i, monomial in enumerate(monomials):
            M[row, i] = hjkl[k].monomial_coefficient(monomial) * monomial.subs(uk=_UU, xk=_XX, yk=_YY, zk=_ZZ)
        row += 1

    matrix_overview(M)
    print '=' * 128

    # LLL
    B = M.LLL()

    matrix_overview(B)

    # Construct polynomials from reduced lattices
    H = [(i, 0) for i in xrange(dim)]
    H = dict(H)
    for j in xrange(dim):
        for i in xrange(dim):
            H[i] += PK((monomials[j] * B[i, j]) / monomials[j].subs(uk=_UU, xk=_XX, yk=_YY, zk=_ZZ))
    H = H.values()

    PQ = PolynomialRing(QQ, 'uq, xq, yq, zq')
    uq, xq, yq, zq = PQ.gens()

    # Inversion of unravelled linearlization
    for i in xrange(dim):
        H[i] = PQ(H[i].subs(uk=xk*yk+1))

    # Calculate Groebner basis for solve system of equations
    '''
    Actually, These polynomials selection (H[1:20]) is heuristic selection.
    Because they are "short" vectors. We need a short vector less than
    Howgrave-Graham bound. So we trying test parameter(at [1]) and decided it.
    '''
    I = Ideal(*H[1:20])
    g = I.groebner_basis('giac')[::-1]
    mon = map(lambda t: t.monomials(), g)

    PX = PolynomialRing(ZZ, 'xs')
    xs = PX.gen()

    x_pol = y_pol = z_pol = None

    for i in xrange(len(g)):
        if mon[i] == [xq, 1]:
            print g[i] / g[i].lc()
            x_pol = g[i] / g[i].lc()
        elif mon[i] == [yq, 1]:
            print g[i] / g[i].lc()
            y_pol = g[i] / g[i].lc()
        elif mon[i] == [zq, 1]:
            print g[i] / g[i].lc()
            z_pol = g[i] / g[i].lc()

    if x_pol is None or y_pol is None or z_pol is None:
        print '[-] Failed: we cannot get a solution...'
        return

    x0 = x_pol.subs(xq=xs).roots()[0][0]
    y0 = y_pol.subs(yq=xs).roots()[0][0]
    z0 = z_pol.subs(zq=xs).roots()[0][0]

    # solution check
    assert f(x0*y0+1, x0, y0, z0) % modulo == 0

    a0 = z0
    a1 = (x0 * (n2 + y0) + 1 - e*l_11*z0) / (e*l_21)

    d = a0 * l_11 + a1 * l_21
    return d

if __name__ == '__main__':
    delta = 0.334
    mm = 4
    tt = 2

    n1 = alice_public_key.N1
    n2 = alice_public_key.N2
    e = alice_public_key.e

    d1 = dual_rsa_liqiang_et_al(e, n1, n2, delta, mm, tt)
    print '[+] d for alice = %d' % d1

    n1 = bob_public_key.N1
    n2 = bob_public_key.N2
    e = bob_public_key.e

    d2 = dual_rsa_liqiang_et_al(e, n1, n2, delta, mm, tt)
    print '[+] d for bob = %d' % d2

    d = ZZ(open('send.bak', 'rb').read().encode('hex'), 16)
    m = ZZ(Mod(ZZ(Mod(d, bob_public_key.N1)^d2), alice_public_key.N2)^alice_public_key.e)
    m_ = hex(m)
    if len(m_) % 2 == 1:
        m_ = '0' + m_
    print repr(m_.decode('hex'))
```

## 答案

flag{eNj0y_th3_CrYpt4na1ysi5_0F_Dual_RSA~~}
