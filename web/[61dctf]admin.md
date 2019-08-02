## 描述

题目入口：http://web.jarvisoj.com:32792/

## 题解

```html
Hello World
```

就啥也没了！Header 里也是没啥有用信息，只知道一些服务器信息

```
Server: Apache/2.4.18 (Unix) OpenSSL/1.0.2h PHP/5.6.21 mod_perl/2.0.8-dev Perl/v5.16.3
X-Powered-By: PHP/5.6.21
```

查了一下发现根本没收获。。只能看题解。

woc....得用 robots.txt....

打开 http://web.jarvisoj.com:32792/robots.txt 发现有 `Disallow: /admin_s3cr3t.php`，再打开这玩意 http://web.jarvisoj.com:32792/admin_s3cr3t.php 得到 `flag{hello guest}`。

试了下发现是错的 flag...

看到 request header 和 response header 的 cookie 操作都有一个 `admin=0`，猜测是这里有问题，改成 `admin=1` 便得到了 flag。

## 答案

flag{hello_admin~}
