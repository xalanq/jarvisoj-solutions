## 描述

都说逆向挺难的，但是这题挺容易的，反正我不会，大家来挑战一下吧~~:)

[easycrackme.6dbc7c78c9bb25f724cd55c0e1412617](./assets/easycrackme.6dbc7c78c9bb25f724cd55c0e1412617)

## 题解

 strings 了一下，发现应该是可执行文件。用 IDA Pro 打开，反编译就得到了以下代码：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  __int64 v3; // rdi
  char v5; // [rsp+0h] [rbp-38h]
  char v6; // [rsp+1h] [rbp-37h]
  char v7; // [rsp+2h] [rbp-36h]
  char v8; // [rsp+3h] [rbp-35h]
  char v9; // [rsp+4h] [rbp-34h]
  char v10; // [rsp+5h] [rbp-33h]
  unsigned __int8 v11; // [rsp+10h] [rbp-28h]
  _BYTE v12[7]; // [rsp+11h] [rbp-27h]

  v5 = -85;
  v6 = -35;
  v7 = 51;
  v8 = 84;
  v9 = 53;
  v10 = -17;
  printf((unsigned __int64)"Input your password:");
  _isoc99_scanf((unsigned __int64)"%s");
  if ( strlen((const char *)&v11) == 26 )
  {
    v3 = 0LL;
    if ( (v11 ^ 0xAB) == list1 )
    {
      while ( (v12[v3] ^ (unsigned __int8)*(&v5 + ((signed int)v3 + 1) % 6)) == byte_6B41D1[v3] )
      {
        if ( ++v3 == 25 )
        {
          printf((unsigned __int64)"Congratulations!");
          return 0;
        }
      }
    }
  }
  printf((unsigned __int64)"Password Wrong!! Please try again.");
  return 0;
}
```

```
.data:00000000006B41D0 list1           db 0FBh                 ; DATA XREF: main+94↑r
.data:00000000006B41D1 ; char byte_6B41D1[31]
.data:00000000006B41D1 byte_6B41D1     db 9Eh                  ; DATA XREF: main+C0↑r
.data:00000000006B41D2                 db  67h ; g
.data:00000000006B41D3                 db  12h
.data:00000000006B41D4                 db  4Eh ; N
.data:00000000006B41D5                 db  9Dh
.data:00000000006B41D6                 db  98h
.data:00000000006B41D7                 db 0ABh
.data:00000000006B41D8                 db    0
.data:00000000006B41D9                 db    6
.data:00000000006B41DA                 db  46h ; F
.data:00000000006B41DB                 db  8Ah
.data:00000000006B41DC                 db 0F4h
.data:00000000006B41DD                 db 0B4h
.data:00000000006B41DE                 db    6
.data:00000000006B41DF                 db  0Bh
.data:00000000006B41E0                 db  43h ; C
.data:00000000006B41E1                 db 0DCh
.data:00000000006B41E2                 db 0D9h
.data:00000000006B41E3                 db 0A4h
.data:00000000006B41E4                 db  6Ch ; l
.data:00000000006B41E5                 db  31h ; 1
.data:00000000006B41E6                 db  74h ; t
.data:00000000006B41E7                 db  9Ch
.data:00000000006B41E8                 db 0D2h
.data:00000000006B41E9                 db 0A0h
.data:00000000006B41EA                 db    0
.data:00000000006B41EB                 db    0
.data:00000000006B41EC                 db    0
.data:00000000006B41ED                 db    0
.data:00000000006B41EE                 db    0
.data:00000000006B41EF                 db    0
.data:00000000006B41F0                 public _dl_tls_static_size
```

根据亦或的交换性 a ^ b = c 即 a = b ^ c，可以写出代码

```cpp
#include <bits/stdc++.h>
using namespace std;
int main() {
    int n;
    char list = 0x0FB;
    char a[6] = {-85, -35, 51, 84, 53, -17};
    unsigned char bytes[31] = {0x9E, 0x67, 0x12, 0x4E, 0x9D, 0x98, 0x0AB, 0, 6, 0x46, 0x8A, 0x0F4, 0x0B4, 6, 0x0B, 0x43, 0x0DC, 0x0D9, 0x0A4, 0x6C, 0x31, 0x74, 0x9C, 0x0D2, 0x0A0, 0, 0, 0, 0, 0};
    printf("%c", list ^ 0xAB);
    for (n = 0; n < 25; ++n) {
        printf("%c", a[(n + 1) % 6] ^ bytes[n]);
    }
}
```

运行便得到答案

## 答案

PCTF{r3v3Rse_i5_v3ry_eAsy}
