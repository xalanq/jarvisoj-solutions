## 描述

nc 47.97.215.88 20002

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
import os
os.chdir("/root/level2")

from random import *

while 1:
    a=raw_input("#")
    target=getrandbits(32)
    if a!=str(target):
        print target
    else:
        print open("flag","rb").read()
```

## 题解

来看看 Py 2 的 `getrandbits` 源代码：https://github.com/python/cpython/blob/2.7/Modules/_randommodule.c#L466

算法是 MT19937

```c
/* Period parameters -- These are all magic.  Don't change. */
#define N 624
#define M 397
#define MATRIX_A 0x9908b0dfUL   /* constant vector a */
#define UPPER_MASK 0x80000000UL /* most significant w-r bits */
#define LOWER_MASK 0x7fffffffUL /* least significant r bits */

/* initializes mt[N] with a seed */
static void
init_genrand(RandomObject *self, unsigned long s)
{
    int mti;
    unsigned long *mt;

    mt = self->state;
    mt[0]= s & 0xffffffffUL;
    for (mti=1; mti<N; mti++) {
        mt[mti] =
        (1812433253UL * (mt[mti-1] ^ (mt[mti-1] >> 30)) + mti);
        /* See Knuth TAOCP Vol2. 3rd Ed. P.106 for multiplier. */
        /* In the previous versions, MSBs of the seed affect   */
        /* only MSBs of the array mt[].                                */
        /* 2002/01/09 modified by Makoto Matsumoto                     */
        mt[mti] &= 0xffffffffUL;
        /* for >32 bit machines */
    }
    self->index = mti;
    return;
}

static unsigned long
genrand_int32(RandomObject *self)
{
    unsigned long y;
    static unsigned long mag01[2]={0x0UL, MATRIX_A};
    /* mag01[x] = x * MATRIX_A  for x=0,1 */
    unsigned long *mt;

    mt = self->state;
    if (self->index >= N) { /* generate N words at one time */
        int kk;

        for (kk=0;kk<N-M;kk++) {
            y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK);
            mt[kk] = mt[kk+M] ^ (y >> 1) ^ mag01[y & 0x1UL];
        }
        for (;kk<N-1;kk++) {
            y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK);
            mt[kk] = mt[kk+(M-N)] ^ (y >> 1) ^ mag01[y & 0x1UL];
        }
        y = (mt[N-1]&UPPER_MASK)|(mt[0]&LOWER_MASK);
        mt[N-1] = mt[M-1] ^ (y >> 1) ^ mag01[y & 0x1UL];

        self->index = 0;
    }

    y = mt[self->index++];
    y ^= (y >> 11);
    y ^= (y << 7) & 0x9d2c5680UL;
    y ^= (y << 15) & 0xefc60000UL;
    y ^= (y >> 18);
    return y;
}

static PyObject *
random_getrandbits(RandomObject *self, PyObject *args)
{
    int k, i, bytes;
    unsigned long r;
    unsigned char *bytearray;
    PyObject *result;

    if (!PyArg_ParseTuple(args, "i:getrandbits", &k))
        return NULL;

    if (k <= 0) {
        PyErr_SetString(PyExc_ValueError,
                        "number of bits must be greater than zero");
        return NULL;
    }

    bytes = ((k - 1) / 32 + 1) * 4;
    bytearray = (unsigned char *)PyMem_Malloc(bytes);
    if (bytearray == NULL) {
        PyErr_NoMemory();
        return NULL;
    }

    /* Fill-out whole words, byte-by-byte to avoid endianness issues */
    for (i=0 ; i<bytes ; i+=4, k-=32) {
        r = genrand_int32(self);
        if (k < 32)
            r >>= (32 - k);
        bytearray[i+0] = (unsigned char)r;
        bytearray[i+1] = (unsigned char)(r >> 8);
        bytearray[i+2] = (unsigned char)(r >> 16);
        bytearray[i+3] = (unsigned char)(r >> 24);
    }

    /* little endian order to match bytearray assignment order */
    result = _PyLong_FromByteArray(bytearray, bytes, 1, 0);
    PyMem_Free(bytearray);
    return result;
}
```

我们先来看

```c
y ^= (y >> 11);
y ^= (y << 7) & 0x9d2c5680UL;
y ^= (y << 15) & 0xefc60000UL;
y ^= (y >> 18);
```

向右移动和向左移动再亦或，是能逆回来的。

比如向右移动一位再亦或，那么我们可以得知最高位是没变的，然后次高位是和原先的次高位和最高位做了一次亦或得到的，根据亦或的可逆性，再亦或一次最高位就能得到原先的次高位。所以能简单的写出这个代码：

```python
def reverse_right_shift(y, shift):
    i = 31
    r = 0
    while i >= 0:
        x = r >> shift
        bit = ((x >> i) ^ (y >> i)) & 1
        r |= bit << i
        i -= 1
    return r
```

向左移动也是同理

```python
def ls(x, s):
    return (x << s) & 0xffffffff

def reverse_left_shift(y, shift, mask):
    i = 0
    r = 0
    while i <= 31:
        x = ls(r, shift) & mask
        bit = ((x >> i) ^ (y >> i)) & 1
        r |= bit << i
        i += 1
    return r
```

我们再看

```c
static unsigned long mag01[2]={0x0UL, MATRIX_A};

if (self->index >= N) { /* generate N words at one time */
    int kk;

    for (kk=0;kk<N-M;kk++) {
        y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK);
        mt[kk] = mt[kk+M] ^ (y >> 1) ^ mag01[y & 0x1UL];
    }
    for (;kk<N-1;kk++) {
        y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK);
        mt[kk] = mt[kk+(M-N)] ^ (y >> 1) ^ mag01[y & 0x1UL];
    }
    y = (mt[N-1]&UPPER_MASK)|(mt[0]&LOWER_MASK);
    mt[N-1] = mt[M-1] ^ (y >> 1) ^ mag01[y & 0x1UL];

    self->index = 0;
}
```

这是用之前的 N 个数来生成下一批 N 个数，因此我们逆出 N 个数后，推导出下一批 N 个数，就能预测下一个随机数是啥了。这段代码翻译成 python 就是这样

```python
N = 624
M = 397
MATRIX_A = 0x9908b0df
UPPER_MASK = 0x80000000
LOWER_MASK = 0x7fffffff
mag01[2] = [0x0, MATRIX_A]

# 初始化 mt 为 N 个初始值

def next_state():
    kk = 0
    while kk < N - M:
        y = (mt[kk] & UPPER_MASK)|(mt[kk+1] & LOWER_MASK)
        mt[kk] = mt[kk+M] ^ (y >> 1) ^ mag01[y & 0x1]
        kk += 1
    while kk < N - 1:
        y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK)
        mt[kk] = mt[kk+(M-N)] ^ (y >> 1) ^ mag01[y & 0x1]
        kk += 1
    y = (mt[N-1]&UPPER_MASK)|(mt[0]&LOWER_MASK)
    mt[N-1] = mt[M-1] ^ (y >> 1) ^ mag01[y & 0x1]
```

总体代码就是这样

```python
from pwn import *

def reverse_right_shift(y, shift):
    i = 31
    r = 0
    while i >= 0:
        x = r >> shift
        bit = ((x >> i) ^ (y >> i)) & 1
        r |= bit << i
        i -= 1
    return r

def ls(x, s):
    return (x << s) & 0xffffffff

def reverse_left_shift(y, shift, mask):
    i = 0
    r = 0
    while i <= 31:
        x = ls(r, shift) & mask
        bit = ((x >> i) ^ (y >> i)) & 1
        r |= bit << i
        i += 1
    return r

N = 624
M = 397
MATRIX_A = 0x9908b0df
UPPER_MASK = 0x80000000
LOWER_MASK = 0x7fffffff
mag01 = [0x0, MATRIX_A]

mt = []

def next_state():
    kk = 0
    while kk < N - M:
        y = (mt[kk] & UPPER_MASK)|(mt[kk+1] & LOWER_MASK)
        mt[kk] = mt[kk+M] ^ (y >> 1) ^ mag01[y & 0x1]
        kk += 1
    while kk < N - 1:
        y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK)
        mt[kk] = mt[kk+(M-N)] ^ (y >> 1) ^ mag01[y & 0x1]
        kk += 1
    y = (mt[N-1]&UPPER_MASK)|(mt[0]&LOWER_MASK)
    mt[N-1] = mt[M-1] ^ (y >> 1) ^ mag01[y & 0x1]

con = remote('47.97.215.88', 20002)

for i in range(N):
    print(i)
    con.send('0\n')
    y = int(con.recvline().strip()[1:])
    y = reverse_right_shift(y, 18)
    y = reverse_left_shift(y, 15, 0xefc60000)
    y = reverse_left_shift(y, 7, 0x9d2c5680)
    y = reverse_right_shift(y, 11)
    mt.append(y)

next_state()

y = mt[0]
y ^= (y >> 11)
y ^= ls(y, 7) & 0x9d2c5680
y ^= ls(y, 15) & 0xefc60000
y ^= (y >> 18)

print(y)
con.send(str(y) + '\n')
print(con.recvline())
```

## 答案

flag{this_soth_f389fh91hff}
