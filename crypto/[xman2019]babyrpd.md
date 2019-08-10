## 描述

nc 47.97.215.88 20000

```python
class Unbuffered(object):
   def __init__(self, stream):
       self.stream = stream
   def write(self, data):
       self.stream.write(data)
       self.stream.flush()
   def __getattr__(self, attr):
       return getattr(self.stream, attr)
import sys
sys.stdout = Unbuffered(sys.stdout)
import signal
signal.alarm(600)

import random
import time
flag=open("/root/level0/flag","r").read()

random.seed(int(time.time()))
def check():
    recv=int(raw_input())
    if recv==random.randint(0,2**64):
        print flag
        return True
    else:
        print "atum tql"
        return False

while 1:
    if check():
        break
```

## 题解

一看就知道要本地也跑一个版本，时间稍微提前几秒就好。模拟一下当前时间的最近几秒即可

```python
from pwn import *
import random
def g(seed, tries):
    random.seed(seed)
    a = 0
    for i in range(tries):
        a = random.randint(0, 2**64)
    return a

con = remote('47.97.215.88', 20000)
seed = int(time.time()) + 5
for i in range(1, 10000):
    print seed, i
    con.send(str(g(seed, i)) + '\n')
    l = con.recvline().strip()
    print l
    if l != 'atum tql':
        break
    seed -= 1
```

## 答案

flag{good_good_study_dayday_up}
