---
layout: post
title: Ajax Access-Control-Allow-Origin实现跨域访问用法
date: 2017-03-11
tag: Ajax
---


>欢迎大家关注我的其他<a herf="http://blog.csdn.net/u014377963">CSDN博客</a>和<a href="http://www.jianshu.com/u/a9f9d36ab057">简书</a>，互相交流！


如果跨域使用POST方式，可以使用创建一个隐藏的iframe来实现，与ajax上传图片原理一样，但这样会比较麻烦。
所以通过设置Access-Control-Allow-Origin来实现跨域访问比较简单。
下面看下我的demo，我的地址是www.text.com(配置虚拟主机)。

例如：客户端的域名是www.text.com,而请求的域名是www.text.com
如果直接使用ajax访问，会有以下错误
XMLHttpRequest cannot load http://www.server.com/ajax.PHP. No 'Access-Control-Allow-Origin' header is present on the requested resource.Origin 'http://www.text.com' is therefore not allowed access.

### 在被请求的Response header中加入

```
// 指定允许其他域名访问  
header('Access-Control-Allow-Origin:*');  
// 响应类型  
header('Access-Control-Allow-Methods:POST');  
// 响应头设置  
header('Access-Control-Allow-Headers:x-requested-with,content-type');  
```
就可以实现ajax POST跨域访问了。

代码如下：
demo.html 路径：`http://www.text.com/demo.html`

```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
 <head>
  <meta http-equiv="content-type" content="text/html;charset=utf-8">
  <title> AJAX跨域测试 </title>
  <script src="//code.jquery.com/jquery-1.11.3.min.js"></script>
 </head>

 <body>
    <div id="show"></div>
    <script type="text/javascript">
    $.post("./ajax.php",{status:"sucess",msg:"AJAX"})
      .done(function(data){
        document.getElementById("show").innerHTML = data.msg + '&nbsp;&nbsp;&nbsp;<B>' + data.status;
      });
    </script>
 </body>
</html>
```
ajax.php 路径：`http://www.text.com/ajax.php`

```

$data = [
    'status' => isset($_POST['status'])? $_POST['status'] : '',
    'msg' => isset($_POST['msg'])? $_POST['msg'] : ''
];

header('content-type:application:json;charset=utf8');  
header('Access-Control-Allow-Origin:*');  
header('Access-Control-Allow-Methods:POST');  
header('Access-Control-Allow-Headers:x-requested-with,content-type');  
  
echo json_encode($data);  
```

#### Access-Control-Allow-Origin:* 表示允许任何域名跨域访问

如果需要指定某域名才允许跨域访问，只需把`Access-Control-Allow-Origin:*改为Access-Control-Allow-Origin:允许的域名`
例如：
header('Access-Control-Allow-Origin:http://www.text.com');
要设置多个域名允许访问，这里需要用php处理一下
例如允许 www.text.com 与 www.text2.com 可以跨域访问

ajax.php 修改为
```
<?php

$data = [
    'status' => isset($_POST['status'])? $_POST['status'] : '',
    'msg' => isset($_POST['msg'])? $_POST['msg'] : ''
];

header('content-type:application:json;charset=utf8');
$origin = isset($_SERVER['HTTP_ORIGIN'])? $_SERVER['HTTP_ORIGIN'] : '';
$allow_origin = array(
    'http://www.text.com',
    'http://www.text1.com'
);

if(in_array($origin, $allow_origin)){
    header('Access-Control-Allow-Origin:'.$origin);
    header('Access-Control-Allow-Methods:POST');
    header('Access-Control-Allow-Headers:x-requested-with,content-type');
	echo json_encode($data);
}else{
	$rs = [
		'status'=>'error',
		'msg'=>'Cross domain is not allowed'
	];
	echo json_encode($rs);
}

?>
```
效果图如下：
![这里写图片描述](http://img.blog.csdn.net/20170311170219183?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)