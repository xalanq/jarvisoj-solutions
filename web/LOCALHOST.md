## 描述

题目入口：http://web.jarvisoj.com:32774/

## 题解

```html
<!DOCTYPE html>
<html>
<head>
<title>Web 150</title>
<style type="text/css">
	body {
		background:gray;
		text-align:center;
	}
</style>
</head>

<body>
	<h3>localhost access only!!</h3>	
</body>
</html>
```

一脸懵逼？？？？？看题解！

有题解说服务器会根据 header 中的 `X-FORWARDED-FOR` 来判断访问 ip，所以让该字段设为本地的即 127.0.0.1 即可。

查了一下 https://en.wikipedia.org/wiki/X-Forwarded-For ，发现的确是这样的。

所以用 `curl` 伪造一下就好了。

`$ curl web.jarvisoj.com:32774 -H "X-FORWARDED-FOR:127.0.0.1"`

所以这个告知我们 `X-FORWARDED-FOR` 是不安全的。

## 答案

PCTF{X_F0rw4rd_F0R_is_not_s3cuRe}
