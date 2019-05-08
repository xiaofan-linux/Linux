# SAMBA实验2：本地用户验证

```
目的：公司有三个程序员，项目进行需要上传代码，
要求：每个人都有自己的上传位置
     每个人只能上传修改自己的目录下的文件，其他人过来只有下载的权限
     且要给主管一个下载查看的权限。
```

### 1.关闭SELinux和iptables防火墙

```
1.关闭SELinux
    查看SELinux是否开启:
            getenfoce
            sestatus
    永久关闭：vim /etc/selinux/conf
    临时关闭：setenforce 0
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

### 2.安装samba服务

```
1.查看系统是否安装samba服务
    rpm -q samba
2.安装samba服务（前提搭建好yum源）
    yum -y install samba
```

### 3.修改配置文件

```
[root@localhost ~]# vim /etc/samba/smb.conf

[daima] #自定义共享名称
        comment = All Printers    #共享描述 可选
        path = /tmp/daima/        #共享目录
        browseable = yes          #共享是否被查看
        writable = yes            #是否可写
        create mask = 644         #文件默认上传权限
        directory mask = 755      #创建目录默认权限

在/tmp/daima/目录下，分别三个程序创建单独的目录
    例如：chengxu1、chengxu2、chengxu3、zhuguan
```

### 4.创建三个系统用户，并且指定家目录和禁止系统登录

```
[root@localhost movie]# useradd -d /tmp/daima/chengxu1 -s /sbin/nologin chengxu1
[root@localhost movie]# useradd -d /tmp/daima/chengxu1 -s /sbin/nologin chengxu2
[root@localhost movie]# useradd -d /tmp/daima/chengxu1 -s /sbin/nologin chengxu3
[root@localhost movie]# useradd  -s /sbin/nologin chengxu3 -M zhuguan
```

### 5.生成samba密码

```
 [root@localhost ~]# pdbedit -a -u chengxu1
 [root@localhost ~]# pdbedit -a -u chengxu2
 [root@localhost ~]# pdbedit -a -u chengxu3
 [root@localhost ~]# pdbedit -a -u zhuguan
```

### 6.修改用户权限

##### 1.第一种方法

```
[root@localhost movie]# chown chengxu1 chengxu1/
[root@localhost movie]# chown chengxu2 chengxu2/
[root@localhost movie]# chown chengxu3 chengxu3/
[root@localhost movie]# ll
总用量 16
drwxr-xr-x. 4 chengxu1 root 4096 10月  4 16:57 chengxu1
drwxr-xr-x. 2 chengxu2 root 4096 10月  4 16:37 chengxu2
drwxr-xr-x. 2 chengxu3 root 4096 10月  4 16:30 chengxu3
```

##### 2.第二种方法
```
[root@localhost movie]# ll
总用量 16
drwxr-xr-x. 4 root root 4096 10月 4 16:57 chengxu1
drwxr-xr-x. 2 root root 4096 10月 4 16:37 chengxu2
drwxr-xr-x. 2 root root 4096 10月 4 16:30 chengxu3

分别把用户对应自己的目录赋予ACL权限：
   setfacl  -m u:chengxu1:7  /tmp/daima/chengxu1
   setfacl  -m u:chengxu2:7  /tmp/daima/chengxu2
   setfacl  -m u:chengxu3:7  /tmp/daima/chengxu3

```
###7.客户端测试
    客户端登录
        Linux端:
	    smbclient  -L 服务器IP   				#查看服务器共享
	    smbclient  -U 用户名 //服务器ip/共享名    #登录服务器共享
        Window端
	    \\服务器ip\共享名
	      net  use * /del  			#清空登录缓存 
