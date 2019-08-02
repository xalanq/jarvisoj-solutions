## 描述

题目入口：http://web.jarvisoj.com:32770/

## 题解

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

直接修改网址的端口根本访问不了，毕竟已经有一个 32770 端口了，51 端口又是什么鬼？

看题解吧。发现是用 `curl` 的 `--local-port` 指定本地的端口....所以这样就好

`$ curl web.jarvisoj.com:32770 --local-port 51`

但是似乎服务器炸了。。所以只能抄一下答案

## 答案

PCTF{M45t3r_oF_CuRl}
