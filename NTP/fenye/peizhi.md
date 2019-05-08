# NTP安装

```
NTP端口：123
NNTP端口：119
```

```
服务端安装：
1、安装NTP服务端软件软件包
[root@xue63 Packages]# rpm -ivh /mnt/Packages/ntp-4.2.4p8-2.el6.x86_64.rpm
或者
yum -y install ntp
```

```
2、安装客户端：
[root@xue64 Package]# rpm -ivh /mnt/Packages/ntpdate-4.2.4p8-2.el6.x86_64.rpm
或者
yum -y install ntpdate
```

```

3、启动NTP服务
先查看123端口是否开放：
[root@xue63 ~]# service ntpd start #启动NTP服务
[root@xue63 ~]# chkconfig ntpd on  #加入开启自启
[root@xue63 ~]# netstat -anutp | grep 123  #查看一样端口
udp        0      0 192.168.1.63:123            0.0.0.0:*         21055/ntpd          
udp        0      0 127.0.0.1:123               0.0.0.0:*         21055/ntpd   
```

