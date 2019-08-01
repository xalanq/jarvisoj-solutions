## 描述

需要密码才能获得flag哦。

题目链接：http://web.jarvisoj.com:32772/

```html

<form action="/" method="post">
  password: <input type="text" name="pass" />
  <input type="submit" value="submit" />
</form>

```

```
Request URL: http://web.jarvisoj.com:32772/
Request Method: GET
Status Code: 200 OK
Remote Address: 120.26.131.152:32772
Referrer Policy: no-referrer-when-downgrade
Connection: Keep-Alive
Content-Length: 128
Content-Type: text/html; charset=UTF-8
Date: Thu, 01 Aug 2019 14:14:10 GMT
Hint: "select * from `admin` where password='".md5($pass,true)."'"
Keep-Alive: timeout=5, max=98
Server: Apache/2.4.18 (Unix) OpenSSL/1.0.2h PHP/5.6.21 mod_perl/2.0.8-dev Perl/v5.16.3
X-Powered-By: PHP/5.6.21
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cache-Control: max-age=0
Connection: keep-alive
Cookie: UM_distinctid=16c4870f86866b-09f63f7b86767a-c343162-190140-16c4870f86999e
Host: web.jarvisoj.com:32772
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36
```

## 题解

这提供的信息也太少了吧？？难道是爆破？？？

先提交一次，然后居然发现 response 头有这玩意

```
Hint: "select * from `admin` where password='".md5($pass,true)."'"
```

哈哈这不是轻松注入吗，然后发现自己构造的语句一点用没有。。。看来还是不太了解 SQL 的语法。。

查了下题解，发现需要利用 PHP 里 `md5` 函数的特性。。。

`md5(string, raw)`，若 `raw = true`，则返回二进制转成 ascii 的数据...

所以构造一个算了 md5 后子串有 `'or'` 之类的串就好了。

Google 了一下 `sql injection php md5`，发现了有趣的东西，下面就放一个串就好。

password: `129581926211651571912466741651878684928`

raw: `?T0D??o#??'or'8.N=?`

## 答案

PCTF{R4w_md5_is_d4ng3rous}
