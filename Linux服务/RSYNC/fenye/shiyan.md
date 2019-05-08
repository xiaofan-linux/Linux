# RSYNC实验1：构建ssh单向同步源

#

```
构建双同步源
    实验环境准备：一台服务器，一台客户端
    在服务器和客户端上创建单独的测试目录（/server/ssh、/client/ssh）
    server IP地址：192.168.21.69
    client IP地址：192.168.21.70
```

#### 1、关闭SELinux和iptables防火墙

#### 2、分别在server和client上创建单独的测试目录

```
      [root@server ~]# mkdir -p /server/ssh  #创建目录
      [root@server ~]# mkdir -p /client/ssh  #创建目录     

```

#### 3、首先，要创建用来做上行同步的用户，并给予用户对上行同步文件所在的目录要有权限执行

```
1.Server端创建用户
    [root@server ~]# useradd user  #创建用户
    [root@server ~]# passwd user   #设置用户密码
结合setfacl使用，保证安全性
    [root@server ~]# setfacl -m u:user:rxw /server/ssh #给同步目录设置acl权限

```

#### 4、启动rsync

```
    rsync --daemon
     注意：如果关闭rsync用killall rsync  
          如果关闭用killall -9 rsync，请手动删除/var/run/rsyncd.pid
```

#### 5、在客户端进行上传文件

```
[root@client ssh]# rsync -avz /client/ssh/install.log user@192.168.21.69:/server/ssh  #上传文件
[root@client ssh]# rsync -avz  user@192.168.21.69:/server/ssh/* /client/ssh/  #下载文件

```

=====================================================================================================

# RSYNC实验2：构建rsync单向同步源

```

构建rsync同步源
    实验环境准备：一台服务器，一台客户端
    在服务器和客户端上创建单独的测试目录（/server/rsync、/client/rsync）
    server IP地址：192.168.21.69
    client IP地址：192.168.21.70

```

#### 1、关闭SELinux和iptables防火墙

#### 2、分别在server和client上创建单独的测试目录

```
    [root@server ~]# mkdir -p /server/rsync  #创建目录
    注意：
    创建目录是默认所属者和所属组是创建的用户，其他的只有rx权限，而rsync上传时默认用户nobody，所以没有权限上传
    第一种方法
        在系统创建一个伪用户
        useradd -s /sbin/nologin -M rsync
        在rsync主配置文件定义两个全局变量/etc/rsyncd.conf
        uid = rsync
        gid = rsync
        将目录的所有者和所属组修改为rsync，或者给目录设置rsync的acl权限rwx
    第二种方法
        给目录设置nobody的acl权限rwx
```

#### 3、在Server创建主配置文件（/etc/rsyncd.conf）默认不存在

```python
          1.修改Server配置文件
               [root@server ~]# touch /etc/rsyncd.conf    #创建配置文件
               [root@server ~]# vim /etc/rsyncd.conf      #编辑配置文件
               address = 192.168.21.69  #监听哪个网卡
               port 873 #指定rsync端口
               log file = /var/log/rsyncd.log  #指定 rsync日志输出路径
               pid file = /var/run/rsyncd.pid  #指定rysnc进程的pid文件位置
               [test] #模块名，可使用任意名称
                    comment = this is test  #模块说明
                    path = /server/rsync  #同步源目录路径
                    read only = no  #是否只读
                    dont compress = *.gz *.bz2 #指定哪些文件不需要压缩
                    auth users = user2
                    secrets file = /etc/rsyncd_users.db  #密码文件保存位置
```

#### 4、创建密码文件

```

1.在server创建密码文件
    [root@server ~]# touch /etc/rsyncd_users.db  #创建密码文件
    [root@server ~]# vim /etc/rsyncd_users.db    #编辑密码文件
    user2:123   #密码文件内容
    [root@server ~]# chmod 600 /etc/rsyncd_users.db  #修改密码文件权限

```

#### 5、启动rsync

```

rsync --daemon
 注意：如果关闭rsync用killall rsync
      如果关闭用killall -9 rsync，请手动删除/var/run/rsyncd.pid

```

#### 6.客户端测试

```

[root@client ~]# rsync -avz  /client/rsync/*  rsync://user@192.168.21.69\/test  #测试上传
[root@client ~]# rsync -avz rsync://user@192.168.21.69/test /client/rsync/ #测试下载

```

=====================================================================================================

## ssh源（密钥对）免密码验证

```

注意：单向只在客户端上传密钥对，将公钥上传至服务端
1、在server端生成秘钥
   [user@server ~]$ ssh-keygen -t rsa  #生成秘钥
   [user@server ~]$ ssh-copy-id user@192.168.21.70
2、在client端生成秘钥
   [user@server ~]$ ssh-keygen -t rsa  #生成秘钥
   [user@server ~]$ ssh-copy-id user@192.168.21.69
```

## rsync同步源免密码验证

```
    通过系统变量
    export RSYNC_PASSWORD=123  #123就是设置rsync同步时，设置的密码
    永久生效加入~/bashrc
```

=====================================================================================================

### rsync + inotify(实时同步，单向同步工具)

```
ssh源（密钥对）免密码验证 （前面构建ssh同步源，和基于rsync验证基本一致）  
    监控脚本：
    #!/bin/bash
    a="/usr/local/bin/inotifywait -mrq -e create,delete /server/ssh/"
    b="/usr/bin/rsync -avz /server/ssh/* user@192.168.21.70:/client/ssh"
    $a | while read directory event file
    do
            $b
    done
启动rsync服务
    rsync --daemon
启动脚本
    sh rsync.sh &   #后台执行
```

=====================================================================================================

### unison+ inotify(实时同步，双向同步工具)

```
准备两台主机
    A主机：192.168.21.69
    B主机：192.168.21.70
分别在A主机和B主机安装unison+ inotify
```

#### 1、安装gcc

```
yum -y install gcc
```

#### 2、安装inotify

```
./configure && make && make install
```

#### 3、安装ocaml

```
./configurer
make world opt  
make install
```

#### 4、安装unison

```
make UISTYLE=text THREADS=true STATIC=true
cp unison /usr/local/bin/
```

注意：两台主机都安装

实时同步脚本：(两台服务器都要配置)(注意：脚本要有执行权限)

```
#!/bin/bash      
a="/usr/local/bin/inotifywait -mrq -e create,delete /server/ssh/"     
b="/usr/bin/unison -batch /server/ssh/ ssh://user@192.168.21.70//client/ssh"      
$a | while read directory event file    
do          
      $b      
done
```
