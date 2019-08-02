## 描述

"没有什么防护是一个漏洞解决不了的，如果有，那
就.....
"

题目入口：http://web.jarvisoj.com:32785/

题目来源：ISCC2016

## 题解

尝试了一下，可以上传文件到 http://web.jarvisoj.com:32785/upload.php ，但限制了格式为 jpg 和 gif（而且是后端验证的），返回了一个图片 id。

然后可以用 http://web.jarvisoj.com:32785/show.php?id=xxxx&type=jpg 来浏览这个照片。并且可以直接用 http://web.jarvisoj.com:32785/uploads/xxxx.jpg 来访问照片。

这是不是可以尝试上传木马提权？不管了，先扫一下。没扫出什么多余的。

各种尝试之后，真的啥也没试出来。。。。。。。。。我太菜了。。。看一下提题解。

首先。。可以利用 `index.php?page=` 这个。。。因为发现这玩意会默认给参数后面加 `.php`，说明要执行他的，比如说

http://web.jarvisoj.com:32785/index.php?page=uploads/xxxx.jpg

会得到

```
Warning: fopen(uploads/xxxx.jpg.php): failed to open stream: No such file or directory in /opt/lampp/htdocs/index.php on line 24
No such file!
```

我们可以在 `xxxx.jpg` 后面插入了一段 PHP 的代码，然后倘若 PHP 能解析到我们的文件，那么就会执行。但是这玩意会在后面加 `.php`，这就不好办了。

所以去学习了一发 %00 截断的姿势，具体说就是直接用 `xxxx.jpg%00` 就好，如果你了解字符串的存储原理，那么你应该很清楚这是为什么。详细可以看 https://zhuanlan.zhihu.com/p/30005652 这篇文章。

在图片后插入的代码为 `<script language='php'>phpinfo();</script>` ，这是因为服务器会过滤掉 `<?`。

涨姿势了。

## 答案

CTF{upl0ad_sh0uld_n07_b3_a110wed}
