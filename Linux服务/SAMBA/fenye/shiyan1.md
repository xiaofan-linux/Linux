# SAMBA实验1：匿名用户验

```
目的：允许任何用户登录
要求： 每个人都能上传修改目录下的文件，可以下载文件
```

### 1.关闭SELinux和iptables防火墙

```

1.关闭SELinux

    查看SELinux是否开启:

        getenfoce

        sestatus

    永久关闭：vim \/etc\/selinux\/conf

        临时关闭：setenforce 
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
1.自定义共享
[root@localhost ~]# vim /etc/samba/smb.conf
[daima] #自定义共享名称
        comment = All Printers    #共享描述 可选
        path = /tmp/daima/        #共享目录
        browseable = yes          #共享是否被查看
        writable = yes            #是否可写
        create mask = 644         #文件默认上传权限
        directory mask = 755      #创建目录默认权限
        public = yes              #public用来指定该共享是否允许guest账户访问


2.修改用户访问验证方式
    security = share
  #设置用户访问Samba Server为匿名验证方式

```

### 4.修改共享目录权限

```
1.将共享目录权限设置为777   !!!生成环境禁止使用!!!
    chomd 777 /tmp/daima
2.将共享目录所有者修改为nobody伪用户
    chown nobody /tmp/daima
```

### 5.客户端测试
```
客户端登录 
    Linux端: 
       smbclient -L 服务器IP #查看服务器共享 
       smbclient  //服务器ip/共享名 #登录服务器共享 
    Window端: 
        \服务器ip\共享名 
        net use * \del #清空登录缓存
```

