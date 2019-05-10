### kubernetes安装(kubeadm安装-单Master节点)


###### 1. 安装方式
  - 1.katacoda的课程：katacoda，可以在网站上帮我们启动一个minikube的环境（学习） ​​katacoda(我没有用过，想用的可以自己了解一下)
  - 2.需要我们自己来搭建的 - Rancher，如果你网速不好的话安装 Rancher 可能需要花费一点时间，不过这是值得的。（测试）(后期会详细讲解搭建)
  - 3.Docker for MAC/Windows（推荐）/minikube/（本地）
  - 4.kubeadm（测试）
  - 5.二进制纯手动搭建（生产）


###### 2. kubernetes安装(kubeadm安装-单Master节点)
  - 1. 环境准备
      ```shell
      100.100.100.101 Centos 7  Master
      100.100.100.102 Centos 7  Node
      100.100.100.103 Centos 7  Node
      ```
  - 2. 关闭防火墙，SElinux,设置ntp服务,关闭swap分区,安装初始软件,这是hosts
      ```shell
      # 设置hosts
      cat >>/etc/hosts<<EOF
      100.100.100.101 master
      100.100.100.102 node1
      100.100.100.103 node2
      EOF
      # 安装系统依赖包和工具包
      yum -y install vim wget net-tools bash-completion gcc gcc-c++ ntp ntpdate yum-utils device-mapper-persistent-data lvm2
      # 关闭防火墙并关闭开机自启动
      systemctl stop firewalld
      systemctl enable firewalld
      # 关闭SElinux并关闭开机自启动
      setenforce 0
      sed -ri '/^[^#]*SELINUX=/s#=.+$#=disabled#' /etc/selinux/config
      # 安装ntp时间服务器,并设置时间服务器
      yum -y install ntp ntpdate
      systemctl enable ntpd
      systemctl start ntpd
      ntpdate -u cn.pool.ntp.org
      hwclock --systohc
      timedatectl set-timezone Asia/Shanghai
      # 关闭swap分区，并关闭开机自动加载
      swapoff -a && sysctl -w vm.swappiness=0
      sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
      ```
  - 3. 设置参数
      ```shell
      cat <<EOF >  /etc/sysctl.d/k8s.conf
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      EOF
      sysctl --system
      ```
  - 4. 加载ipvs相关内核模块, 如果重新开机，需要重新加载
    ```shell
    # 安装 ipvadm模块
    yum install -y ipvsadm
    # 加载ipvs相关内核模块
    cat <<EOF > /etc/sysconfig/modules/ipvs.modules
    modprobe ip_vs
    modprobe ip_vs_rr
    modprobe ip_vs_wrr
    modprobe ip_vs_sh
    modprobe nf_conntrack_ipv4
    EOF
    # 设置权限
    chmod u+x /etc/sysconfig/modules/ipvs.modules
    # 开机启动自动加载
    echo "bash /etc/sysconfig/modules/ipvs.modules" >> /etc/rc.local
    # 查看ipvs模块是否启用
    lsmod | grep ip_vs
    ```
  - 4.安装docker
    ```shell
    # 配置yum源(这里用的是阿里源)
    cd /etc/yum.repos.d/
    wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    # 卸载docker(如果没有安装过，此步骤略过)
    yum remove -y docker-ce docker-ce-selinux container-selinux
    # 安装指定版本的docker
    yum install -y --setopt=obsoletes=0 docker-ce-17.03.1.ce-1.el7.centos docker-ce-selinux-17.03.1.ce-1.el7.centos
    # 启动docker
    systemctl enable docker && systemctl restart docker
    ```

  - 5. 安装 kubeadm, kubelet 和 kubectl(注意：三个软件的版本号要一致，三个节点都要安装)
    ```shell
    # 配置yum源(这里用的是阿里源)
    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
    enabled=1
    gpgcheck=1
    repo_gpgcheck=1
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
    EOF
    # 安装 kubeadm, kubelet 和 kubectl
    yum install -y kubelet-1.12.3 kubeadm-1.12.3 kubectl-1.12.3

    # 配置kubelet的cgroups
    DOCKER_CGROUPS=$(docker info | grep 'Cgroup' | cut -d' ' -f3)
    echo $DOCKER_CGROUPS
    cat >/etc/sysconfig/kubelet<<EOF
    KUBELET_EXTRA_ARGS="--cgroup-driver=$DOCKER_CGROUPS"
    EOF
    # # 启动kubelet
    systemctl daemon-reload
    systemctl enable kubelet && systemctl restart kubelet
    ```
  - 6. 下载镜像
    ```shell
    由于网络原因无法连接kubernetes仓库下载镜像，在阿里云上下载镜像，重新打标签
    阿里云镜像地址：dev.aliyun.com

    # Master节点
    # 下载镜像
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-proxy:v1.12.3 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-apiserver:v1.12.3 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-controller-manager:v1.12.3 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-scheduler:v1.12.3 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/etcd:3.2.24 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/coredns:1.2.2 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/flannel:v0.10.0-amd64 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/pause:3.1
    # 重新打标签
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-proxy:v1.12.3 k8s.gcr.io/kube-proxy:v1.12.3 ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-apiserver:v1.12.3 k8s.gcr.io/kube-apiserver:v1.12.3;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-controller-manager:v1.12.3 k8s.gcr.io/kube-controller-manager:v1.12.3;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-scheduler:v1.12.3 k8s.gcr.io/kube-scheduler:v1.12.3;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24 ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/coredns:1.2.2 k8s.gcr.io/coredns:1.2.2 ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64   ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/pause:3.1 k8s.gcr.io/pause:3.1
    # Node节点
    # 下载镜像
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-proxy:v1.12.3 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/coredns:1.2.2 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/flannel:v0.10.0-amd64 ;\
    docker pull registry.cn-hangzhou.aliyuncs.com/kuberimages/pause:3.1
    # 重新打标签
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/kube-proxy:v1.12.3 k8s.gcr.io/kube-proxy:v1.12.3 ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/etcd:3.2.24 k8s.gcr.io/etcd:3.2.24 ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/coredns:1.2.2 k8s.gcr.io/coredns:1.2.2 ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/flannel:v0.10.0-amd64 quay.io/coreos/flannel:v0.10.0-amd64   ;\
    docker tag registry.cn-hangzhou.aliyuncs.com/kuberimages/pause:3.1 k8s.gcr.io/pause:3.1
    ```
  - 7. 安装Master节点
    ```shell
    kubeadm init --kubernetes-version=v1.12.3  --apiserver-advertise-address=100.100.100.101 --pod-network-cidr=10.244.0.0/16  --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap

    # --kubernetes-version 指定安装版本
    # --apiserver-advertise-address 指定Master节点IP
    # --pod-network-cidr=10.244.0.0/16 指定pod网络的IP网段(可以修改其他的网段。但是后面安装flanneld需要和设置的网段一致)
    # --service-cidr 指定service网络
    # --ignore-preflight-errors 忽略swap分区错误

    # 安装完毕后，设置访问kubernetes连接证书
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    # 查看Master是否部署成功(如果DNS不是running，属于正常，部署完网络插件就正常了)
    kubectl get pods -n kube-system
    ```
- 8. 配置使用网络插件
   ```shell
   # 下载flannel网络yaml文件
   wget https://raw.githubusercontent.com/coreos/flannel/v0.10.0/Documentation/kube-flannel.yml
   # 修改配置
   # 此处的ip配置要与上面kubeadm的pod-network一致
    net-conf.json: |
      {
        "Network": "10.244.0.0/16",
        "Backend": {
          "Type": "vxlan"
        }
      }
    #  启动
    kubectl apply -f kube-flannel.yml
    # 查看flanneld部署是否成功
    kubectl get pods --namespace kube-system
    kubectl get svc --namespace kube-system
    ```
- 8. 安装Node节点
    ```shell
    # 此命令为初始化master成功后返回的结果,在node节点执行，会自动安装node节点
    kubeadm join 100.100.100.101:6443 --token r1sp2u.a3sp677pe8md3gvz --discovery-token-ca-cert-hash sha256:9734c35d542a5a4e529963c8826772b3fde2c60c39bcdf4b2bc2394dbe201855

    # 查看node是否部署成功
    kubectl get nodes

    # 忘记初始master节点时的node节点加入集群命令怎么办
    # 简单方法
    kubeadm token create --print-join-command

    # 第二种方法
    token=$(kubeadm token generate)
    kubeadm token create $token --print-join-command --ttl=0
    ```
