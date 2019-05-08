# FTP实验1：匿名用户访问

```
目的：允许任何用户登录要求： 每个人都能上传修改目录下的文件，可以下载文件
```

### 1.关闭SELinux和iptables防火墙

```
1.关闭SELinux  
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

### 3.修改配置文件（以下配置不一定全部配置，根据自己需求配置）

```
配置文件位置：/etc/vsftpd/vsftpd.conf
vsftpd默认情况下是可以下载、查看，但不能上传，默认文件路径/vat/ftp/

1.修改配置文件允许匿名用户上传
    a.anon_upload_enable=YES  #允许匿名用户上传
    b.在/var/ftp/下创建上传目录
        [root@localhost ~]# mkdir /var/ftp/pub

    c.修改上传目录的权限或者所有者，让匿名用户有写入权限
        [root@localhost ~]# chown ftp /var/ftp/pub/
    d.修改匿名用户上传默认mask权限
        anon_umadk=073
2.修改配置文件匿名用户创建目录或文件,并且能修改文件或目录名称
        anon_mkdir_write_enable=YES	#允许匿名用户创建目录
        anon_other_write_enable=YES	#允许匿名用户修改名称
3.用户进入某个文件夹时，弹出相应的说明
      a.在对应目录下创建 .message 文件，并写入相应内容
      b.确认dirmessage_enable=YES是否启用
      c.尝试却换目录查看效果（同一次登录仅提示一次）
4.修改被动模式数据传输使用端口
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

