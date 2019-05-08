### 企业级邮件服务

#### 一、配置DNS服务

```
    1. 第一台上搭建主服务器如下 
       a. 安装bind软件  
       b. 主配置文件的修改  
       c. 区域配置文件的修改  
       d. 配置数据文件   
             正向数据文件  
       e. 启动named服务  
       f. 客户端测试
```

##### 1. 关闭SELinux和iptables防火墙

```
    关闭SELinux  
        永久关闭：vim /etc/selinux/conf  
        临时关闭：setenforce 0
    关闭iptables防火墙 
        临时关闭：service iptables stop   
        永久关闭：iptables -F         
                 iptables -X              
                 service iptables save
```

##### 2.安装DNS服务

```
     yum -y install bind
```

##### 3.修改主配置文件

```python
options{ 
         listen-on port 53 { any; }; #声明服务的端口和对外监听的地址  （监听所有网卡用any）                    
         listen-on-v6 port 53 { ::1; };      
         directory       "/var/named";   
         dump-file       "/var/named/data/cache_dump.db";     
         statistics-file "/var/named/data/named_stats.txt";      
         memstatistics-file "/var/named/data/named_mem_stats.txt";  
         allow-query     { any; }; #允许哪些客户端访问     
         recursion yes;
```

#### 4.修改位置区域文件

```python
zone "extmail.org" IN { #指明增加的DNS域的名称    
           type master;  #指明DNS为主DNS服务器    
           file "extmail.localhost"; #设置实现正向域名解析的区域文件名称       
           allow-update { none; }; #设置该DNS不允许动态更新};
```

##### 5.配置数据文件  \/var\/named\/

```python
A．先复制生成正向解析文件和反向解析文件    cp -a named.localhost named.localhost.xiaofan #复制正向解析文件
B．编辑正向解析文件（注意域名结尾的 “.”）
[root@localhost named]# vim named.localhost.xiaofan    
$TTL 1D    @       IN SOA  extmail.org. rname.invalid. ( #xiaofan.com. 写解析的域名                                        
                            0       ; serial             
                            1D      ; refresh                                        
                            1H      ; retry            
                            1W      ; expire       
                            3H )    ; minimum      
       NS      dns.extmail.org.
       MX 3    mail.extmail.org. #邮件解析
dns     A       192.168.21.70      
mail     A       192.168.21.69   #表示正向邮件解析的对应关系
```

##### 5.重启DNS服务

```
    service named restart
```

##### 6.客户端测试

```
    在网卡配置文件中添加DNS服务器的地址，然后用nslookup测试
```

=====================================================================================================

#### 二、搭建邮件发信服务（postfix模块）

```
需要的软件：
    postfix
    mysql
    extman-1.1.tar.gz
    extmail-1.2.tar.gz
    Unix-Syslog-1.1.tar.gz

```

##### 1、安装gcc编译工具

```

[root@server ~]#
 yun -y install gcc*
```

##### 2、安装mysql mysql-server mailx

```
[root@server ~]#
 yum -y install mysql mysql-server mailx
[root@server ~]#
 /etc/init.d/mysqld restart  #启动数据库
```

##### 3、创建一个目录（用来存储所有邮件服务的配置）

```
[root@server ~]#
 mkdir -p /var/www/extsuite
```

##### 4、解压extman、extmail到上一步创建的目录

```
[root@server ~]#
 tar -xf extman-1.1.tar.gz -C /var/www/extsuite/
[root@server ~]#
 tar -xf extmail-1.2.tar.gz -C /var/www/extsuite/
```

##### 5、修改这个解压的文件名

```
[root@server ~]#
 cd /var/www/extsuite/
[root@server extsuite]#
 mv extman-1.1/ extman
[root@server extsuite]#
 mv extmail-1.2/ extmail
```

##### 6、将\/extman\/docs中模板和数据导入到数据库中

```
1.导入extmail.sql到数据库
    [root@postfix-server docs]# mysql < extmail.sql
2.将init.sql该一下再导入数据库中（最后一部分）（先修改init.sql文件）
    修改root@extmail.org的密码为明文密码：123456
    导入到数据库
    [root@server docs]# mysql < init.sql
```

##### 7、将虚拟目录的模板拷贝到邮件服务器的主目下（\/var\/www\/extsuite\/extman\/docs）

```
[root@server docs] cp mysql_virtual_alias_maps.cf mysql_virtual_domains_maps.cf mysql_virtual_mailbox_maps.cf /etc/postfix/
```

##### 8、创建真实的映射用户（UID号不要被系统用户占用）

```
[root@server ~]# useradd -u 600 vmail
```

#### 9、修改邮件服务的主配置文件（vim \/etc\/postfix\/main.cf）

```
注意：postfix默认系统是安装，如果没有安装yum安装一下
    [root@server ~]# vim /etc/postfix/main.cf
    inet_interfaces = all  #监听网卡该为all
在文件最后面添加添加如下参数：
    virtual_mailbox_base = /home/vmail  #接收到的邮件保存位置。保证在vmail用户的家目录
    virtual_uid_maps = static:600
    virtual_gid_maps = static:600
    virtual_alias_maps = mysql:/etc/postfix/mysql_virtual_alias_maps.cf
    virtual_mailbox_maps = mysql:/etc/postfix/mysql_virtual_mailbox_maps.cf
    virtual_mailbox_domains = mysql:/etc/postfix/mysql_virtual_domains_maps.cf
```

##### 10、重启postfix服务

```
[root@server ~]# service postfix restart
[root@server ~]# netstat -antp | grep postfix
```

##### 11、测试

```
[root@server ~]# echo "hi" | mail -s test support@extmail.org
```

##### 12、进入vmail家牡目录下是否有文件

```
[root@server ~]# ls /home/vmail/
extmail.org
```

=====================================================================================================

#### 三、搭建邮件收信服务\(dovecot模块\)

##### 1、安装dovecot服务

```
[root@server ~]# yum -y install dovecot dovecot-mysql
```

##### 2、修改dovecot里面的配置文件

```
    1).修改/etc/dovecot/conf.d/ 10-mail.conf  #用来设置dovecot收取邮件的配置
        mail_location = maildir:/home/vmail/%d/%n/Maildir  #定义dovecot去哪里找邮件。%d：显示地址中域的部分。%n：邮件地址中用户部分。
        first_valid_uid = 600  #设置虚拟用户收邮件UID从600开始往拍
    2).修改/etc/dovecot/conf.d/10-auth.conf  #dovecot收取邮件的认证方式
        !include auth-sql.conf.ext #表示认证通过数据库认证
```

##### 3、修改如何在数据库里读取数据的文件（需要拷贝模板）

```
1.拷贝模板
   [root@server ~]# cp /usr/share/doc/dovecot-2.0.9/example-config/dovecot-sql.conf.ext /etc/dovecot/

2.修改配置文件(dovecot-sql.conf.ext)

    driver = mysql  #数据库类型
    connect = host=localhost dbname=extmail user=extmail password=extmail #数据库的连接
    default\_pass\_scheme = MD5  #加密方式

    password_query = SELECT username, domain, password FROM mailbox WHERE username = '%u' AND domain = '%d'  #检索密码的sql语句

    user\_query = SELECT maildir, 600 AS uid, 600 AS gid FROM mailbox WHERE username = '%u' #查询用户
```

##### 4、启动dovecot服务

```
[root@server ~]# service dovecot start
```

=====================================================================================================

#### 三、搭建MAIL+WEB

##### 1、安装Apache服务

```
[root@server ~]# yum -y install httpd
```

##### 2、修改Apache配置文件(/etc/httpd/conf/httpd.conf)

```
NameVirtualHost *:80 #开启

<VirtualHost *:80>
    DocumentRoot /var/www/extsuite/extmail/html  #首页文件位置
    ServerName mail.extmail.org  #邮件服务的域名
    scriptalias /extmail/cgi /var/www/extsuite/extmail/cgi  #访问extmail的时候，设置cgi如何调用
    alias /extmail /var/www/extsuite/extmail/html  #访问extmail时，目录的别名
    scriptalias /extman/cgi /var/www/extsuite/extman/cgi  ##访问extman的时候，设置cgi如何调用
    alias /extman /var/www/extsuite/extman/html  ##访问extman时，目录的别名
    suexecusergroup vmail vmail  #设置运行extman和extmail的所有者和所属组
</VirtualHost>
```

##### 3、extmail中更改cgi的属组属主，让vmail有权限执行 ，生成主配置文件，修改主配置文件

```
1、修改属组属主  
    [root@server extmail]# chown -R vmail:vmail cgi/
2、生成配置文件，
    [root@server extmail]# cp webmail.cf.default webmail.cf
3、修改配置文件
    [root@bogon extmail]# vim webmail.cf
        SYS_MAILDIR\_BASE = /home/vmail #邮件接收目录
        SYS_CRYPT_TYPE = plain #加密方式
        SYS_MYSQL_USER = extmail #数据库用户
        SYS_MYSQL_PASS = extmail #数据库密码

```
##### 3、extman中更改cgi的属组属主，让vmail有权限执行，生成主配置文件，修改主配置文件
```
1、修改属组属者
    [root@server extman]# chown -R vmail:vmail cgi/
2.生成主配置文件
    [root@server extman]# cp webman.cf.default webman.cf
3、修改配置文件
    [root@bogon extman]# vim webman.cf
        SYS_MAILDIR_BASE = /home/vmai # 邮件接收目录
        SYS_SESS_DIR = /tmp  #临时会话
        SYS_CAPTCHA_ON = 0   #校验码
        SYS_CRYPT_TYPE = plain #加密方式
```
=====================================================================================================

### 安装Unix-Syslog这个软件

```
tar -xd Unix-Syslog-1.1.tar.gz
cd Unix-Syslog-1.1
perl Makefile.PL
make
make install
```

## 在客户点测试DNS地址指向DNS服务器，在浏览器上访问

