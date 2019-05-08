#Nginx压缩实现性能优化


####1.Nginx gzip压缩功能介绍
```
Nginx gzip压缩模块提供了压缩文件内容的功能，用户请求的内容在发送出用客户端之前，
Nginx服务器会根据一些具体的策略实施压缩，以节约网站出口带宽，同时加快了数据传输效率，提升了用户访问体验。
```
####2.Nginx gzip 压缩的优点
```
1.提升网站用户体验：由于发给用户的内容小了，所以用户访问单位大小的页面就快了，用户体验提升了，网站口碑就好了。
2.节约网站带宽成本，由于数据是压缩传输的，因此，此举节省了网站的带宽流量成本，不过压缩会稍微消耗一些CPU资源，这个一般可以忽略。此功能既能让用户体验增强，公司也少花钱。对于几乎所有的Web服务来说，这是一个非常重要的功能，Apache服务也由此功能。
```
####3.需要和不需要压缩的对象
```
1、纯文本内容压缩比很高，因此纯文本内容是最好压缩，例如：html、js、css、xml、shtml等格式的文件
2、被压缩的纯文本文件必须要大于1KB，由于压缩算法的特殊原因，极小的文件压缩可能反而变大。
3、图片、视频（流媒体）等文件尽量不要压缩，因为这些文件大多数都是经历压缩的，如果再压缩很坑不会减小或减少很少，或者可能增加。而在压缩时还会消耗大量的CPU、内存资源
```
####4、参数介绍及配置说明
```
此压缩功能很类似早起的Apache服务的mod_defalate压缩功能，Nginx的gzip压缩功能依赖于ngx_http_gzip_module模块，默认已安装。
```

###参数说明如下：
```
gzip on; #开启gzip压缩功能
gzip_min_length 1k;
  #设置允许压缩的页面最小字节数，页面字节数从header头的Content-Length中获取，默认值是0，表示不管页面多大都进行压缩，建议设置成大于1K，如果小于1K可能会越压越大
gzip_buffers 4 16k;
  #压缩缓冲区大小，表示申请4个单位为16K的内存作为压缩结果流缓存，默认是申请与原始是数据大小相同的内存空间来存储gzip压缩结果；
gzip_http_version 1.1
  #压缩版本（默认1.1 前端为squid2.5时使用1.0）用于设置识别HTTP协议版本，默认是1.1，目前大部分浏览器已经支持GZIP压缩，使用默认即可。
gzip_comp_level 2;
  #压缩比率，用来指定GZIP压缩比，1压缩比最小，处理速度最快；9压缩比最大，传输速度快，但处理最慢，也消耗CPU资源
gzip_types  text/css text/xml application/javascript;
  #用来指定压缩的类型，“text/html”类型总是会被压缩，这个就是HTTP原理部分讲的媒体类型。
gzip_vary on;
  #vary hear支持，该选项可以让前端的缓存服务器缓存经过GZIP压缩的页面，例如用缓存经过Nginx压缩的数据。

配置在http标签端:
http{
  gzip on;
  gzip_min_length  1k;
  gzip_buffers     4 32k;
  gzip_http_version 1.1;
  gzip_comp_level 9;
  gzip_types  text/css text/xml application/javascript;
  gzip_vary on;
}

```
