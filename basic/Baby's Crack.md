## 描述

既然是逆向题，我废话就不多说了，自己看着办吧。

[babyscrack.rar.831d813059fb7a7eb9cd0e9904726977](./assets/babyscrack.rar.831d813059fb7a7eb9cd0e9904726977)

## 题解

解压得到两个文件，一个纯文本，一个 exe，这个 exe 可能可以读入这个文本得到答案。

strings 了一下发现了很多 C 的代码，因此尝试 gdb，然后发现应该是没开 -g 编译的，啥也看不到。

网上查题解，说可以用 IDA Pro 直接反编译出代码来，我？？？？

我试了试，只能说 woc 了。反编译得到如下代码：

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int result; // eax
  char Dest; // [rsp+20h] [rbp-80h]
  FILE *v5; // [rsp+88h] [rbp-18h]
  FILE *File; // [rsp+90h] [rbp-10h]
  char v7; // [rsp+9Fh] [rbp-1h]
  int v8; // [rsp+B0h] [rbp+10h]
  const char **v9; // [rsp+B8h] [rbp+18h]

  v8 = argc;
  v9 = argv;
  _main();
  if ( v8 <= 1 )
  {
    printf("Usage: %s [FileName]\n", *v9);
    printf("FileName是待加密的文件");
    exit(1);
  }
  File = fopen(v9[1], "rb+");
  if ( File )
  {
    v5 = fopen("tmp", "wb+");
    while ( feof(File) == 0 )
    {
      v7 = fgetc(File);
      if ( v7 != -1 && v7 )
      {
        if ( v7 > 47 && v7 <= 96 )
        {
          v7 += 53;
        }
        else if ( v7 <= 46 )
        {
          v7 += v7 % 11;
        }
        else
        {
          v7 = 61 * (v7 / 61);
        }
        fputc(v7, v5);
      }
    }
    fclose(v5);
    fclose(File);
    sprintf(&Dest, "del %s", v9[1]);
    system(&Dest);
    sprintf(&Dest, "ren tmp %s", v9[1]);
    system(&Dest);
    result = 0;
  }
  else
  {
    printf("无法打开文件%s\n", v9[1]);
    result = -1;
  }
  return result;
}
```

尝试了一下发现数据只会在 `v7 > 47 && v7 <= 96` 这个地方被加密，所以将数据全部减去 53 即可，得到

504354467B596F755F6172335F476F6F645F437261636B33527D

交了发现错了，检查了几遍后发现我的想法没问题，然后去看了下题解，原来得转这个 16 进制。。将其转换成字节得到

PCTF{You_ar3_Good_Crack3R}

## 答案

PCTF{You_ar3_Good_Crack3R}
