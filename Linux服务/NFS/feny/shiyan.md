#NFS配置实例1：

##### 实例1、共享/tmp/gongxiang 目录给10.0.0.0/24这个网段。

### 3 1.关闭SELinux和iptables防火墙

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

#### 2.安装nfs-utils rpcbind

```
1.查询系统是否安装NFS服务
    rpm -aq nfs-utils rpcbind
2.用yum安装
    yum -y install nfs-utils rpcbind
3.启动rpcbind服务(为了使NFS服务器能正常工作，需要启动rpcbind和nfs两个服务，并且rpcbind一定要先于nfs启动。)
    service rpcbind restart
4.启动NFS服务
    service nfs start
5.设置NFS服务器的自动启动状态
    设置rpcbind和nfs服务在系统运行级别2345自动启动。
    chkconfig --level 2345 rpcbind on
    chkconfig --level 2345 nfs on
6.查看RPC服务器开启了哪些端口
    rpcinfo –p localhost
```

### 服务器端操作：
```
    服务端：IP地址：192.168.21.69
    客户端：IP地址：192.168.21.70
```

```
1.修改配置文件(/etc/exports)
    目标：将NFS服务器的/home/zhangsan共享给192.168.21.0/24网段，rw权限、sync
    [root@localhost ~]# vim /etc/exports
    /tmp/gongxiang 192.168.21.0/24(rx,sync)
2.创建共享目录
    [root@localhost ~]# mkdir /tmp/gongxiang
    drwxr-xr-x. 2 root root 4096 10月  4 17:46 gongxiang #注意现在共享的目录的权限为只有root才有写权限
3.重启NFS服务
    [root@localhost ~]# service nfs reload  #roload检测配置文件是否语法错误，并不需要重启服务就能将新的配置文件加载进服务
4.服务器端查看nfs共享状态
    showmount –e 192.168.21.69  #查看自己共享的服务或者用exportfs
```
###客户端操作：
```
    1.查询客户端是否安装rpcbind服务   
         rpm -aq  rpcbind
    2.用yum安装   
         yum -y install rpcbind
    3.启动rpcbind服务
        service rpcbind restart
    4.客户端查看nfs共享状态
        [root@nfs-server ~]# showmount -e 192.168.21.69
        Export list for 192.168.21.69:
        /tmp/gongxiang 192.168.21.0/24
    5.客户端挂载nfs服务器共享目录
    命令格式：mount -t 类型 NFS服务器IP:共享目录 本地挂载点目录
            #mount –o vers=3 	#指定挂载使用nfs V3版本（避免同步延迟）
        [root@nfs-server ~]# mount -t nfs 192.168.21.69:/tmp/gongxiang /mnt
        [root@nfs-server ~]# mount | grep mnt
        192.168.21.69:/tmp/gongxiang on /mnt type nfs (rw,vers=4,addr=192.168.21.69,clientaddr=192.168.21.70)
        [root@nfs-server ~]# df -h
        192.168.21.69:/tmp/gongxiang  18G  2.2G   15G  13% /mnt
    6.在客户端测试创建文件
        [root@nfs-server mnt]# touch lanp.sh
        touch: 无法创建"lanp.sh": 权限不够  #发现没有现在是没有权限的，因为我们服务器的本地的rwx权限没有开启来。

    7.在服务端创建文件，测试客户是否同步到服务端
    8.加入开机自动挂载(/etc/fstab)
        [root@nfs-server ~]# vim /etc/fstab
        192.168.21.69:/tmp/gongxiang    /mnt    nfs    defaults     0 0
        [root@nfs-server ~]# mount -a  #检测是否有语法错误

权限说明：
    当我们在/etc/exports中给了NFS  rw权限。
    但为什么客户端还是写不了。因为如果我们仅在NFS配置文件中配置了rw权限。
    只表明了网络端的主机能够有权限去服务器端去写文件，但还需要通过服务器端的本地目录的权限。
    那么也就是说客户端需要通过两层的权限来控制的。
    NFS配置文件—>共享目录文件的权限。并且客户端往服务端去写文件的用户身份是nfsnobody、nfsnobody UID=65534。
    建议将共享目录的所有者和所属组修改为nfsnobody



```
