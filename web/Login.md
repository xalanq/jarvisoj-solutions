## 描述

需要密码才能获得flag哦。

题目链接：http://web.jarvisoj.com:32772/

```html
<form action="/" method="post">
  password: <input type="text" name="pass" />
  <input type="submit" value="submit" />
</form>
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
