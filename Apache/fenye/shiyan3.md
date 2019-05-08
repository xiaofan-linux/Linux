# 实验三：虚拟主机

###虚拟主机的分类：
    1.基于IP的虚拟主机：一台服务器，多个ip，搭建多个网站
    2.基于端口的虚拟主机：一台服务器，一个ip，搭建多个网站，每个网络使用不同端口访问
    3.基于名字的虚拟主机：一台服务器，一个ip，搭建多个网站，每个网站使用不同域名访问
###实验准备：
    1.域名解析：准备两个域名
        www.sohu.com
        www.sina.com
    2.没搭建DNS，只能手动添加到本地hosts文件内进行解析
        hosts
    3.网站主页目录规划
        在/htdocs/目录下分别创建sohu 和 sina 两个目录
        并且在分别在新建目录内创建index.html文件（分别写不一样的内容）
### 实验流程
#### 1.创建sohu和sina目录并分别创建index.html
```
[root@localhost htdocs]# mkdir sina
[root@localhost htdocs]# mkdir sohu
[root@localhost htdocs]# vim sina/index.html
[root@localhost htdocs]# vim sohu/index.html

```
#### 2.配置host文件
```
    windows host文件位置：C:\Windows\System32\drivers\etc\hosts
    linux   host文件位置：/etc/host
```
#### 3.修改Apache主配置文件
```
[root@localhost ~]# vim /usr/local/apache2/etc/httpd.conf
include etc/extra/httpd-vhosts.conf  #开启apache虚拟主机子配置文件
```
#### 4.修改虚拟主机子配置文件
```
[root@localhost ~]# vim /usr/local/apache2/etc/extra/httpd-vhosts.conf

<Directory "/usr/local/apache2/htdocs/sina">网页权限
        Options Indexes
        AllowOverride None
        Require all granted
</Directory>
<VirtualHost *:80>
    ServerAdmin sina@sina.com    #管理邮箱
    DocumentRoot "/usr/local/apache2/htdocs/sina"    #网页主目录
    ServerName www.sina.com    #完整域名
    ErrorLog "logs/sina-error_log"    #错误日志
    CustomLog "logs/sina-access_log" common    #正确日志    common：日志等级
</VirtualHost>


<Directory "/usr/local/apache2/htdocs/sohu">#网页权限
        Options Indexes
        AllowOverride None    
        Require all granted
</Directory>
<VirtualHost *:80>
    ServerAdmin sohu@sohu.com    #管理员邮箱
    DocumentRoot "/usr/local/apache2/htdocs/sohu"    #网页主目录
    ServerName www.sohu.com    #完整域名
    ErrorLog "logs/sohu-error_log"    #错误日志
    CustomLog "logs/sohu-access_log" common    #正确日志    common：日志等级
</VirtualHost>
```
#### 5.重启Apache服务
    /usr/local/apache2/bin/apachectl stop
    /usr/local/apache2/bin/apachectl start
    注：重启源码包安装的apache需要先关闭，再启动。
#### 6.测试
    浏览器输入：www.sohu.com
               www.sina.com
