# Nginx简介

```
CentOS6.8
关闭SELinux和iptables防火墙
```


####1.安装依赖软件
```
yum -y install gcc

Nginx的配置及运行需要 pcre、zlib 等软件包的支持，因此应预先安装这些软件的开发包（devel），以便提供相应的库和头文件，确保
Nginx的安装

yum -y install pcre-devel
yum -y install zlib-devel
yum -y install openssl-devel
```
####2.创建Nginx运行账户
```
注：Nginx服务程序默认以nobody身份运行，建议为其创建专门的用户账号，以便更准确地控制其访问权限。增加灵动性、降低安全风
险。如：创建一个名为Nginx的用户，不建立宿主目录以及禁止登录到shell环境

useradd -r -s /sbin/nologin nginx
```
####3.编译安装Nginx
```
tar -zxvf /iso/nginx-1.0.8.tar.gz
cd nginx-1.0.8/
./configure --prefix=/usr/local/nginx \
--user=nginx --group=nginx \
--with-http_ssl_module \
--with-http_stub_status_module

注：配置前可以参考 ./configuer --help 给出说明
      --prefix 指定 Nginx 的安装目录
      --user 和 --group 指定 Nginx 运行用户和组
      --with-http_stub_status_module 启动 http_stub_status_module模块支持状态统计
      --with-http_ssl_module 支持https的ssl加密

make
make install

echo "export PATH=//usr/local/nginx/sbin:$PATH" >>/etc/profile
source /etc/profile
```
####4.启动Nginx
```
nginx  && /usr/local/nginx/sbin/nginx
netstat -antp
```
