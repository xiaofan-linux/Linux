### docker常用命令
- 我们可以通过 docker -h 去查看命令的详细的帮助文档。在这里我只会讲一些日常我们可能会用的比较多的一些命令。

![](images/docker-h.png)

- 例如，我们需要拉取一个 Docker 镜像，我们可以用如下命令：
  ```shell
  docker pull image_name
  ```
- image_name 为镜像的名称，而如果我们想从 Docker Hub 上去下载某个镜像，我们可以使用以下命令：
  ```shell
  docker pull centos:latest
  - cento：lastest 是镜像的名称，Docker Daemon 发现本地没有我们需要的镜像，
    会自动去 Docker Hub 上去下载镜像，下载完成后，该镜像被默认保存到 /var/lib/docker 目录下。
  ```

- 接着我们如果想查看主机下存在多少镜像，我们可以用如下命令：
  ```shell
  docker images
  ```
- 我们要想知道当前有哪些容器在运行，我们可以用如下命令：
  ```shell
  docker ps -a
    -a 是查看当前所有的容器，包括未运行的。我们该如何去对一个容器进行启动，重启和停止呢？
  ```
- 我们可以用如下命令：
    ```shell
    docker start container_name/container_id
    docker restart container_name/container_id
    docker stop container_name/container_id
    ```
- 这个时候我们如果想进入到这个容器中，我们可以使用 attach 命令：
  ```shell
  docker attach container_name/container_id
  ```
- 那如果我们想运行这个容器中的镜像的话，并且调用镜像里面的 bash ，我们可以使用如下命令：
  ```shell
  docker run -t -i container_name/container_id /bin/bash
  ```
- 那如果这个时候，我们想删除指定镜像的话，由于 Image 被某个 Container 引用（拿来运行），如果不将这个引用的 Container 销毁（删除），那 Image 肯定是不能被删除。

  - 1. 我们首先得先去停止这个容器：
  ```shell
  docker ps
  docker stop container_name/container_id
  ```
  - 2. 然后我们用如下命令去删除这个容器：
  ```shell
  docker rm container_name/container_id
  ```
  - 3. 然后这个时候我们再去删除这个镜像：
    ```shell
    docker rmi image_name
    ```
