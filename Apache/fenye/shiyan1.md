# 实验1-别名目录

### 目录别名作用
    可以用于网站扩展

##### 1.创建目录和网页文件
    [root@localhost ~]# mkdir -p /data/xiaofan
    [root@localhost ~]# vim /data/xiaofan/index.html
##### 2.主配置文件修改
```
    [root@localhost ~]# vim /usr/local/apache2/etc/httpd.conf
    Include etc/extra/httpd-autoindex.conf   # 开启apache系统别名子配置文件
    LoadModule negotiation_module modules/mod_negotiation.so #泛匹配模块
```
##### 3.修改子配置文件
```
    [root@localhost ~]# vim /usr/local/apache2/etc/extra/httpd-autoindex.conf
    Alias /xiaofan/ "/data/xiaofan/"  # xiaofan:别名  /data/xiaofan/:网页的具体目录
    <Directory "/data/xiaofan">
        Options Indexes MultiViews  # 1.允许泛匹配。2.开启浏览权限
        AllowOverride None  # .htaccess中权限不生效
        Require all granted # 允许所有人访问
    </Directory>
```
#### 4.重启Apache服务
    /usr/local/apache2/bin/apachectl stop
    /usr/local/apache2/bin/apachectl start
    注：重启源码包安装的apache需要先关闭，再启动。
#### 5.浏览器测试
    http://IP地址/xiaofan/


