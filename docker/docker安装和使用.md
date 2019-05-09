### docker安装与使用
```python
Docker 的安装和使用有一些前提条件，主要体现在体系架构和内核的支持上。

对于体系架构，除了 Docker 一开始就支持的 X86-64 ，其他体系架构的支持则一直在不断地完善和推进中。

Docker 分为 CE 和 EE 两大版本。CE 即社区版；EE 即企业版，强调安全，付费使用。

我们在安装前可以参看官方文档获取最新的 Docker 支持情况，官方文档在这里：https://docs.docker.com/install/。

Docker 对于内核支持的功能，即内核的配置选项也有一定的要求（比如必须开启 Cgroup 和 Namespace 相关选项，以及其他的网络和存储驱动等）。

Docker 源码中提供了一个检测脚本来检测和指导内核的配置，脚本链接在这里：https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh
```

在满足前提条件后，安装就变得非常的简单了。

Docker CE 的安装请参考官方文档：
  - MacOS：https://docs.docker.com/docker-for-mac/install/
  - Windows：https://docs.docker.com/docker-for-windows/install/
  - Ubuntu：https://docs.docker.com/install/linux/docker-ce/ubuntu/
  - Debian：https://docs.docker.com/install/linux/docker-ce/debian/
  - CentOS：https://docs.docker.com/install/linux/docker-ce/centos/
  - Fedora：https://docs.docker.com/install/linux/docker-ce/fedora/
  - 其他 Linux 发行版：https://docs.docker.com/install/linux/docker-ce/binaries/

##### 这里我们以 CentOS 7 作为演示。

- 环境准备：
阿里云服务器（1 核 2G，1M 带宽）
CentOS 7.4 64 位

  - 1. 由于 Docker-CE 支持 64 位版本的 CentOS 7 ，并且要求内核版本不低于 3.10，首先我们需要卸载掉旧版本的 Docker：

  ```shell
  $ sudo yum remove docker \
              docker-client \
              docker-client-latest \
              docker-common \
              docker-latest \
              docker-latest-logrotate \
              docker-logrotate \
              docker-selinux \
              docker-engine-selinux \
              docker-engine
  ```

  - 2. 我们执行以下安装命令去安装依赖包：
  ```shell
  $ sudo yum install -y yum-utils \
       device-mapper-persistent-data \
       lvm2
  ```

  - 3. 安装 Docker
  ```shell
  yum install -y docker-ce
  ```

  - 4. 启动 Docker-CE：
  ```shell
  $ systemctl start docker
  $ systemctl enable docker
  ```

  
