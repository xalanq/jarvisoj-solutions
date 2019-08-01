## 描述

nit yqmg mqrqn bxw mtjtm nq rqni fiklvbxu mqrqnl xwg dvmnzxu lqjnyxmt xatwnl, rzn nit uxnntm xmt zlzxuuk mtjtmmtg nq xl rqnl. nitmt vl wq bqwltwlzl qw yivbi exbivwtl pzxuvjk xl mqrqnl rzn nitmt vl atwtmxu xamttetwn xeqwa tsftmnl, xwg nit fzruvb, nixn mqrqnl ntwg nq gq lqet qm xuu qj nit jquuqyvwa: xbbtfn tutbnmqwvb fmqamxeevwa, fmqbtll gxnx qm fiklvbxu ftmbtfnvqwl tutbnmqwvbxuuk, qftmxnt xznqwqeqzluk nq lqet gtamtt, eqdt xmqzwg, qftmxnt fiklvbxu fxmnl qj vnltuj qm fiklvbxu fmqbtlltl, ltwlt xwg exwvfzuxnt nitvm twdvmqwetwn, xwg tsivrvn vwntuuvatwn rtixdvqm - tlftbvxuuk rtixdvqm yivbi evevbl izexwl qm qnitm xwvexul. juxa vl lzrlnvnzntfxllvldtmktxlkkqzaqnvn. buqltuk mtuxntg nq nit bqwbtfn qj x mqrqn vl nit jvtug qj lkwnitnvb rvquqak, yivbi lnzgvtl twnvnvtl yiqlt wxnzmt vl eqmt bqefxmxrut nq rtvwal nixw nq exbivwtl.

提交格式：PCTF{flag}

## 题解

```python
a = "nit yqmg mqrqn"
for i in range(26):
    b = ""
    for x in a:
        if ord('a') <= ord(x) and ord(x) <= ord('z'):
            b += chr((ord(x) - ord('a') + i) % 26 + ord('a'))
        else:
            b += x
    print(b)
```

发现不行，感觉不是简单的映射，可能要统计词频

```python
a = '''nit yqmg mqrqn bxw mtjtm nq rqni fiklvbxu mqrqnl xwg dvmnzxu lqjnyxmt xatwnl, rzn nit uxnntm xmt zlzxuuk mtjtmmtg nq xl rqnl. nitmt vl wq bqwltwlzl qw yivbi exbivwtl pzxuvjk xl mqrqnl rzn nitmt vl atwtmxu xamttetwn xeqwa tsftmnl, xwg nit fzruvb, nixn mqrqnl ntwg nq gq lqet qm xuu qj nit jquuqyvwa: xbbtfn tutbnmqwvb fmqamxeevwa, fmqbtll gxnx qm fiklvbxu ftmbtfnvqwl tutbnmqwvbxuuk, qftmxnt xznqwqeqzluk nq lqet gtamtt, eqdt xmqzwg, qftmxnt fiklvbxu fxmnl qj vnltuj qm fiklvbxu fmqbtlltl, ltwlt xwg exwvfzuxnt nitvm twdvmqwetwn, xwg tsivrvn vwntuuvatwn rtixdvqm - tlftbvxuuk rtixdvqm yivbi evevbl izexwl qm qnitm xwvexul. juxa vl lzrlnvnzntfxllvldtmktxlkkqzaqnvn. buqltuk mtuxntg nq nit bqwbtfn qj x mqrqn vl nit jvtug qj lkwnitnvb rvquqak, yivbi lnzgvtl twnvnvtl yiqlt wxnzmt vl eqmt bqefxmxrut nq rtvwal nixw nq exbivwtl.'''
cnt = [0] * 26
for x in a:
    if ord('a') <= ord(x) and ord(x) <= ord('z'):
        cnt[ord(x) - ord('a')] += 1
for i in range(26):
    t = -1
    for j in range(26):
        if cnt[j] >= 0 and (t < 0 or cnt[j] > cnt[t]):
            t = j
    print(chr(ord('a') + t) + ' ' + str(cnt[t]))
    cnt[t] = -1
```

根据词频和 https://www.zhihu.com/question/19805834 手动构造 map，然后找一些常用的词 the，there，is 啥的试试，将表修改一下，经过十几分钟的尝试，得到答案

```python
def f():
    b = ""
    for x in a:
        if x in m:
            b += m[x]
        else:
            b += x
    print(b)

m = {
    't': 'e', #
    'n': 't', #
    'q': 'o', #
    'l': 's', #
    'x': 'a', #
    'm': 'r', #
    'v': 'i', #
    'w': 'n', #
    'u': 'l', #
    'i': 'h', #
    'b': 'c', #
    'f': 'p', #
    'e': 'm', #
    'z': 'u', #
    'r': 'b', #
    'k': 'y', #
    'g': 'd', #
    'a': 'g', #
    'j': 'f', #
    'y': 'w', #
    'd': 'v', #
    's': 'x', #
    'p': 'q', #
    'c': 'k',
    'h': 'j',
    'o': 'z',
}
f()
```

解密文本

the word robot can refer to both physical robots and virtual software agents, but the latter are usually referred to as bots. there is no consensus on which machines qualify as robots but there is general agreement among experts, and the public, that robots tend to do some or all of the following: accept electronic programming, process data or physical perceptions electronically, operate autonomously to some degree, move around, operate physical parts of itself or physical processes, sense and manipulate their environment, and exhibit intelligent behavior - especially behavior which mimics humans or other animals. flag is substitutepassisveryeasyyougotit. closely related to the concept of a robot is the field of synthetic biology, which studies entities whose nature is more comparable to beings than to machines.

网上搜题解发现可以用这个网站 https://quipqiup.com/ ，我好 naive 啊

## 答案

PCTF{substitutepassisveryeasyyougotit}
