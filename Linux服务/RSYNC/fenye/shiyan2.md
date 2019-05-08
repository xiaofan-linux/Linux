# RSYNC实验2（扩展）

```
准备两台主机
    一台RSYNC服务端：192.168.21.69
    一台RSYNC客户端：192.168.21.70
```

### 1.关闭SELinux和iptables防火墙

### ==============================服务端===============================

### 2.查看RSYNC是否安装(如果没有安装yum安装或者源码包安装)

```
[root@rsync-server ~]# rpm -qa rsync  #服务端
rsync-3.0.6-12.el6.x86_64
[root@rsync-client ~]# rpm -qa rsync  #客户端
rsync-3.0.6-12.el6.x86_64
```

#### 3.添加rsync服务的用户，管理本地目录

```
[root@rsync-server ~]# useradd -s /sbin/nologin -M rsync
```

#### 4.生成rsync配置文件，并编辑

```
[root@rsync-server ~]# touch /etc/rsyncd.conf
[root@rsync-server ~]# vim /etc/rsyncd.conf
uid = rsync  #用户，远端的命令使用rsync用户访问共享目录
gid = rsync  #用户组
max connections = 200  #最大连接数
timeout = 300  #超时时间
pid file = /var/run/rsyncd.pid  #进程对应的进程文件
lock file = /var/run/rsyncd.lock  #锁文件
log file = /var/log/rsyncd.log  #rsync日志文件

[backup] #模块名称
    path = /rsync  #共享目录
    ignore errors  #忽略错误
    read only = no  #是否只读
    hosts allow = 192.168.21.0/24  #允许的访问的IP
    auth_users = rsync_backup  #虚拟用户
    secrets file = /etc/rsync.password  #虚拟用户和密码保存位置
```

#### 5.创建共享目录和创建虚拟用户密码保存位置

```
1.创建共享目录
    [root@rsync-server ~]# mkdir /rsync  #创建共享目录
    [root@rsync-server ~]# chown rsync.rsync /rsync/ # 将共享目录所有者和所属组修改为rsync伪用户
    注意：如果没有创建目录会报错。就会chdir failed

2.创建虚拟用户密码文件
    [root@rsync-server ~]# touch /etc/rsync.password  #创建虚拟用户密码文件
    [root@rsync-server ~]# vim /etc/rsync.password #编辑虚拟用户密码文件
    rsync_back：123456
    [root@rsync-server ~]# chmod 600 /etc/rsync.password  #修改密码权限
```

#### 6.启动rsync服务并检查

```
[root@rsync-server ~]# rsync --daemon
```

# 启动rsync服务

```
[root@rsync-server ~]# ps aux | grep rsync | grep -v grep
[root@rsync-server ~]# netstat -antp | grep rsync

[root@rsync-server ~]# lsof -i :873  #已知端口，查服务
```

#### 7.加入开机自启动

```
[root@rsync-server ~]# echo "/usr/bin/rsync --daemon" >> /etc/rc.local
[root@rsync-server ~]# tail -1 /etc/rc.local
```

### ==============================客户端===============================

#### 测试

```
格式：
rsync -avz  /本地目录/* 用户名@服务器地址：：共享模块名
rsync -avz  /本地目录/*  rsync://用户名@服务器地址/共享模块名

[root@nfs-server ~]# rsync -avz /rsync/xiaofan/ rsycn://rsync_backup@192.168.21.69/backup/xiaofan/
[root@nfs-server ~]# rsync -avz /rsync/ rysnc_backup@192.168.21.69::backup/
```
