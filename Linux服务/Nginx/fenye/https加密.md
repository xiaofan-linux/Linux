#Nignx重定向

####1.1.1检查Nginx的SSL模块是否安装
<font color=#FF0000 size=3>SSL模块：--with-http_ssl_module</font>
```
[root@web-node1~]# /application/nginx/sbin/nginx -V
nginx version: nginx/1.6.3
built by gcc 4.4.7 20120313 (Red Hat 4.4.7-16) (GCC)
TLS SNI support enabled
configure arguments: --prefix=/application/nginx-1.6.3 --user=nginx --group=nginx ** --with-http_ssl_module ** --with-http_stub_status_module
```
####2.创建私钥
```
注：在实验环境中可以用一下命令生成测试，在生产环境中必须要在https厂商注册
openssl genrsa -des3 -out server.key 1024
    #建立服务器私钥（过程需要输入密码，请记住这个密码）生成RSA密钥
openssl req -new -key server.key -out server.csr
    #需要依次输入国家，地区，组织，email。最重要的是有一个commonname，可以写你的名字或者域名。如果为了https申请，这个必须和域名吻合，否则会引发浏览器警报。生成的csr文件交给CA
签名后形成服务端自己的证书
cp server.key server.key.bak
    #备份私钥
openssl x509 -req -days 365 -sha256 -in server.csr -signkey server.key -out servernew.crt
    #命令生成v1版证书
cp servernew.crt /usr/local/nginx/conf/server.crt
cp server.key /usr/local/nginx/conf/server.key
    #将证书拷贝到nginx配置文件目录
```
####注意：如果你购买的是第三方服务证书,那么只需要参考3-6的配置信息即可完整企业ssl配置实践。
####3.开启Nginx SSL
```
ssl on; #开启ssl验证
ssl_certificate server.crt; #指定证书位置
ssl_certificate_key server.key; #指定私钥位置
ssl_session_timeout 5m;
ssl_protocols TLSv1;
ssl_ciphers HIGH:!RC4:!MD5:!aNULL:!eNULL:!NULL:!DH:!EDH:!EXP:+MEDIUM;
ssl_prefer_server_ciphers on;
```
####4.重启nginx生效
```
注意:配置修改完成好检查语法是否有错误
  /usr/local/nginx/sbin/nginx -t
平滑加载配置文件
  /usr/local/nginx/sbin/nginx -s reload
```
####5.测试https
```
由于该证书非第三方权威机构颁发，而是我们自己签发的，所以浏览器会警告，如果是对外的业务需要加密，必须使用商用第三方签名证书。
必须访问:https://域名
```
####6.配置重定向80端口转443端口
```
以上配置有个不好的地方，如果用户使用时忘了使用https或者443端口，那么将会报错.
所以我们需要80端口的访问转到443端口并使用ssl加密访问。
加入以下代码：
server {
      listen 80;
      server_name www.xdl.com;
      rewrite ^(.*)$ https://$host$1 permanent;
}

注意:配置修改完成好检查语法是否有错误
  /usr/local/nginx/sbin/nginx -t
平滑加载配置文件
  /usr/local/nginx/sbin/nginx -s reload
```
