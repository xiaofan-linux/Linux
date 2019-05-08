# 实验四：rewrite重写功能

##作用:
    域名跳转
    文件跳转
##目的：
    搜狐网站跳转新浪网站
#### 1.配置host文件
```
    windows host文件位置：C:\Windows\System32\drivers\etc\hosts
    linux   host文件位置：/etc/host
```
#### 2.打开主配置文件开启重写模块
```
[root@localhost ~]# vim /usr/local/apache2/etc/httpd.conf
LoadModule rewrite_module modules/mod_rewrite.so
```
#### 3.修改虚拟主机配置文件
```
[root@localhost ~]# vim /usr/local/apache2/etc/extra/httpd-vhosts.conf
<Directory "/usr/local/apache2/htdocs/sohu">
    Options Indexes FollowSymLinks #准许进过次目录连接到其他目录
    AllowOverride All  #允许目录下.htaccess文件中的权限生效
    Require all granted
</Directory>


yum安装：
RedirectMatch Permanent ^/(.*) http://xxxx.com/$1  
```
#### 4.创建规则匹配文件
```
[root@localhost ~]# vim /usr/local/apache2/htdocs/sohu/.htaccess
RewriteEngine on    #开启rewrite功能
RewriteCond %{HTTP_HOST} www.sohu.com    #把以www.sina.com	开头的内容赋值给HTTP_HOST变量
RewriteRule  .*   http://www.sina.com    #.*  输入任何地址，都跳转到http://www.sohu.com
```
#### 5.apache服务重启：
    /usr/local/apache2/bin/apachectl stop
    /usr/local/apache2/bin/apachectl start
    注：重启源码包安装的apache需要先关闭，再启动。
#### 6.测试
    浏览器输入：www.sohu.com
