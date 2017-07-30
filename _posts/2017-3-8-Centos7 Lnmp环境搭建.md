---
layout: post
title: fpm配置解释
date: 2017-03-6
tag: PHP
---




由于自己容易忘记，特写一篇博客记录Lnmp环境搭建。互相交流！
>欢迎大家关注我的其他<a herf="http://blog.csdn.net/u014377963">CSDN博客</a>和<a href="http://www.jianshu.com/u/a9f9d36ab057">简书</a>，互相交流！

本机环境：服务器是腾讯云；使用的镜像是：公共镜像 CENTOS 7.2

### **一、nginx安装**

1.下载对应当前系统版本的nginx包(package)

```
wget http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

2.建立nginx的yum仓库（默认yum是没有nginx的）

```
rpm -ivh nginx-release-centos-7-0.el7.ngx.noarch.rpm

```
3.下载并安装nginx

```
yum install nginx

```
4.nginx启动（nginx安装目录下－/usr/sbin/）

```
systemctl start nginx.service

```

ps：一些其它nginx相关命令：
nginx相关配置文件：
默认的配置文件在 /etc/nginx 路径下，使用该配置已经可以正确地运行nginx；
如需要自定义，修改其下的 nginx.conf 等文件即可；
在浏览器地址栏中输入部署nginx环境的机器的IP，如果一切正常，应该能看到如下字样的内容。
![这里写图片描述](http://img.blog.csdn.net/20170307122102404?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### **二、MYSQL安装**
1.先下载mysql的repo源；相关命令：

```
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
```
2.安装mysql-community-release-el7-5.noarch.rpm包

（安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo）

```
rpm -ivh mysql-community-release-el7-5.noarch.rpm
```
3.安装MYSQL

```
sudo yum install mysql-server

```

4.重置密码

更改用户权限：

```
sudo chown -R root:root /var/lib/mysql

```
重启服务：

```
systemctl restart mysql.service

```
登录，并修改密码：

```
mysql -u root
 mysql > use mysql;
 mysql > update user set password=password('123456') where user='root';
 mysql > exit;
```
###**三、安装php**
1.查看当前安装的php版本（ yum list installed | grep php）
如果存在php安装包先删除之前版本  用yum remove 移除 php相关的包

2.rpm 安装 Php7 相应的 yum源
```
rpm -Uvh https://mirror.webtatic.com/yum/el7/epel-release.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
```
3.安装php7.0

```
yum install php70w

```
4.安装PHP FPM

```
yum install php70w-fpm

```
这里解释下：关于 php－fpm ，PHP-FPM其实是PHP源代码的一个补丁，旨在将FastCGI进程管理整合进PHP包中。必须将它patch到你的PHP源代码中，在编译安装PHP后才可以使用。
现在我们可以在最新的PHP 5.3.2的源码树里下载得到直接整合了PHP-FPM的分支，据说下个版本会融合进PHP的主分支去。相对Spawn-FCGI，PHP-FPM在CPU和内存方面的控制都更胜一筹，而且前者很容易崩溃，必须用crontab进行监控，而PHP-FPM则没有这种烦恼。
PHP5.3.3已经集成php-fpm了，不再是第三方的包了。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置，比spawn-fcgi具有更多优点，所以被PHP官方收录了。在./configure的时候带 –enable-fpm参数即可开启PHP-FPM。
使用PHP-FPM来控制PHP-CGI的FastCGI进程

###**四、配置nginx**
修改配置文件之前记得备份

1.nginx配置文件位置：（/etc/nginx/conf.d/default.conf）
修改web root目录（如果没有需求也可以不用修改，使用默认即可）：
root /wwwdata/html;
将其中的

```
location ~.php$ {
	fastcgi_pass 127.0.0.1:9000;
	fastcgi_index index.php;
	fastcgi_param SCRIPT_FILENAME
	$document_root$fastcgi_script_name;
	include    fastcgi_params;
 }
```
改为

```
location / {
	root /wwwdata/html;
    index  index.php index.html index.htm;
}
```
然后再将

```
location ~ \.php$ {
	root           html;
	fastcgi_pass   127.0.0.1:9000;
	fastcgi_index  index.php;
	fastcgi_param  SCRIPT_FILENAME
	$document_root$fastcgi_script_name;
	include        fastcgi_params;
}
```
改为

```
location ~ \.php$ {
	root           root /wwwdata/html;
	fastcgi_pass   127.0.0.1:9000;
	fastcgi_index  index.php;
	fastcgi_param  SCRIPT_FILENAME
	$document_root$fastcgi_script_name;
	include        fastcgi_params;
}
```

2.php-fpm配置文件位置：（/etc/php-fpm.d/www.conf）
修改
user =nginx
group=nginx
3.启动nginx服务：

```
systemctl start nginx.service
```

如需设置开机自启使用以下命令：

```
sudo systemctl enable nginx.service
```
查看启动状态：

```
systemctl status nginx  

```
看到以下字眼说明启动成功！

```
Active: active (running) since 六 2016-11-19 13:40:04 CST; 50min ago

```
4.启动PHP-FPM：

```
systemctl start php-fpm.service

```
如需设置开机自启试用以下命令：

```
sudo systemctl enable php-fpm.service

```
查看启动状态：

```
systemctl status php-fpm.service 

```
看到以下字眼说明启动成功！
Active: active (running) since 六 2016-11-19 14:14:33 CST; 18min ago
至此，php＋mysql＋nginx 环境搭建完成！
最后，在web根目录下下一个php文件测试下；呼呼～～
为了更方便的访问修改数据库，需要安装mysql管理工具 phpMyAdmin
跳转web根目录：（根据之前设置跳转自己的网站根目录）

cd /wwwdata/html/
下载pma安装包：

```
wget https://files.phpmyadmin.net/phpMyAdmin/4.4.12/phpMyAdmin-4.4.12-all-languages.zip
```
解压安装包：

```
unzip phpMyAdmin-4.4.12-all-languages.zip

```
这里如果缺少 unzip 需要先安装unzip；

```
yum install unzip zip

```
重命名安装文件夹

```
mv phpMyAdmin-4.4.12-all-languages phpmyadmin
```

修改session存放目录权限：

```
chown -R nginx.nginx /var/lib/php/session
```

如果需要修改session根目录位置 需要修改位置：
>/etc/php.ini 中修改 session.save_path
由于 /etc/php-fpm.d/www.conf中 的 session.save_path  比php.ini优先级更高所以如果 存在  /etc/php-fpm.d/www.conf中 的 session.save_path 只需修改这个即可
访问http://youipaddress/phpmyadin，如果能访问上，那么就成功了！
以为这样就可以了 但是登录又出现问题！登陆不上 ！

做了一下修改：

1.修改/var/lib/php/session文件夹权限 770以上
2.在phpmyadmin目录下 config.sample.inc.php 中修改 $cfg'Servers'['user'] = 'root';$cfg'Servers'['password'] = '123456'; 为自己设置的用户名密码；