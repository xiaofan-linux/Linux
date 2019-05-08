# FTP实验4：openssl+vsftpd 加密验证方式

#### 一、ftp是明文传输，极不安全，可以使用tcpdump等抓包工具进行抓取

```
1.查询tcpdump工具是否安装
    [root@localhost ~]# rpm -q tcpdump             #查询系统是否安装tcpdump
2.如果没有安装，yum安装tcpdump工具
    [root@localhost ~]# yum -y install tcpdump     #yum安装tcpdump
3.利用tcpdump工具进行抓包破解（在FTP服务器上运行一下命令）
    [root@localhost ~]# tcpdump -i eth0 -nnX port 21
4.在客户端连接登录FTP
```

### 二、ftps是用openssl进行加密传输，相比较明文传输更加安全。

```
1.查看是否安装了openssl
rpm –q openssl
2.如果没有安装。yum安装
    yum -y install openssl
3.查看vsftpd 是否支持openssl
    ldd /usr/sbin/vsftpd  |  grep libssl    
4.生成加密信息
    openssl req –new –x509 –nodes –out vsftpd.pem –keyout vsftpd.pem
        req         #标注格式
        -new        #创建一个新的证书
        -x509       #证书内容格式
        -nodes      #不使用密码
        -out        #生成文件名
        -keyout     #生成的秘钥文件名
    创建过程当中根据提示要一次填写：国家、省份、城市、组织、部门、个人或主机名、邮箱
5.把证书存放到特定目录
    [root@localhost ~]\# cp -a vsftpd.pem /etc/ssl/certs/
    [root@localhost ~]# chmod -R 500 /etc/ssl/certs/
6.修改主配置文件/etc/vsftpd/vsftpd.conf
    ssl_enable=YES                   #启用ssl认证
    ssl_tlsv1=YES
    ssl_sslv2=YES                    #开启tlsv1、sslv2、sslv3都支持
    ssl_sslv3=YES
    allow_anon_ssl=YES               #表示允许匿名用户使用ssl加密
    force_anon_logins_ssl=YES        #强制匿名用户登录时使用ssl加密
    force_anon_data_ssl=YES          #强制匿名用户数据传输使用ssl加密
    force_local_logins_ssl=YES       #强制本地用户登录时使用ssl加密
    force_local_data_ssl=YES         #强制本地用户数据传输使用ssl加密
    rsa_cert_file=/etc/ssl/cert/vsftpd.pem        #证书文件所在目录
7.重启服务
	service vsftpd restart
8.测试(使用第三方客户端连接)
	FileZilla-FTP（第三方客户端工具）
	连接测试时选择：
		服务器类型：通过显式 TLS/SSL 
		登录类型： 一般或匿名
```
