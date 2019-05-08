# 实验2-用户认证

#### 1.建立要保护的目录和网页文件

```
    [root@localhost ~]# mkdir /data/renzheng
    [root@localhost ~]# vim /data/renzheng/index.html

```

#### 2.设置要保护目录的权限

```
[root@localhost ~]# vim /usr/local/apache2/etc/extra/httpd-autoindex.conf

Alias  /renzheng/  "/data/renzheng/"
<Directory "/data/renzheng">        #声明被保护目录
    Options Indexes
    AllowOverride All          #开启权限认证文件.htaccess
    Require all granted
</Directory>
* 注意：我把目录设置权限放在别名的子配置文件内，也可以放当到主配置文件。放在主配置文件里去掉Alias那句！
```

#### 3.在指定目录下创建权限文件\(给哪个目录设置保护，就在哪个目录创建权限文件\)

```
[root@localhost ~]# vim /data/renzheng/.htaccess
AuthName "Welcome"#提示信息
AuthType basic    #加密类型
AuthUserFile /data/apache.passwd    #密码文件，文件名自定义。（但是路径要对，使用绝对路径）
require valid-user    #允许密码文件中所有用户访问
```
#### 4.3.建立密码文件，加入允许访问的用户。（此用户和系统用户无关）
```
[root@localhost ~]# /usr/local/apache2/bin/htpasswd -c /data/apache.passwd test1
    -c  建立密码文件，只有添加第一个用户时，才能-c
[root@localhost ~]# /usr/local/apache2/bin/htpasswd -c /data/apache.passwd test1
    -m  再添加更多用户时，使用-m 参数
```
#### 4.apache服务重启：
    /usr/local/apache2/bin/apachectl stop
    /usr/local/apache2/bin/apachectl start
    注：重启源码包安装的apache需要先关闭，再启动。
#### 5.测试
    浏览器输入  http://IP地址/renzheng/
