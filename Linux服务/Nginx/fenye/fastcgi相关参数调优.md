#fastcgi相关参数调（配合PHP引擎动态服务）

#####Nginx Fastcgi（PHP优化）常见参数列表说明（写在http{}里面）
|PHP优化参数|说明|
|--|--|
|fastcgi_connect_timeout|表示nginx服务器和后端FastCGI服务器连接的超时时间，默认值为60s，这个参数通常不要超过75s，因为建立的连接越多消耗的资源就越多|
|fastcgi_send_timeout|设置nginx允许FastCGI服务返回数据的超时时间，即在规定时间之内后端服务器必须传完所有的数据，否则，nginx将断开这个连接，默认值为60s|
|fastcgi_read_timeout|设置Nginx从FastCGI服务端读取响应信息的超时时间。表示建立连接成功后，nginx等待后端服务器的响应时间，是nginx已经进入后端的排队之中等候处理的时间|
|fastcgi_buffer_size|	这是nginx fastcgi的缓冲区大小参数，设定用来读取FastCGI服务端收到的第一部分响应信息的缓冲区大小，这里的第一部分通常会包含一个小的响应头部，默认情况，这个参数大小是由fastcgi_buffers指定的一个缓冲区的大小|
|fastcgi_buffers|设定用来读取从FastCGI服务端收到的响应信息的缓冲区大小以及缓冲区数量。默认值fastcgi_buffers 8 4或8k；指定本地需要用多少和多大的缓冲区来缓冲FastCGI的应答请求。如果一个PHP脚本所产生的页面大小为256lb，那么会为其分配4个64kb的缓存区用来缓存。如果站点大部分脚本所产生的页面大小为256kb，那么可以把这个值设置为“16 16k”、“464k”等|
|fastcgi_busy_buffers_size|用于设置系统很忙时可以使用fastcgi_buffers大小，官方推荐的大小为fastcgi_buffers*2，默认fastcgi_busy_buffers_size 8k或16k|
|fastcgi_temp_file_write_size|fastcgi临时文件的大小，可设置128-256k|

#####Nginx Fastcgi（PHP缓存）常见参数列表说明（可以配置在server标签和http标签）
|PHP缓存参数|说明|
|--|--|
|
