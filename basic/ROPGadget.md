## 描述

都说学好汇编是学习PWN的基础，以下有一段ROPGadget的汇编指令序列，请提交其十六进制机器码(大写，不要有空格)

```
XCHG EAX,ESP
RET
MOV ECX,[EAX]
MOV [EDX],ECX
POP EBX
RET
```

提交格式：PCTF{你的答案}

## 题解

真不会。。看题解说可以用 pwntools，但是这玩意不支持 python 3 有点蛋疼。。用 py 2 运行得到

```
from pwn import *
asm('XCHG EAX,ESP; RET; MOV ECX,[EAX]; MOV [EDX],ECX; POP EBX; RET'.lower()).encode('hex').upper()
```

## 答案

PCTF{94C38B08890A5BC3}
