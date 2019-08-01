## 描述

题目入口：http://web.jarvisoj.com:32774/

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

```
Request URL: http://web.jarvisoj.com:32774/
Request Method: GET
Status Code: 200 OK
Remote Address: 120.26.131.152:32774
Referrer Policy: no-referrer-when-downgrade
Connection: Keep-Alive
Content-Length: 205
Content-Type: text/html; charset=UTF-8
Date: Thu, 01 Aug 2019 14:02:52 GMT
Keep-Alive: timeout=5, max=100
Server: Apache/2.4.18 (Unix) OpenSSL/1.0.2h PHP/5.6.21 mod_perl/2.0.8-dev Perl/v5.16.3
X-Powered-By: PHP/5.6.21
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cache-Control: max-age=0
Connection: keep-alive
Cookie: UM_distinctid=16c4870f86866b-09f63f7b86767a-c343162-190140-16c4870f86999e
Host: web.jarvisoj.com:32774
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36
```

## 题解

一脸懵逼？？？？？看题解！

有题解说服务器会根据 header 中的 `X-FORWARDED-FOR` 来判断访问 ip，所以让该字段设为本地的即 127.0.0.1 即可。

查了一下 https://en.wikipedia.org/wiki/X-Forwarded-For ，发现的确是这样的。

所以用 `curl` 伪造一下就好了。

`$ curl web.jarvisoj.com:32774 -H "X-FORWARDED-FOR:127.0.0.1"`

所以这个告知我们 `X-FORWARDED-FOR` 是不安全的。

## 答案

PCTF{X_F0rw4rd_F0R_is_not_s3cuRe}
