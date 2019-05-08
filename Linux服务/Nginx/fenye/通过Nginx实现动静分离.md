#通过Nginx实现动静分离

####静态分离可以根据以下两种分离：
```
第一：根据后缀名 例如：.jpg|.png|css|js  等待
第二：根据目录，图片全部在存在某个目录下  例如：/image
```
####第一：根据后缀名 例如：.jpg|.png|css|js  等(用正则表达式)
```
upstream dynamic_mywebserver {   
    server 127.0.0.1:9090 down;  #down策略
    server 127.0.0.2:8080 weight=2;  #权重策略
    server 127.0.0.3:6060;           #支持IP+port
    server 127.0.0.4:7070 backup;    #backup策略
    server 54.244.56.3:8081 max_fails=3 fail_timeout=30s;  #重试策略
    server ppzedu.com/article;       #域名方式
    server unix:/tmp/backend3;       #unix套接字方式    
  }
upstream static_mywebserver {   
    server 127.0.0.1:9090 down;  #down策略
    server 127.0.0.2:8080 weight=2;  #权重策略
    server 127.0.0.3:6060;           #支持IP+port
    server 127.0.0.4:7070 backup;    #backup策略
    server 54.244.56.3:8081 max_fails=3 fail_timeout=30s;  #重试策略
    server ppzedu.com/article;       #域名方式
    server unix:/tmp/backend3;       #unix套接字方式    
  }
location ~ .*\.(js|jpg|JPG|jpeg|JPEG|css|bmp|gif|GIF)?$ {
    proxy_pass http://static_mywebserver;
    access_log off;
}
location ~ .*\.(php|php5)?$ {
    proxy_pass http://dynamic_mywebserver;

}

```
####第二：根据目录，图片全部在存在某个目录下  例如：/image
```
location /image/ {
    proxy_pass http://static_mywebserver;
    access_log off;
}
location /php/ {
    proxy_pass http://dynamic_mywebserver;

}
```
