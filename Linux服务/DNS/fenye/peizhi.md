### DNS配置文件详解
    DNS相关配置文件
        /etc/named.conf  #DNS主配置文件
        /etc/named.rfc1912.zones  #DNS区域配置文件
        /var/named/named.localhost #正向配置文件
        /var/named/named.loopback # 反向配置文件
    DNS端口：53
#### 1.DNS主配置文件

``` shel
        [root@localhost ~]# vim /etc/named.conf
                listen-on port 53 { any; };    #声明服务的端口和对外监听的地址  （监听所有网卡用any）                
                listen-on-v6 port 53 { ::1; };         #IPV6
                directory       "/var/named";        #用户设置服务的数据目录
                dump-file       "/var/named/data/cache_dump.db"; #默认的缓存备份文件
                statistics-file "/var/named/data/named_stats.txt"; #状态的备份文件
                memstatistics-file "/var/named/data/named_mem_stats.txt"; #内存的备份文件
                allow-query     { any; }; #允许哪些客户端访问（所有都允许用any）
                recursion yes;  #表示这台DNS服务器启用了递归查询的

            #防止DNS劫持机制
            dnssec-enable yes; 
            dnssec-validation yes;

            /* Path to ISC DLV key */
            bindkeys-file "/etc/named.iscdlv.key";

            managed-keys-directory "/var/named/dynamic";
            };
    #日志信息
    logging {
            channel default_debug {
                file "data/named.run"; #日志文件的位置
                severity dynamic;  #设置日志等级   dynamic是个特殊的日志级别，自动匹配相应的日志级别，不需要设置
            };
            };
    #设置根DNS相关配置
    zone "." IN {
            type hint; #用来指定DNS服务器类型  hint代表根DNS
            file "named.ca"; #用来指定根DNS信息在哪里保存
            };

    include "/etc/named.rfc1912.zones"; #用于声明区域文件位置
    include "/etc/named.root.key"; 访问根DNS时认证秘钥
```

### 2.DNS区域配置文件

``` python
[root@localhost ~]# vim /etc/named.rfc1912.zones

        zone "xiaofan.com" IN { #zone后面写的是我们注册的域名
                    type master; #用来指定DNS服务器类型  master代表主DNS  slave代表从DNS"
                    file "named.localhost; #正向解析数据文件名称  （表示IP地址和域名的对应关系保存位置）
                    allow-update { none; }; #是否允许客户端动态升级
         };
        zone "21.168.192.in-addr.arpa" IN { #zone后面写IP地址  网络位反写
                     type master; #用来指定DNS类型  master代表主DNS  slave代表从DNS
                     file "named.loopback"; #反向解析数据文件名称  （表示IP地址和域名的对应关系保存位置）
                     allow-update { none; }; #是否允许客户端动态升级
        };
```

### 3.配置数据文件

``` python
[root@localhost ~]# vim /var/named/named.localhost #正向配置文件
$TTL 1D #生存时间为1
@       IN SOA  xiaofan.com. rname.invalid. (  #SOA:起始授权机构资源记录，xiaofan.com.：解析域名
                                        0       ; serial    #序列号
                                        1D      ; refresh   #更新时间
                                        1H      ; retry     #重试时间
                                        1W      ; expire    #过期时间
                                        3H )    ; minimum   #缓存时间
        NS      dns.xiaofan.com. #指定DNS
dns        A       192.168.21.70    
www        A       192.168.21.70    #指定正向解析的对应关系

```

``` python
[root@localhost samba]# vim /var/named/named.loopback # 反向配置文件
$TTL 1D #生存时间为1
@       IN SOA  @ rname.invalid. ( #SOA:起始授权机构资源记录，xiaofan.com.：解析域名
                                        0       ; serial   #序列号
                                        1D      ; refresh  #更新时间
                                        1H      ; retry    #重试时间
                                        1W      ; expire   #过期时间
                                        3H )    ; minimum  #缓存时间
        NS      dns.xiaofan.com #指定DNS
70        A       dns.xiaofan.com
70        A       www.xiaofan.com


```




