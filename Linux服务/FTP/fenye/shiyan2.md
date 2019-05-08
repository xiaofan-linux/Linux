# FTP实验1：本地用户验证实验

### 1.关闭SELinux和iptables防火墙

```1.关闭SELinux
        查看SELinux是否开启:          
              getenfoce       
              sestatus      
        永久关闭：   
              vim /etc/selinux/conf          
        临时关闭：      
              setenforce 0
2.关闭iptables防火墙    
        查看iptables是否开启：     
              iptables -L -n    
        临时关闭：          
               service iptables stop   
         永久关闭：        
               iptables -F        
               iptables -X        
               service iptables save
```

### 2.安装vsftpd服务

```
1.查看系统是否安装vsftpd服务    
        rpm -q vsftpd
2.安装vsftpd服务（前提搭建好yum源）  
         yum -y install vsftpf
```

### 3.创建本地用户（实验创建三个用户，设置密码   密码：123）

```
[root@localhost ~]# useradd user1
[root@localhost ~]# useradd user2
[root@localhost ~]# useradd user3
[root@localhost ~]# passwd user1
[root@localhost ~]# passwd user2
[root@localhost ~]# passwd user3
```

### 4.修改配置文件（以下配置根据需求配置）

```
    配置文件位置：/etc/vsftpd/vsftpd.conf
        local_root=/tmp/username    #当本地用户登入时，将被更换到定义的目录下。默认值为各用户的家目录。
    1.关闭匿名用户登录、开启本地用户登录
        anonymous_enable=NO    # 禁止匿名用户登录
        local_enable=YES       #开启本地用户登录
        write_enable=YES       #允许本地用户写权限
    2.控制用户是否允许切换到上级目录
        chroot_local_user=YES        #将用户禁锢在自己的家目录禁止切换目录
        chroot_list_enable=YES       #启用chroot_list_file配置项指定的用户列表文件
        chroot_list_file=/etc/vsftpd/chroot_list    #写此文件内的用户允许切换目录（注意：chroot_list文件默认不存在，需要手工创建）
    通过搭配能实现以下几种效果：
        ①当chroot_list_enable=YES，chroot_local_user=YES时，在/etc/vsftpd.chroot_list文件中列出的用户，可以切换到其他目录；未在文件中列出的用户，不能切换到其他目录。
        ②当chroot_list_enable=YES，chroot_local_user=NO时，在/etc/vsftpd.chroot_list文件中列出的用户，不能切换到其他目录；未在文件中列出的用户，可以切换到其他目录。
        ③当chroot_list_enable=NO，chroot_local_user=YES时，所有的用户均不能切换到其他目录。
        ④当chroot_list_enable=NO，chroot_local_user=NO时，所有的用户均可以切换到其他目录。
    3.禁止某些用户登录 
       /etc/vsftpd/ftpusers文件专门用于定义不允许访问FTP服务器的用户列表，vim打开将用户添加进去即可，立即生效。
    4.6.允许某些用户登录,其他人一概不允许登录
        userlist_enable=YES		    #开启用户访问控制
        userlist_deny=NO				
            #YES=黑名单、NO=白名单（默认是YES）
            #若设置为YES，则vsftpd.user_list文件中的用户不允许访问FTP，若设置为NO，则只有vsftpd.user_list文件中的用户才能访问FTP。
        userlist_file=/etc/vsftpd/user_list
    5.修改被动模式数据传输使用端口      
        a.关闭主动模式        
            connect_from_port_20=NO  #指定FTP使用20端口进行数据传输      
        b.开启被动模式        
            pasv_enable=YES         #开启被动模式传输        
            pasv_min_port=30000     #数据连接可以使用的端口范围的最小端口  
            pasv_max_port=30010     #数据连接可以使用的端口范围的最大端口
```
###4.启动VSFTP服务，并且客户端测试    
    [root@localhost ~]# service vsftpd restart    
    [root@localhost ~]# netstat -antp    
    tcp        0      0 0.0.0.0:21                  0.0.0.0:*                   LISTEN      2349/vsftpd


