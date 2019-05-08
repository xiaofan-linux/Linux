# SAMBA简介

### 一、Samba概述

```
Samba是一套使用SMB(Server Message Block)协议的应用程序, 通过支持这个协议, Samba允许Linux服务器与Windows系统之间进行通信,使跨平台的互访成为可能。Samba采用C/S模式, 其工作机制是让NetBIOS( Windows 网上邻居的通信协议)和SMB两个协议运行于TCP/IP通信协议之上,并且用NetBEUI协议让Windows在“网上邻居”中能浏览Linux服务器。

Samba服务器包括两个后台应用程序: Smbd 和 Nmbd。Smbd 是Samba的核心, 主要负责建立 Linux Samba服务器与Samba客户机之间的对话, 验证用户身份并提供对文件和打印系统的访问; Nmbd主要负责对外发布Linux Samba服务器可以提供的NetBIOS名称和浏览服务,使Windows用户可以在“网上邻居”中浏览Linux Samba服务器中共享的资源。另外Samba还包括一些管理工具, 如 smb-client, smbmount, testparm, Smbpasswd 等程序。

Samba服务器可实现如下功能:WINS和DNS服务; 网络浏览服务; Linux和Windows域之间的认证和授权; UNICODE字符集和域名映射;满足CIFS协议的UNIX共享等。
```

### 二、 Samba的主要应用

```
SAMBA的主要目的就是用来沟通Windows与Unix这两种不同的作业平台。
主要应用：
    共享档案与打印机服务；
    提供身份认证；
    提供Windows网络上的主机名称解析(NetBIOS name)。
```

### 三、Samba的两个进程

```
SAMBA主机使用两个进程来管理两个不同的服务：
    smbd：用来处理文件和打印服务请求。
    nmbd：用来处理NetBIOS名称服务请求和网络浏览功能。
当我们启动了SAMBA之后，主机系统就会启动137,138,139这三个port，并且同时会有TCP/UDP的监听服务
```

### 五、Samba的几个主要命令

```
smbpasswd：用来设置Samba用户的帐号和密码。
smbclient：用来查看别的Linux主机的共享。也可以在自己的Samba主机上使用，用来查看设置是否成功。
smbmount：用来将Samba服务器共享的文档和目录挂载到自己的Linux主机上。
testparm：用来检查smb.conf是否有错误。
```

### 六、四种安全等级

```
①security=share：用户访问Samba服务器不需要提供用户名和口令, 安全性能较低。
②security=user：Samba服务器默认的安全等级, 每一个共享目录只能被一定的用户访问, 并由Samba服务器负责检查账号和密码的正确性。
③security=server：第三方验证服务器（用户名和密码是递交到另外一个Samba服务器或Windows服务器去验证,此时必须指定负责验证的那个服务器名称）
④security=domain：域安全级别,使用主域控制器(PDC)来完成认证。
```

