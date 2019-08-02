## 描述

咦，奇怪，说好的WEB题呢，怎么成逆向了？不过里面有个help_me函数挺有意思的哦

[udf.so.02f8981200697e5eeb661e64797fc172](./assets/udf.so.02f8981200697e5eeb661e64797fc172)

## 题解

`strings` + `binwalk` 了下发现这玩意是个可执行文件，尝试执行一下，直接 `Segmentation fault` 了，想想 `.so` 的文件一般也是一个库文件对吧，应该得用啥东西来调用。

尝试用 IDA Pro 反编译一下，得到了一些能反编译的代码，但都是一些简单的函数（但发现了有一个 `getflag` 函数），而且 `help_me` 函数还反编译不了。逆向我是真的不太会，还是看题解吧

艹了，这是啥玩意，用 MySQL 来执行这个库文件。我就应该去谷歌一下 `udf.so` 这个文件名的意义的。。Naive 了。

> 知识点啊同志们！  
  扩展MySQL函数------ UDF  
  有时候我们需要对表中的数据进行一些处理而内置函数不能满足需要的时候，就需要对MySQL进行一些扩展，使用者自行添加的MySQL函数就称为UDF(User Define Function)。

做法就是

```
$ mysql
> select @@plugin_dir
```

得到插件的目录，将 `udf.so` 拷贝过去，然后再

```
$ mysql
> create function getflag returns string soname 'udf.so';
> select getflag();
```

便能得到 flag

## 答案

PCTF{Interesting_U5er_d3fined_Function}