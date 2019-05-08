## FTP实验3：虚拟用户验证实验

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
        yum -y install vsftpd
```

### 3.建立FTP的虚拟用户的用户数据库文件（在\/etc\/vsftpd）

```
    a.创建虚拟用户表单
            vim vsftpd.user
            user1
            123456
            user2
            123456
            user3
            123456
        注：该文件名可以随便定义，文件内容格式：一行用户一行密码
    b.#将用户密码的存放文本转化为数据库类型，并使用hash加密

       [root@localhost vsftpd]# db_load -T -t hash -f vsftp.user vsftp.db
       [root@localhost vsftpd]# chmod 600 vsftp.db

        #修改文件权限为600，保证其安全性
```

### 4.创建FTP虚拟用户的映射用户，并制定其用户家目录

```python
       [root@localhost ~]# useradd -d /var/ftproot -s /sbin/nologin  vritual
        #创建virtual 用户作为ftp的虚拟用户的映射用户
       [root@localhost ~]# mkdir -p /var/ftproot
        #创建目录上传目录
```

### 5.建立支持虚拟用户的PAM认证文件，添加虚拟用户支持

```
    #生成自己的认证配置文件，方便一会调用
        [root@localhost ~]# cd /etc/pam.d/
        [root@localhost pam.d]# cp -a vsftpd vsftpd.pam
    #编辑新生成的文件vsftpd.pam (添加下列两行，剩余的全部注释掉)
        auth       required     pam_userdb.so  db=/etc/vsftpd/vsftp
        account    required     pam_userdb.so  db=/etc/vsftpd/vsftp
```

### 6.在vsftpd.conf文件中添加支持配置
```
    anonymous_enable=NO        #关闭用户登录
    local_enable=YES           #开启本地用户登录
    write_enable=YES           #允许用户写权限
    anon_umask=073             #设置默认创建或上传默认权限
 #修改：
    pam_service_name=vsftpd.pam
 #添加：
    guest_enable=YES     #开启虚拟用户
    guest_username=virtual    #指定映射系统用户
    user_config_dir=/etc/vsftpd/dir    #指定虚拟用户独立的配置文件目录
```

### 7.为个别虚拟用户建立独立的配置文件

```
 #创建指定虚拟用户独立的配置文件目录
        [root@localhost ~]# mkdir /etc/vsftpd/dir
 #进入/etc/vsftpd/dir为每个用户创建独立配置文件
        [root@localhost dir]# touch user1
        [root@localhost dir]# touch user2
        [root@localhost dir]# touch user3
```

### 8.配置虚拟用户规则

```
一个用户可以上传
    [root@localhost dir]#vim user1
    anon_world_readable_only=NO    #允许查看和上传下载文件
    anon_upload_enable=YES        #允许上传文件
一个用户可以创建目录或文件
    [root@localhost dir]#vim user2
    anon_world_readable_only=NO    #允许查看和上传下载文件
    anon_mkdir_write_enable=YES    #允许创建目录
一个用户可以修改文件名
    [root@localhost dir]#vim user3
    anon_world_readable_only=NO    #允许查看和上传下载文件
    anon_other_write_enable=YES    #允许重名和删除文件
```

### 9.启动VSFTP服务，并且客户端测试

```
 [root@localhost ~]# service vsftpd restart       
 [root@localhost ~]# netstat -antp        
 tcp        0      0 0.0.0.0:21                  0.0.0.0:*                   LISTEN      2349/vsftpd
```
