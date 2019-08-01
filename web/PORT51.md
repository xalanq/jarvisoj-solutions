## 描述

题目入口：http://web.jarvisoj.com:32770/

```html
<!DOCTYPE html>
<html>
<head>
<title>Web 100</title>
<style type="text/css">
	body {
		background:gray;
		text-align:center;
	}
</style>
</head>

<body>
	<h3>Please use port 51 to visit this site.</h3>	
</body>
</html>
```

```
Request URL: http://web.jarvisoj.com:32770/
Request Method: GET
Status Code: 200 OK
Remote Address: 120.26.131.152:32770
Referrer Policy: no-referrer-when-downgrade
Connection: Keep-Alive
Content-Length: 219
Content-Type: text/html; charset=UTF-8
Date: Thu, 01 Aug 2019 14:04:12 GMT
Keep-Alive: timeout=5, max=100
Server: Apache/2.4.18 (Unix) OpenSSL/1.0.2h PHP/5.6.21 mod_perl/2.0.8-dev Perl/v5.16.3
X-Powered-By: PHP/5.6.21
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Connection: keep-alive
Cookie: UM_distinctid=16c4870f86866b-09f63f7b86767a-c343162-190140-16c4870f86999e
Host: web.jarvisoj.com:32770
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36
```

## 题解

直接修改网址的端口根本访问不了，毕竟已经有一个 32770 端口了，51 端口又是什么鬼？

看题解吧。发现是用 `curl` 的 `--local-port` 指定本地的端口....所以这样就好

`$ curl web.jarvisoj.com:32770 --local-port 51`

但是似乎服务器炸了。。所以只能抄一下答案

## 答案

PCTF{M45t3r_oF_CuRl}
