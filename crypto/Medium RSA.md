## 描述

你没看错，这还真是密码学系列了，相信你已经解出前两题了，那么继续看这题吧。

[mediumRSA.rar.07aab25c9c54464a8c9821ca28503330](./assets/mediumRSA.rar.07aab25c9c54464a8c9821ca28503330)

## 题解

先看看公钥里的信息

```
$ RsaCtfTool.py --dumpkey --key pubkey.pem
[*] n: 87924348264132406875276140514499937145050893665602592992418171647042491658461
[*] e: 65537
```

n 不长不短，e 不特殊，应该是要暴力破解了。

```
$ RsaCtfTool.py --publickey pubkey.pem --uncipherfile flag.enc
[+] Clear text : b'\x00\x02\xc0\xfe\x04\xe3&\x0e[\x87\x00PCTF{256b_i5_m3dium}\n'
```

## 答案

PCTF{256b_i5_m3dium}
