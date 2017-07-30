
---
layout: post
title: Ajax Jsonp 跨域实例
date: 2017-03-11
tag: Ajax
---


>欢迎大家关注我的其他<a herf="http://blog.csdn.net/u014377963">CSDN博客</a>和<a href="http://www.jianshu.com/u/a9f9d36ab057">简书</a>，互相交流！




### 1.什么是jsonp：
JSONP(JSON with Padding)是一个非官方的协议，它允许在服务器端集成Script tags返回至客户端，通过javascript callback的形式实现跨域访问（这仅仅是JSONP简单的实现形式）。
### 2.JSONP有什么用？
由于同源策略的限制，XmlHttpRequest只允许请求当前源（域名、协议、端口）的资源，为了实现跨域请求，可以通过script标签实现跨域请求，然后在服务端输出JSON数据并执行回调函数，从而解决了跨域的数据请求。

### 3.如何使用JSONP？
下边这一DEMO实际上是JSONP的简单表现形式，在客户端声明回调函数之后，客户端通过script标签向服务器跨域请求数据，然后服务端返回相应的数据并动态执行回调函数。

下面是我自己做的一个简单的demo。实现的是306关键字搜索功能。采用的就是jsonp的跨域功能！
看例子
![这里写图片描述](http://img.blog.csdn.net/20170311160513672?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>推荐搜索</title>
</head>
<body>
	<p>推荐搜索</p>
	<input type="text" id="val" name ="wd" >
	<input type="button" id="dian" value="搜索">
	<ul></ul>
<script>
	function abc(res) {
		var wds = res['result'];
		for(var i =0,str=''; i<wds.length; i++) {
			str = str +  '<li>' + wds[i].word + '</li>';
		}
		document.getElementsByTagName('ul')[0].innerHTML = str;
	}

	document.getElementsByName('wd')[0].oninput = function () {
		var url = 'http://sug.so.360.cn/suggest?callback=abc&encodein=utf-8&encodeout=utf-8&format=json&word=' + this.value;
		var sc = document.createElement('script');
		sc.src = url;
		var hd = document.getElementsByTagName('head')[0];
		hd.appendChild(sc);
	}
	</script>

</body>
</html>
```


