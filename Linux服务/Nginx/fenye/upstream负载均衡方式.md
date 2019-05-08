#.upstream负载均衡方式


```
Nginx的upstream目前调度算法:
1)轮询（默认）
每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。
2)weight
指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。
3)ip_hash
每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，一定程度上可以解决session的问题(排除同一个公网地址多内网的情况下)。
4)fair（第三方）
按后端服务器的响应时间来分配请求，响应时间短的优先分配。
5)url_hash（第三方）
按访问URL的hash结果来分配请求，使每个URL定向到同一个后端服务器，后端服务器为缓存时比较适用。另外，在upstream中加入hash语句后，server语句不能写入weight等其他参数。
6)least_conn
最少连接数，哪个机器连接数少就分发给谁

nginx负载均衡的配置
    在http节点里添加:
    #定义负载均衡设备的 Ip及设备状态
    upstream mywebserver {   
        server 127.0.0.1:9090 down;  #down策略
        server 127.0.0.2:8080 weight=2;  #权重策略
        server 127.0.0.3:6060;           #支持IP+port
        server 127.0.0.4:7070 backup;    #backup策略
        server 54.244.56.3:8081 max_fails=3 fail_timeout=30s;  #重试策略
        server ppzedu.com/article;       #域名方式
        server unix:/tmp/backend3;       #unix套接字方式    
      }
      location / {
              #在需要使用负载的Server节点下添加
              proxy_pass http://mywebserver;
              #具体参数设置，具体的大小根据业务需求和官方推荐来设置。
              proxy_set_header Host $host;
              proxy_set_header X-Forwarded-For $remote_addr;
              client_body_buffer_size 90;
              proxy_connect_timeout 90;
              proxy_send_tiemout 90;
              proxy_read_tiemout 90;
              proxy_buffers 4 32k;
              proxy_buffers_size 4k;
              proxy_busy_buffer_size 64k;
              proxy_temp_file_write_size 64;
      }


upstream 每个设备的状态解释:
      1.down #表示当前的server暂时不参与负载，这个参数配合ip_hash使用
      2.weight #默认为1.weight越大，权重大接受的请求越多。
      3.max_fails=number：#允许请求失败的次数默认为1.当超过最大次数时，从池中踢出，0表示禁止失败尝试，一般两三次为佳。
      4.fail_timeout=mumber #失败超时时间，默认是10s，常规业务2-3秒合理
      5.backup：#热备配置。当其他机器宕机，热备机器会接手过来，等其他机器好了，在交还回去。
      6.max-conns=number #最大并发连接数，默认是0，不限制。
      7.slow-start=time  #节点从坏到好加进来的时间，默认不启用
proxy_pass代理参数：
      	1.proxy_set_header：设置有后端的服务器获取用户的主机名或者真实IP地址，以及代理者的真实IP地址
            proxy_set_header Host $host;
                当后端web服务器上也配置有多个虚拟主机时，需要用该header来区分反向代理哪个主机名
            proxy_set_header X-Forwarded-For $remote_addr;
                如果后端web服务器上的程序获取用户IP地址，从该header头获取，如果后端web服务器是nginx，默认支持。如果apache，需要在配置文件配置日志格式，加入(\"%{X-Forwarded-For}i"\),而且日志格式调整为combined
        2.client_body_buffer_size：用于指定客户端请求主体缓冲区大小，可以理解为先保证在本地在在传给客户
        3.proxy_connect_timeout:表示与后端服务器连接的超时时间，即发起握手等候响应的超时时间
        4.proxy_send_tiemout:表示后端服务器的数据回传时间，即在规定的时间之内后端服务器必须传完所有的数据，否则。Nginx将断开这个连接
        5.proxy_read_tiemout:设置Nginx从代理的后端服务器获取信息的时间，表示连接建立成功后，Nginx等待后端服务器的响应时间，其实是Nginx已经进入后端的排队之中的等待处理时间
        6.proxy_buffers：这是缓冲区的数量和大小，Nginx从代理的后端服务器获取的响应信息，会放置到缓冲区。
        7.proxy_busy_buffer_size:用于设置系统很忙时可以使用的proxy_buffers大小，官方推荐的大小是proxy_buffers * 2
        8.proxy_temp_file_write_size:指定proxy缓存临时文件的大小。
```
