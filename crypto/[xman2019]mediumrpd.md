## 描述

nc 47.97.215.88 20001

[Main.class.2185324843251fb0751087c080a47835](./assets/Main.class.2185324843251fb0751087c080a47835)

[level1.py.d69fbe0bb495b5a22966108e6274b097](./assets/level1.py.d69fbe0bb495b5a22966108e6274b097)

[Main.java.d6ff2cebe1571a806db6c945d85913e9](./assets/Main.java (1).d6ff2cebe1571a806db6c945d85913e9)

## 题解

和 [babyrpd]() 这道题差不多，只是换成了 java 的随机数。

先看一下 `Main.class`，再 wiki 了下 `.class` 的编码格式，用二进制编辑器看到 `major version` 为 52，也就是 Java SE 8，

查一下 Java 8 的 `Random` 源码 http://hg.openjdk.java.net/jdk8/jdk8/jdk/file/tip/src/share/classes/java/util/Random.java 可以发现

```java
public Random() {
    this(seedUniquifier() ^ System.nanoTime());
}
```

md，有点难度。。。再细看了一下，发现根本没办法卡好么。。

然后重新看回给的 `.py` 文件。。。原来，是给了前 2 个随机出来的数字，让你求第三个呀。。那难度就降低很多了，先看看其随机数生成算法：

```java
private final AtomicLong seed;
private static final long multiplier = 0x5DEECE66DL;
private static final long addend = 0xBL;
private static final long mask = (1L << 48) - 1;

protected int next(int bits) {
    long oldseed, nextseed;
    AtomicLong seed = this.seed;
    do {
        oldseed = seed.get();
        nextseed = (oldseed * multiplier + addend) & mask;
    } while (!seed.compareAndSet(oldseed, nextseed));
    return (int)(nextseed >>> (48 - bits));
}

public int nextInt() {
    return next(32);
}
```

可以发现，得到的随机数是随机种子 `seed` 向右移动了 16 个比特位。要是我们能破解出 seed，那么预测后面的数字就会相当的简单。由于 2^16 只有 6 万多，我们暴力枚举即可。

```java
import java.util.Scanner;
import java.util.concurrent.atomic.AtomicLong;

class Random {
    private final AtomicLong seed;
    private static final long multiplier = 0x5DEECE66DL;
    private static final long addend = 0xBL;
    private static final long mask = (1L << 48) - 1;

    Random(long seed) {
        this.seed = new AtomicLong(seed);
    }

    protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            nextseed = (oldseed * multiplier + addend) & mask;
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }

    public int nextInt() {
        return next(32);
    }
}

public class Main {
    public static int parse(long t) {
        if (t >= (1L << 31)) {
            t -= 0xffffffffL;
        }
        return (int)t;
    }
    public static void main(String []args) {
        Scanner scan = new Scanner(System.in);
        int a = parse(scan.nextLong());
        int b = parse(scan.nextLong());
        for (long i = 0; i < (1L << 16); ++i) {
            long seed = ((long)a << 16) | i;
            Random r = new Random(seed);
            int t = r.nextInt();
            if (b == t) {
                System.out.printf("seed = %d\n", seed);
                long c = (long)r.nextInt();
                if (c < 0) {
                    c += 0xffffffffL;
                }
                System.out.printf("next = %d\n", c);
            }
        }
    }
}
```

## 答案

flag{random_get_rrrr10mjs0f}
