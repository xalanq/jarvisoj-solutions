## 描述

这么简单的题，是WEB吗？

题目入口：http://web.jarvisoj.com:9891/

## 题解

发现是 react 写的，渲染后的 html 如下

```html
<html lang="en" class="wf-roboto-n3-active wf-roboto-n4-active wf-roboto-n5-active wf-active"><head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <title>QWB</title>

    <!-- Use minimum-scale=1 to enable GPU rasterization -->
    <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=0, maximum-scale=1, minimum-scale=1">
    <link rel="stylesheet" type="text/css" href="css/main.css">
    <link href="//cdn.bootcss.com/material-design-icons/3.0.1/iconfont/material-icons.min.css" rel="stylesheet">
<link rel="stylesheet" href="http://fonts.googleapis.com/css?family=Roboto:400,300,500&amp;subset=latin" media="all"><style type="text/css" abt="234"></style></head>
<body style="">
<div id="app"><div data-reactroot="" style="margin: 10px;"><!-- react-empty: 2 --><div style="font-size: 16px; line-height: 24px; width: 256px; height: 48px; display: inline-block; position: relative; background-color: transparent; font-family: Roboto, sans-serif; transition: height 200ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;"><div style="position: absolute; opacity: 1; color: rgba(0, 0, 0, 0.3); transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms; bottom: 12px;">Password</div><input type="password" value="" id="pass" style="padding: 0px; position: relative; width: 100%; border: none; outline: none; background-color: rgba(0, 0, 0, 0); color: rgba(0, 0, 0, 0.87); cursor: initial; font: inherit; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); height: 100%;"><div><hr style="border-top: none rgb(224, 224, 224); border-right: none rgb(224, 224, 224); border-bottom: 1px solid rgb(224, 224, 224); border-left: none rgb(224, 224, 224); border-image: initial; bottom: 8px; box-sizing: content-box; margin: 0px; position: absolute; width: 100%;"><hr style="border-top: none rgb(0, 188, 212); border-right: none rgb(0, 188, 212); border-bottom: 2px solid rgb(0, 188, 212); border-left: none rgb(0, 188, 212); border-image: initial; bottom: 8px; box-sizing: content-box; margin: 0px; position: absolute; width: 100%; transform: scaleX(0); transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms;"></div><!-- react-text: 9 --><!-- /react-text --></div><div style="color: rgba(0, 0, 0, 0.87); background-color: rgb(255, 255, 255); transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms; box-sizing: border-box; font-family: Roboto, sans-serif; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); box-shadow: rgba(0, 0, 0, 0.12) 0px 1px 6px, rgba(0, 0, 0, 0.12) 0px 1px 4px; border-radius: 2px; display: inline-block; min-width: 88px; vertical-align: top; margin-top: 3px;"><button tabindex="0" type="button" style="border: 10px; box-sizing: border-box; display: inline-block; font-family: Roboto, sans-serif; -webkit-tap-highlight-color: rgba(0, 0, 0, 0); cursor: pointer; text-decoration: none; margin: 0px; padding: 0px; outline: none; font-size: inherit; font-weight: inherit; transform: translate(0px, 0px); position: relative; height: 36px; line-height: 36px; width: 100%; border-radius: 2px; transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms; background-color: rgb(0, 188, 212); text-align: center;"><div><div style="height: 36px; border-radius: 2px; transition: all 450ms cubic-bezier(0.23, 1, 0.32, 1) 0ms; top: 0px;"><span style="position: relative; opacity: 1; font-size: 14px; letter-spacing: 0px; text-transform: uppercase; font-weight: 500; margin: 0px; user-select: none; padding-left: 16px; padding-right: 16px; color: rgb(255, 255, 255);">Check</span></div></div></button></div></div></div>

<!-- This script adds the Roboto font to our project. For more detail go to this site:  http://www.google.com/fonts#UsePlace:use/Collection:Roboto:400,300,500 -->
<script src="http://cdn.bootcss.com/webfont/1.6.26/webfontloader.js" type="text/javascript" async=""></script><script>
    var WebFontConfig = {
        google: { families: [ 'Roboto:400,300,500:latin' ] }
    };
    (function() {
        var wf = document.createElement('script');
        wf.src = wf.src = ('https:' == document.location.protocol ? 'https' : 'http') +
                '://cdn.bootcss.com/webfont/1.6.26/webfontloader.js';
        wf.type = 'text/javascript';
        wf.async = 'true';
        var s = document.getElementsByTagName('script')[0];
        s.parentNode.insertBefore(wf, s);
    })();
</script>
<script src="//cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
<script src="app.js"></script><style>
      button::-moz-focus-inner,
      input::-moz-focus-inner {
        border: 0;
        padding: 0;
      }
    </style>

</body></html>
```

`app.js` 我下载到了[本地](./assets/app.js)，太长了，稍后再看。header 没啥有用的信息。

先尝试提交几次，发现是表单 `passowrd: ' or 1=1--` POST 到了 `http://web.jarvisoj.com:9891/checkpass.json`，尝试注入没啥用。再去看看 `app.js` 吧。

搜索 `password` 字眼，找到一份这个代码

```js
key: "__handleTouchTap__REACT_HOT_LOADER__",
value: function() {
    var e = this.state.passcontent,
        t = {
            passowrd: e
        };
    self = this, $.post("checkpass.json", t, function(t) {
        self.checkpass(e) ? self.setState({
            errmsg: "Success!!",
            errcolor: b.green400
        }) : (self.setState({
            errmsg: "Wrong Password!!",
            errcolor: b.red400
        }), setTimeout(function() {
            self.setState({
                errmsg: ""
            })
        }, 3e3))
    })
}
```

woc？本地验证？看看 `checkpass` 这个函数。

```js
}, r.checkpass = function() {
    var e;
    return (e = r).__checkpass__REACT_HOT_LOADER__.apply(e, arguments)
}, r.handleTouchTap = function() {
```

在找找 `checkpass`，得到这份代码

```js
}, {
key: "__checkpass__REACT_HOT_LOADER__",
value: function(e) {
    if (25 !== e.length) return !1;
    for (var t = [], n = 0; n < 25; n++) t.push(e.charCodeAt(n));
    for (var r = [325799, 309234, 317320, 327895, 298316, 301249, 330242, 289290, 273446, 337687, 258725, 267444, 373557, 322237, 344478, 362136, 331815, 315157, 299242, 305418, 313569, 269307, 338319, 306491, 351259], o = [
            [11, 13, 32, 234, 236, 3, 72, 237, 122, 230, 157, 53, 7, 225, 193, 76, 142, 166, 11, 196, 194, 187, 152, 132, 135],
            [76, 55, 38, 70, 98, 244, 201, 125, 182, 123, 47, 86, 67, 19, 145, 12, 138, 149, 83, 178, 255, 122, 238, 187, 221],
            [218, 233, 17, 56, 151, 28, 150, 196, 79, 11, 150, 128, 52, 228, 189, 107, 219, 87, 90, 221, 45, 201, 14, 106, 230],
            [30, 50, 76, 94, 172, 61, 229, 109, 216, 12, 181, 231, 174, 236, 159, 128, 245, 52, 43, 11, 207, 145, 241, 196, 80],
            [134, 145, 36, 255, 13, 239, 212, 135, 85, 194, 200, 50, 170, 78, 51, 10, 232, 132, 60, 122, 117, 74, 117, 250, 45],
            [142, 221, 121, 56, 56, 120, 113, 143, 77, 190, 195, 133, 236, 111, 144, 65, 172, 74, 160, 1, 143, 242, 96, 70, 107],
            [229, 79, 167, 88, 165, 38, 108, 27, 75, 240, 116, 178, 165, 206, 156, 193, 86, 57, 148, 187, 161, 55, 134, 24, 249],
            [235, 175, 235, 169, 73, 125, 114, 6, 142, 162, 228, 157, 160, 66, 28, 167, 63, 41, 182, 55, 189, 56, 102, 31, 158],
            [37, 190, 169, 116, 172, 66, 9, 229, 188, 63, 138, 111, 245, 133, 22, 87, 25, 26, 106, 82, 211, 252, 57, 66, 98],
            [199, 48, 58, 221, 162, 57, 111, 70, 227, 126, 43, 143, 225, 85, 224, 141, 232, 141, 5, 233, 69, 70, 204, 155, 141],
            [212, 83, 219, 55, 132, 5, 153, 11, 0, 89, 134, 201, 255, 101, 22, 98, 215, 139, 0, 78, 165, 0, 126, 48, 119],
            [194, 156, 10, 212, 237, 112, 17, 158, 225, 227, 152, 121, 56, 10, 238, 74, 76, 66, 80, 31, 73, 10, 180, 45, 94],
            [110, 231, 82, 180, 109, 209, 239, 163, 30, 160, 60, 190, 97, 256, 141, 199, 3, 30, 235, 73, 225, 244, 141, 123, 208],
            [220, 248, 136, 245, 123, 82, 120, 65, 68, 136, 151, 173, 104, 107, 172, 148, 54, 218, 42, 233, 57, 115, 5, 50, 196],
            [190, 34, 140, 52, 160, 34, 201, 48, 214, 33, 219, 183, 224, 237, 157, 245, 1, 134, 13, 99, 212, 230, 243, 236, 40],
            [144, 246, 73, 161, 134, 112, 146, 212, 121, 43, 41, 174, 146, 78, 235, 202, 200, 90, 254, 216, 113, 25, 114, 232, 123],
            [158, 85, 116, 97, 145, 21, 105, 2, 256, 69, 21, 152, 155, 88, 11, 232, 146, 238, 170, 123, 135, 150, 161, 249, 236],
            [251, 96, 103, 188, 188, 8, 33, 39, 237, 63, 230, 128, 166, 130, 141, 112, 254, 234, 113, 250, 1, 89, 0, 135, 119],
            [192, 206, 73, 92, 174, 130, 164, 95, 21, 153, 82, 254, 20, 133, 56, 7, 163, 48, 7, 206, 51, 204, 136, 180, 196],
            [106, 63, 252, 202, 153, 6, 193, 146, 88, 118, 78, 58, 214, 168, 68, 128, 68, 35, 245, 144, 102, 20, 194, 207, 66],
            [154, 98, 219, 2, 13, 65, 131, 185, 27, 162, 214, 63, 238, 248, 38, 129, 170, 180, 181, 96, 165, 78, 121, 55, 214],
            [193, 94, 107, 45, 83, 56, 2, 41, 58, 169, 120, 58, 105, 178, 58, 217, 18, 93, 212, 74, 18, 217, 219, 89, 212],
            [164, 228, 5, 133, 175, 164, 37, 176, 94, 232, 82, 0, 47, 212, 107, 111, 97, 153, 119, 85, 147, 256, 130, 248, 235],
            [221, 178, 50, 49, 39, 215, 200, 188, 105, 101, 172, 133, 28, 88, 83, 32, 45, 13, 215, 204, 141, 226, 118, 233, 156],
            [236, 142, 87, 152, 97, 134, 54, 239, 49, 220, 233, 216, 13, 143, 145, 112, 217, 194, 114, 221, 150, 51, 136, 31, 198]
        ], n = 0; n < 25; n++) {
        for (var i = 0, a = 0; a < 25; a++) i += t[a] * o[n][a];
        if (i !== r[n]) return !1
    }
    return !0
}
```

仔细分析一下，这像是一个矩阵乘法，r 和 t 是 25 x 1 的向量，然后 o 是 25 x 25 的矩阵，然后验证 o * t 是否等于 r。

假若 o 有逆，那么直接可以得到答案 t = inv(o) * t，尝试一下，还真有逆，那么很快就能得到答案了。

```python
import numpy as np

r = np.array([325799, 309234, 317320, 327895, 298316, 301249, 330242, 289290, 273446, 337687, 258725, 267444, 373557, 322237, 344478, 362136, 331815, 315157, 299242, 305418, 313569, 269307, 338319, 306491, 351259])
o = np.array([
    [11, 13, 32, 234, 236, 3, 72, 237, 122, 230, 157, 53, 7, 225, 193, 76, 142, 166, 11, 196, 194, 187, 152, 132, 135],
    [76, 55, 38, 70, 98, 244, 201, 125, 182, 123, 47, 86, 67, 19, 145, 12, 138, 149, 83, 178, 255, 122, 238, 187, 221],
    [218, 233, 17, 56, 151, 28, 150, 196, 79, 11, 150, 128, 52, 228, 189, 107, 219, 87, 90, 221, 45, 201, 14, 106, 230],
    [30, 50, 76, 94, 172, 61, 229, 109, 216, 12, 181, 231, 174, 236, 159, 128, 245, 52, 43, 11, 207, 145, 241, 196, 80],
    [134, 145, 36, 255, 13, 239, 212, 135, 85, 194, 200, 50, 170, 78, 51, 10, 232, 132, 60, 122, 117, 74, 117, 250, 45],
    [142, 221, 121, 56, 56, 120, 113, 143, 77, 190, 195, 133, 236, 111, 144, 65, 172, 74, 160, 1, 143, 242, 96, 70, 107],
    [229, 79, 167, 88, 165, 38, 108, 27, 75, 240, 116, 178, 165, 206, 156, 193, 86, 57, 148, 187, 161, 55, 134, 24, 249],
    [235, 175, 235, 169, 73, 125, 114, 6, 142, 162, 228, 157, 160, 66, 28, 167, 63, 41, 182, 55, 189, 56, 102, 31, 158],
    [37, 190, 169, 116, 172, 66, 9, 229, 188, 63, 138, 111, 245, 133, 22, 87, 25, 26, 106, 82, 211, 252, 57, 66, 98],
    [199, 48, 58, 221, 162, 57, 111, 70, 227, 126, 43, 143, 225, 85, 224, 141, 232, 141, 5, 233, 69, 70, 204, 155, 141],
    [212, 83, 219, 55, 132, 5, 153, 11, 0, 89, 134, 201, 255, 101, 22, 98, 215, 139, 0, 78, 165, 0, 126, 48, 119],
    [194, 156, 10, 212, 237, 112, 17, 158, 225, 227, 152, 121, 56, 10, 238, 74, 76, 66, 80, 31, 73, 10, 180, 45, 94],
    [110, 231, 82, 180, 109, 209, 239, 163, 30, 160, 60, 190, 97, 256, 141, 199, 3, 30, 235, 73, 225, 244, 141, 123, 208],
    [220, 248, 136, 245, 123, 82, 120, 65, 68, 136, 151, 173, 104, 107, 172, 148, 54, 218, 42, 233, 57, 115, 5, 50, 196],
    [190, 34, 140, 52, 160, 34, 201, 48, 214, 33, 219, 183, 224, 237, 157, 245, 1, 134, 13, 99, 212, 230, 243, 236, 40],
    [144, 246, 73, 161, 134, 112, 146, 212, 121, 43, 41, 174, 146, 78, 235, 202, 200, 90, 254, 216, 113, 25, 114, 232, 123],
    [158, 85, 116, 97, 145, 21, 105, 2, 256, 69, 21, 152, 155, 88, 11, 232, 146, 238, 170, 123, 135, 150, 161, 249, 236],
    [251, 96, 103, 188, 188, 8, 33, 39, 237, 63, 230, 128, 166, 130, 141, 112, 254, 234, 113, 250, 1, 89, 0, 135, 119],
    [192, 206, 73, 92, 174, 130, 164, 95, 21, 153, 82, 254, 20, 133, 56, 7, 163, 48, 7, 206, 51, 204, 136, 180, 196],
    [106, 63, 252, 202, 153, 6, 193, 146, 88, 118, 78, 58, 214, 168, 68, 128, 68, 35, 245, 144, 102, 20, 194, 207, 66],
    [154, 98, 219, 2, 13, 65, 131, 185, 27, 162, 214, 63, 238, 248, 38, 129, 170, 180, 181, 96, 165, 78, 121, 55, 214],
    [193, 94, 107, 45, 83, 56, 2, 41, 58, 169, 120, 58, 105, 178, 58, 217, 18, 93, 212, 74, 18, 217, 219, 89, 212],
    [164, 228, 5, 133, 175, 164, 37, 176, 94, 232, 82, 0, 47, 212, 107, 111, 97, 153, 119, 85, 147, 256, 130, 248, 235],
    [221, 178, 50, 49, 39, 215, 200, 188, 105, 101, 172, 133, 28, 88, 83, 32, 45, 13, 215, 204, 141, 226, 118, 233, 156],
    [236, 142, 87, 152, 97, 134, 54, 239, 49, 220, 233, 216, 13, 143, 145, 112, 217, 194, 114, 221, 150, 51, 136, 31, 198]
])

inv = np.linalg.inv(o)
t = np.matmul(inv, r)
print(t)
```

得到

```
array([ 81.,  87.,  66., 123.,  82.,  51.,  97.,  99.,  55.,  95.,  49.,
       115.,  95., 105., 110., 116., 101., 114., 101., 115., 116., 105.,
       110., 103., 125.])
```

再转成 `ascii`

```python
s = ''
for x in t:
    s += chr(int(x + 0.5))
```

得到 flag

## 答案

QWB{R3ac7_1s_interesting}
