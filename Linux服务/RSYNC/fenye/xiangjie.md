# RSYNC

### 1、命令

```
rsync [选项] 
    -a：归档模式，递归并保留对象属性
    -v：显示同步过程
    -z：在传输文件时进行压缩
    --exclude：排除
    --delete：无差异同步   慎用!
    --bwlimit=Kbps 限速
```

### 2、格式

```
上行同步（上传）
    rsync -avz  /本地目录/* 服务器地址：/服务器目录
下行同步（下载）
    rsync -avz 服务器地址：/服务器目录/* 本地目录    
```

### 构建rsync同步源参数

#### 1.全局参数\(加粗的常用\)

| 参数 | 作用 |
| --- | --- |
| **address=IP地址** | 等等等 |
| **port 端口** | 指定rsync端口 |
| uid=root | 该选项指定当该模块传输文件时守护进程应该具有的uid;默认值是"nobody" |
| gid=root | 该选项指定当该模块传输文件时守护进程应该具有的gid;默认值是”nobody” |
| timeout=300 | 超时参数（单位\/秒） |
| use chroot=no | 不使用chroot |
| max connections=0 | 设置最大连接数，默认为0，即无限制。你可以随意设置，10、20都可以 |
| strict mode=yes | 设置是否检查口令文件的权限 |
| **pid file=\/var\/run\/rsyncd.pid** | 指定rysnc进程的pid文件位置 |
| **log file=\/var\/log\/rsyncd.log** | 指定 rsync日志输出路径 |
| lock file=\/var\/lock\/rsync.lock | 锁文件，防止文件不一致 |

#### 2.分享目录参数\(加粗常用\)

| 参数 | 作用 |
| --- | --- |
| **\[lansggtest\]** | 模块名，可使用任意名称 |
| **path=\/root\/test\/** | 同步源目录路径 |
| **comment=lansgg test** | 模块描述 |
| ignore errors | 出现I\/O错误时可忽略 |
| **read only=no** | 是否只读，设置为no时客户端可上传文件 |
| **write only=no** | 设置为no 时客户端可下载文件 |
| **hosts allow=192.168.182.129** | 允许访问的主机，也可以写一个网段 |
| **hosts deny=\*** | 拒绝访问的主机，\*表示所有主机 |
| list=false | 设置客户端请求时是否列出该模块，false为隐藏 |
| auth users=lansgg | 设置连接时使用的用户，即密码文件里面定义的用户名。如果没有这行，则表明是匿名 |
| secrets file=\/etc\/rsyncd.pass | 指定密码文件位置 |
| **dont compress = **__**.gz **__**.bz2** | 传输进行  压缩 |
| **auth users = user2** | 指定用户 |



