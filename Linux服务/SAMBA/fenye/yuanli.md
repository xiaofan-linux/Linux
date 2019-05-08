# SAMBA原理

```
SMB是基于客户机/服务器型的Samba协议    
在windows下使用的是NetBIOS协议，
组成Samba运行的有两个服务
        一个是SMB，
        另一个是NMB
     SMB是Samba 的核心启动服务，主要负责建立 Linux Samba服务器与Samba客户机之间的对话， 验证用户身份并提供对文件和打印系统的访问，只有SMB服务启动，才能实现文件的共享，监听139 TCP端口；
     而NMB服务是负责解析用的，类似与DNS实现的功能，NMB可以把Linux系统共享的工作组名称与其IP对应起来，如果NMB服务没有启动，就只能通过IP来访问共享文件，监听137和138 UDP端口。
    Samba服务器可实现如下功能：
        WINS和DNS服务； 
        网络浏览服务； 
        Linux和Windows域之间的认证和授权； 
        UNICODE字符集和域名映射；
        满足CIFS协议的UNIX共享等。
samba是一套使用SMB协议的应用程序，通过支持这个协议，samba允许linux服务器与windows系统之间进行通信
```
```
协议：SMB/CIFS服务：    
    smb         实现资源共享、权限验证        TCP 139 445    
    nmb         实现计算机名解析              UDP 137 138（一般不开启，更节省资源）
```

