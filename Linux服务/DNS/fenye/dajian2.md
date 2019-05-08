#DNS实验2：主从DNS搭建


### 实验准备

```关闭SELinux和iptables防火墙！
    一台主DNS服务器 IP地址：192.168.21.69 系统：CentOS6.8
    一台从DNS服务器 IP地址：192.168.21.70 系统：CentOS6.8
    一台客户机    IP地址：192.168.21.71
```

### 具体实验步骤

```
 1. 主DNS服务器上搭建主如下       
     a. 安装bind软件        
     b. 主配置文件的修改       
     c. 区域配置文件的修改        
     d. 配置数据文件            
            正向数据文件            
            反向数据文件        
     e. 启动named服务        
 2. 从DNS服务器上搭建如下
     a. 安装bind软件                 
     b. 主配置文件的修改              
     c. 区域配置文件的修改 
     d. 启动named服务
```

#### 关闭SELinux和iptables防火墙

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

=====================================================================================================

## 一、配置主DNS服务器（IP地址：192.168.21.69）

#### 1.安装DNS服务

```
    yum -y install bind
```

#### 2.修改主DNS主配置文件

``` python
    options {
        listen-on port 53 { any; }; #声明服务的端口和对外监听的地址  （监听所有网卡用any）                        
        listen-on-v6 port 53 { ::1; };        
        directory       "/var/named";        
        dump-file       "/var/named/data/cache_dump.db";        
        statistics-file "/var/named/data/named_stats.txt";        
        memstatistics-file "/var/named/data/named_mem_stats.txt";        
        allow-query     { any; }; #允许哪些客户端访问        
        recursion yes;
```

#### 3.修改位置区域文件

``` python
   zone "xiaofan.com" IN { #指明增加的DNS域的名称        
        type master;  #指明DNS为主DNS服务器        
        file "named.localhost.xiaofan"; #设置实现正向域名解析的区域文件名称        
        allow-update { none; }; #设置该DNS不允许动态更新};
    zone "21.70.192.in-addr.arpa" IN {  #指明该区域为反向查找区域，IP地址反写        
        type master;  # 指明DNS为主DNS服务器        
        file "named.loopback.xiaofan"; #设置实现反向域名解析的区域文件名称        
        allow-update { none; };  #设置该DNS不允许动态更新};
```

#### 4.配置数据文件  /var/named/

``` python
    A．先复制生成正向解析文件和反向解析文件    
        cp -a named.localhost named.localhost.xiaofan #复制正向解析文件    
        cp -a named.loopback named.loopback.xiaofan   # 复制反向解析文件
    B．编辑正向解析文件（注意域名结尾的 “.”）
    [root@localhost named]# vim named.localhost.xiaofan    
    $TTL 1D    @       IN SOA  xiaofan.com. rname.invalid. ( #xiaofan.com. 写解析的域名
                                                   0       ; serial
                                                   1D      ; refresh                                        
                                                   1H      ; retry                
                                                   1W      ; expire                                      
                                                   3H )    ; minimum            
            NS      dns.xioafan.com.    
    dns     A       192.168.21.70      
    www     A       192.168.21.70   #表示正向解析的对应关系
    c.编辑反向解析文件（注意域名结尾的 “.”）
    [root@localhost named]# vim named.loopback.xiaofan    
    $TTL 1D    @       IN SOA  xiaofan.com. rname.invalid. (   #xiaofan.com. 写解析的域名)   
                                                   0       ; serial                                      
                                                   1D      ; refresh                                       
                                                   1H      ; retry                                       
                                                   1W      ; expire                                        
                                                   3H )    ; minimum            
            NS      dns.xiaofan.com.    
    70      PTR     dns.xiaofan.com.    
    70      PTR     www.xiaofan.com.  #表示反向解析的对应关系
```

#### 5.重启DNS服务

``` python
service named restart
```

=====================================================================================================

## 二、配置从DNS服务器（IP地址：192.168.21.70）

#### 1.安装DNS服务

```
    yum -y install bind
```

#### 2.修改主DNS主配置文件

``` python
    options { listen-on port 53 { any; }; #声明服务的端口和对外监听的地址 （监听所有网卡用any） 
            listen-on-v6 port 53 { ::1; }; 
            directory        "/var/named"; 
            dump-file        "/var/named/data/cache_dump.db"; 
            statistics-file  "/var/named/data/named_stats.txt"; 
            memstatistics-file "/var/named/data/named_mem_stats.txt"; 
            allow-query     { any; }; #允许哪些客户端访问 
            recursion yes;
```

#### 3.修改位置区域文件

``` python
    zone "xiaofan.com" IN { #指明增加的DNS域的名称 
        type slave; #指明DNS为从DNS服务器 
        masters { 192.168.21.70; }; #指定主DNS地址
        file "slaves/named.localhost.xiaofan"; #设置同步主DNS服务器正向解析文件保存位置
        allow-update { none; }; #设置该DNS不允许动态更新}; 
    zone "21.70.192.in-addr.arpa" IN { #指明该区域为反向查找区域，IP地址反写 
        type slave; # 指明DNS为从DNS服务器 
        masters { 192.168.21.70; }; #指定主DNS地址
        file "named.loopback.xiaofan"; #设置同步主DNS反向域名解析文件保存位置 
        allow-update { none; }; #设置该DNS不允许动态更新};
```

#### 4.重启DNS服务

``` python
service named restart
```

=====================================================================================================

## 三、客户机测试

```
1.将客户机DNS地址修改为从DNS服务器地址（IP地址：192.168.21.70）
2. nslookup测试
```

