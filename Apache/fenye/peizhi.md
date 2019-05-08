# Apache配置文件详解

### 基本配置(httpd.conf）
```
ServerRoot "/usr/local/apache2"  #Apache主目录
Listen 80                        #监听端口
LoadModule                       #加载相关模块
User                             #伪用户
Group                            #伪用户组
ServerAdmin                      #管理员邮箱
ServerName                       #服务器名称（生产环境一般写域名）
DirectoryIndex                   #默认网页加载文件
ErrorLog                         #错误日志
LogLevel                         #日志等级
CustomLog                        #正确访问日志
Include                          #加载子配置文件
```
### 主页目录与权限
```
# 网页主目录
DocumentRoot "/usr/local/apache2/htdocs" 
# 网页目录权限 
<Directory "/usr/local/apache2/htdocs">
......
</Directory>

# Options目录权限
    None：没有任何额外权限
    All：所有权限
    Indexes：浏览权限（如果在主目录下没有找到index文件，就会打目录下所有的内容列出来）
    FollowSymLinks:准许进过次目录连接到其他目录
    MultiViews：准许文件名泛匹配（需要手动开启模块才有效negotiation）
# AllowOverride子权限文件开关
    定义是否允许目录下.htaccess文件中的权限生效
    None：.htaccess中权限不生效
    All：文件中所有权限都生效
# Require访问控制列表（依据IP地址）
    Require all granted  #允许所有访问
    Require all denied   #拒绝所有访问
# 仅允许IP为192.168.1.1的主机访问
    <RequireAll> 
          Require all  granted 
          Require ip 192.168.1.1 
    </RequireAll> 
# 仅允许192.168.0.0/24网络的主机访问
    <RequireAll>  
          Require all  granted  
          Require ip 192.168.1.0/24 
    </RequireAll>
# 禁止192.168.1.2的主机访问,其他的都允许访问,
    <RequireAll> 
          Require all  granted 
          Require not ip 192.168.1.2 
    </RequireAll> 
```
# 常用子配置文件(extra)
```
httpd-autoindex.conf              #apache系统别名
httpd-default.conf                #线程控制（建议开启）
httpd-info.conf                   #状态统计网页
httpd-languages.conf              #语言编码（建议开启）
httpd-manual.conf                 #帮助文档
httpd-mpm.conf                    #最大连接数（建议开启）
httpd-multilang-errordoc.conf     #错误页面
httpd-ssl.conf                    #ssl安全套接字访问
httpd-userdir.conf                #用户主目录设置
httpd-vhosts.conf                 #虚拟主机

```

##httpd-default.conf                #线程控制参数（建议开启）
    Timeout 60              #超时时间，多少 s 没有反应就超时
    KeepAlive Off           #开启线程控制（不开启的话用户访问页面会产生一个进程，访问其他页面会产生另一个进程，这样的话一个用户会产生好多个进程，会降低apache性能。开启此项，当用户访问网站时会产生一个进程，打开其他页面时会产生线程，保证了一个用户只产生一个进程。网站此项功能必须开启。）
    MaxKeepAliveRequests 100    #最大线程连接数
##httpd-mpm.conf                    #最大连接数（默认：worker MPM）

