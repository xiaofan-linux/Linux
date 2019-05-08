# Apache简介

```
Apache HTTPD Server 简称 Apache，是 Apache 软件基金会的一个开源的网页服务器， 可以在大多数计算机操作系统中运行，由于其多平台和安全性被广泛使用，是最流行的 Web 服务器端软件之一。
它快速、可靠并且可通过简单的 API 扩展，将 Perl/Python 等解释器编 译到服务器中！Apache HTTP 服务器是一个模块化的服务器,各个功能使用模块化进行插拔！ 目前支持 Windows，Linux，Unix 等平台！
```

```

Apache 软件基金会（也就是 Apache Software Foundation，简称为 ASF），是专门为运 作一个开源软件项目的 Apache 的团体提供支持的非盈利性组织，这个开源软件项目就是 Apache 项目！那么我们的 HTTPD 也只是 Apache 的开源项目之一！
主要的开源项目：HTTP Server，Ant，DB，iBATIS，Jakarta，Logging，Maven，Struts， Tomcat，Tapestry,Hadoop 等等。只是最有名的是 HTTP Server，所以现在所说的 Apache 已 经就是 HTTPD Server 的代号了! 我们还见的比较多的是 Tomcat，Hadoop 等项目
```

```
官方网站：http://www.apache.org/httpd：http://httpd.apache.org/
```

图标：
![](/Apache/image/1.png)

# URL:统一资源定位符

* 协议+主机地址或域名+资源地址
   -如: [http:\/\/www.itxdl.cn\/linux\/](http://www.itxdl.cn/linux/)
  # 安装方法

  * rpm包安装
  * 源码编译安装
# 主配置文件
* Apache配置文件
  * 源码包安装
    ```
      /usr/local/apache2/etc/httpd.conf #主配置文件
      /usr/local/apache2/etc/extra/*.conf #子配置文件
    ```
  * rpm包安装
    ```
      /etc/httpd/conf/httpd.conf
    ```
# 相关文件
* 网页默认保存位置
    * 源码包安装
```
        /usr/local/apache2/htdocs
```
    * rpm包安装
```
        /var/www/html
```
* 日志保存位置
    * 源码包安装
```
        /usr/local/apache2/logs/
```
    * rpm包安装
```
        /var/log/httpd/
```
### 注意日志的轮替


