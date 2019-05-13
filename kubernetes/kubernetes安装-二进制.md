### kubernetes安装-二进制(单Master)

- 1. 软件环境

  软件|版本|下载地址
  -|-|-
  操作系统|CentOS7.5_x64
  Docker|17.03.2
  kubernetes|1.13.3|https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#v1133
  etcd|3.3.12|https://github.com/coreos/etcd/releases/tag/v3.3.12
  Flannel|0.11.0|https://github.com/coreos/flannel/releases/tag/v0.11.0

  网络|IP网段
  -|-
  Cluster IP|10.244.0.0/16
  Service Cluster IP|10.96.0.0/12
  Service DNS IP| 10.96.0.10
  DNS DN|cluster.local

- 2. 服务器角色

  角色|IP|组件
  -|-|-
  Master|100.100.100.100|kube-apiserver，kube-controller-manager，kube-scheduler
  node1|100.100.100.101|kubelet，kube-proxy，docker，flannel，etcd
  node2|100.100.100.102|kubelet，kube-proxy，docker，flannel，etcd
  node3|100.100.100.103|kubelet，kube-proxy，docker，flannel，etcd



- 3. 关闭防火墙，SElinux,设置ntp服务,关闭swap分区,安装初始软件,这是hosts(所有服务器操作)
    ```shell
    # 设置hosts
    cat >>/etc/hosts<<EOF
    100.100.100.100 master
    100.100.100.101 node1
    100.100.100.102 node2
    100.100.100.103 node3
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
- 4. 设置参数
    ```shell
    cat <<EOF >  /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    EOF
    sysctl --system
    ```
- 5. 加载ipvs相关内核模块, 如果重新开机，需要重新加载
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
 - 6.部署Etcd集群
    * 1. 使用cfssl来生成自签证书，先下载cfssl工具(Master执行)
      ```shell
      wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
      wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
      wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
      chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
      mv cfssl_linux-amd64 /usr/local/bin/cfssl
      mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
      mv cfssl-certinfo_linux-amd64 /usr/bin/cfssl-certinfo
      ```
    * 2. 生成证书(Master执行)
      ```shell
      mkdir ~/ssl && cd ~/ssl
        # cat ca-config.json
        {
          "signing": {
            "default": {
              "expiry": "87600h"
            },
            "profiles": {
              "www": {
                 "expiry": "87600h",
                 "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
              }
            }
          }
        }
      ```
      ```shell
      # cat ca-csr.json
      {
          "CN": "etcd CA",
          "key": {
              "algo": "rsa",
              "size": 2048
          },
          "names": [
              {
                  "C": "CN",
                  "L": "Beijing",
                  "ST": "Beijing"
              }
          ]
      }
      ```
    ```shell
      # cat server-csr.json
      {
          "CN": "etcd",
          "hosts": [
          "100.100.100.101", # 这个三个IP地址没有etcd的服务器IP，根据自己的实际IP地址修改
          "100.100.100.102",
          "100.100.100.103"
          ],
          "key": {
              "algo": "rsa",
              "size": 2048
          },
          "names": [
              {
                  "C": "CN",
                  "L": "BeiJing",
                  "ST": "BeiJing"
              }
          ]
      }
      ```
      ```shell
      cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
      cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare etcd
      # ls *pem
      ca-key.pem  ca.pem  etcd_server-key.pem  etcd_server.pem
      ```

      * 3. 部署Etcd(Node1、Node2、Node3执行)
      > 二进制包下载地址：https://github.com/coreos/etcd/releases/tag/v3.3.12

      > **以下部署步骤在规划的三个etcd节点操作一样，唯一不同的是etcd配置文件中的服务器IP要写当前的：**
      ```shell
      # 解压二进制包：
      $ mkdir /opt/etcd/{bin,cfg,ssl,data} -p
      $ tar zxvf etcd-v3.3.12-linux-amd64.tar.gz
      $ mv etcd-v3.3.12-linux-amd64/{etcd,etcdctl} /opt/etcd/bin/
      ```
      ```shell
      # 将Master生成的生成复制到Node1、Node2、Node3的/opt/etcd/ssl/
      $ scp *pem root@node1:/opt/etcd/ssl/
      $ scp *pem root@node2:/opt/etcd/ssl/
      $ scp *pem root@node3:/opt/etcd/ssl/
      ```
      ```shell
      # 创建etcd配置文件
      $ cat /opt/etcd/cfg/etcd
      #[Member]
      ETCD_NAME="etcd01" # node2为etcd02，node3为etcd03
      ETCD_DATA_DIR="/opt/etcd/data/"
      ETCD_LISTEN_PEER_URLS="https://100.100.100.101:2380" # IP为当前服务器IP
      ETCD_LISTEN_CLIENT_URLS="https://100.100.100.101:2379,https://127.0.0.1:2379" # IP为当前服务器IP

      #[Clustering]
      ETCD_INITIAL_ADVERTISE_PEER_URLS="https://100.100.100.101:2380" # IP为当前服务器IP
      ETCD_ADVERTISE_CLIENT_URLS="https://100.100.100.101:2379" # IP为当前服务器IP
      ETCD_INITIAL_CLUSTER="etcd01=https://100.100.100.101:2380,etcd02=https://100.100.100.102:2380,etcd03=https://100.100.100.103:2380"
      ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
      ETCD_INITIAL_CLUSTER_STATE="new"

      #[Security]
      ETCD_CERT_FILE="/opt/etcd/ssl/etcd_server.pem"
      ETCD_KEY_FILE="/opt/etcd/ssl/etcd_server-key.pem"
      ETCD_TRUSTED_CA_FILE="/opt/etcd/ssl/ca.pem"
      ETCD_CLIENT_CERT_AUTH="true"
      ETCD_PEER_CERT_FILE="/opt/etcd/ssl/etcd_server.pem"
      ETCD_PEER_KEY_FILE="/opt/etcd/ssl/etcd_server-key.pem"
      ETCD_PEER_TRUSTED_CA_FILE="/opt/etcd/ssl/ca.pem"
      ETCD_PEER_CLIENT_CERT_AUTH="true"
      ```
      * ETCD_NAME 节点名称
      * ETCD_DATA_DIR 数据目录
      * ETCD_LISTEN_PEER_URLS 集群通信监听地址
      * ETCD_LISTEN_CLIENT_URLS 客户端访问监听地址
      * ETCD_INITIAL_ADVERTISE_PEER_URLS 集群通告地址
      * ETCD_ADVERTISE_CLIENT_URLS 客户端通告地址
      * ETCD_INITIAL_CLUSTER 集群节点地址
      * ETCD_INITIAL_CLUSTER_TOKEN 集群Token
      * ETCD_INITIAL_CLUSTER_STATE 加入集群的当前状态，new是新集群，existing表示加入已有集群
      * ETCD_CERT_FILE etcd公钥证书
      * ETCD_KEY_FILE etcd私钥证书
      * ETCD_TRUSTED_CA_FILE 指定了客户端的CA证书
      * ETCD_CLIENT_CERT_AUTH 启用客户端证书验证
      * ETCD_PEER_CERT_FILE etcd的Peers通信的公钥证书
      * ETCD_PEER_KEY_FILE etcd的Peers通信的私钥证书
      * ETCD_PEER_TRUSTED_CA_FILE Peer 指定了Peers的CA证书
      * ETCD_PEER_CLIENT_CERT_AUTH 启用对等客户端证书验证
      ```shell
      # systemd管理etcd：
      # cat /usr/lib/systemd/system/etcd.service
      [Unit]
      Description=Etcd Server
      After=network.target
      After=network-online.target
      Wants=network-online.target

      [Service]
      Type=notify
      EnvironmentFile=/opt/etcd/cfg/etcd
      ExecStart=/opt/etcd/bin/etcd \
      --name=${ETCD_NAME} \
      --data-dir=${ETCD_DATA_DIR} \
      --listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
      --listen-client-urls=${ETCD_LISTEN_CLIENT_URLS} \
      --advertise-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
      --initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
      --initial-cluster=${ETCD_INITIAL_CLUSTER} \
      --initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
      --initial-cluster-state=new \
      --cert-file=${ETCD_CERT_FILE} \
      --key-file=${ETCD_KEY_FILE} \
      --peer-cert-file=${ETCD_PEER_CERT_FILE} \
      --peer-key-file=${ETCD_PEER_KEY_FILE} \
      --trusted-ca-file=${ETCD_TRUSTED_CA_FILE} \
      --peer-trusted-ca-file=${ETCD_PEER_TRUSTED_CA_FILE}

      Restart=on-failure
      LimitNOFILE=65536

      [Install]
      WantedBy=multi-user.target
      ```
      ```shell
      # 启动并设置开启启动
      systemctl daemon-reload
      systemctl start etcd
      systemctl status etcd
      systemctl enable etcd
      ```
      ```shell
      # 都部署完成后，检查etcd集群状态
      $ /opt/etcd/bin/etcdctl \
      --ca-file=/opt/etcd/ssl/ca.pem \
      --cert-file=/opt/etcd/ssl/etcd_server.pem \
      --key-file=/opt/etcd/ssl/etcd_server-key.pem \
      --endpoints="https://100.100.100.101:2379,https://100.100.100.102:2379,https://100.100.100.103:2379" \
      cluster-health
      # 输入结果
      member 18218cfabd4e0dea is healthy: got healthy result from https://100.100.100.101:2379
      member 541c1c40994c939b is healthy: got healthy result from https://100.100.100.102:2379
      member a342ea2798d20705 is healthy: got healthy result from https://100.100.100.103:2379
      cluster is healthy
      ```
- 7.安装docker(在Node1、Node2、Node3执行)
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

- 8. 部署网络组件Flannel(在Node1、Node2、Node3执行)
  ```shell
  # 随意找台etcd节点执行
  # 1. Falnnel要用etcd存储自身一个子网信息，所以要保证能成功连接Etcd，写入预定义子网段：
  $ /opt/etcd/bin/etcdctl \
  --ca-file=/opt/etcd/ssl/ca.pem \
  --cert-file=/opt/etcd/ssl/etcd_server.pem \
  --key-file=/opt/etcd/ssl/etcd_server-key.pem \
  --endpoints="https://100.100.100.101:2379,https://100.100.100.102:2379,https://100.100.100.103:2379" \
  set /coreos.com/network/config  '{ "Network": "172.17.0.0/16", "Backend": {"Type": "vxlan"}}'
  ```
    ###### 以下部署步骤在规划的每个node节点都操作
  ```shell
  # 下载二进制包
  $ wget https://github.com/coreos/flannel/releases/download/v0.10.0/flannel-v0.10.0-linux-amd64.tar.gz
  $ tar zxvf flannel-v0.9.1-linux-amd64.tar.gz
  $ mkdir -p /opt/flanneld/{bin,ssl,cfg,run}
  $ mv flanneld mk-docker-opts.sh /opt/flanneld/bin
  ```
  ```shell
  # 配置Flannel配置文件
  $ cat /opt/flanneld/cfg/flanneld
  FLANNEL_ETCD_ENDPOINTS="-etcd-endpoints=https://100.100.100.101:2379,https://100.100.100.102:2379,https://100.100.100.103:2379"
  FLANNEL_ETCD_PREFIX="-etcd-prefix=/coreos.com/network"
  FLANNEL_ARGS="--ca-file=/opt/etcd/ssl/ca.pem --cert-file=/opt/etcd/ssl/etcd_server.pem --key-file=/opt/etcd/ssl/etcd_server-key.pem"
  ```
  * FLANNEL_ETCD_ENDPOINTS etcd集群节点地址
  * FLANNEL_ETCD_PREFIX 网络配置记录路径
  * FLANNEL_ARGS 连接etcd认证公钥和私钥
  ```shell
  # systemd管理Flannel
  $ cat /usr/lib/systemd/system/flanneld.service
  [Unit]
  Description=Flanneld overlay address etcd agent
  After=network-online.target network.target
  Before=docker.service

  [Service]
  Type=notify
  EnvironmentFile=/opt/flanneld/cfg/flanneld
  ExecStart=/opt/flanneld/bin/flanneld --ip-masq $FLANNEL_ETCD_ENDPOINTS $FLANNEL_ARGS -iface=eth0
  # --iface  指定flannel通过哪个网卡走，如果有多块网卡建议指定网卡
  ExecStartPost=/opt/flanneld/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /opt/flanneld/run/subnet.env
  Restart=on-failure

  [Install]
  WantedBy=multi-user.target
  ```
  ```shell
  # 配置Docker启动指定子网段
  $ cat /usr/lib/systemd/system/docker.service

  [Unit]
  Description=Docker Application Container Engine
  Documentation=https://docs.docker.com
  After=network-online.target firewalld.service
  Wants=network-online.target

  [Service]
  Type=notify
  EnvironmentFile=/opt/flanneld/run/subnet.env #修改此次
  ExecStart=/usr/bin/dockerd $DOCKER_NETWORK_OPTIONS #修改此次
  ExecReload=/bin/kill -s HUP $MAINPID
  LimitNOFILE=infinity
  LimitNPROC=infinity
  LimitCORE=infinity
  TimeoutStartSec=0
  Delegate=yes
  KillMode=process
  Restart=on-failure
  StartLimitBurst=3
  StartLimitInterval=60s

  [Install]
  WantedBy=multi-user.target
  ```
  ```shell
  # 启动flanneld
  systemctl daemon-reload
  systemctl start flanneld
  systemctl status flanneld
  systemctl enable flanneld

  # 重启docker
  systemctl daemon-reload
  systemctl restart docker
  systemctl status docker
  systemctl enable docker
  ```
  ```shell
  # 检查是否生效,确保docker0与flannel.1在同一网段
  $ ps -ef |grep docker
    root     20941     1  1 Jun28 ?        09:15:34 /usr/bin/dockerd --bip=172.17.34.1/24 --ip-masq=false --mtu=1450
  $ ip addr
    3607: flannel.1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UNKNOWN
        link/ether 8a:2e:3d:09:dd:82 brd ff:ff:ff:ff:ff:ff
        inet 172.17.34.0/32 scope global flannel.1
           valid_lft forever preferred_lft forever
    3608: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP
        link/ether 02:42:31:8f:d3:02 brd ff:ff:ff:ff:ff:ff
        inet 172.17.34.1/24 brd 172.17.34.255 scope global docker0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:31ff:fe8f:d302/64 scope link
           valid_lft forever preferred_lft forever
   ```
   ```shell
   # 测试不同节点互通，在当前节点访问另一个Node节点docker0 IP
   $  ping 172.17.58.1
      PING 172.17.58.1 (172.17.58.1) 56(84) bytes of data.
      64 bytes from 172.17.58.1: icmp_seq=1 ttl=64 time=0.263 ms
      64 bytes from 172.17.58.1: icmp_seq=2 ttl=64 time=0.204 ms

- 9. 部署Master节点
  ###### 在部署Kubernetes之前一定要确保etcd、flannel、docker是正常工作的，否则先解决问题再继续

  ```shell
  # 生成证书
  $ cat ca-config.json
  {
    "signing": {
      "default": {
        "expiry": "87600h"
      },
      "profiles": {
        "kubernetes": {
           "expiry": "87600h",
           "usages": [
              "signing",
              "key encipherment",
              "server auth",
              "client auth"
          ]
        }
      }
    }
  }

  $ cat ca-csr.json
  {
      "CN": "kubernetes",
      "key": {
          "algo": "rsa",
          "size": 2048
      },
      "names": [
          {
              "C": "CN",
              "L": "Beijing",
              "ST": "Beijing",
              "O": "k8s",
              "OU": "System"
          }
      ]
  }
  ```
  ```shell
  # 创建ca证书
  [root@master ssl]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
  2018/12/27 09:47:08 [INFO] generating a new CA key and certificate from CSR
  2018/12/27 09:47:08 [INFO] generate received request
  2018/12/27 09:47:08 [INFO] received CSR
  2018/12/27 09:47:08 [INFO] generating key: rsa-2048
  2018/12/27 09:47:08 [INFO] encoded CSR
  2018/12/27 09:47:08 [INFO] signed certificate with serial number 156611735285008649323551446985295933852737436614
  [root@master ssl]# ls
  ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem
  ```
  ```shell
  # 制作apiserver证书
  # cat server-csr.json
  {
      "CN": "kubernetes",
      "hosts": [
        "10.0.0.1",
        "127.0.0.1",
        "100.100.100.100", # 此处为Master地址
        "kubernetes",
        "kubernetes.default",
        "kubernetes.default.svc",
        "kubernetes.default.svc.cluster",
        "kubernetes.default.svc.cluster.local"
      ],
      "key": {
          "algo": "rsa",
          "size": 2048
      },
      "names": [
          {
              "C": "CN",
              "L": "BeiJing",
              "ST": "BeiJing",
              "O": "k8s",
              "OU": "System"
          }
      ]
  }
  ```
  ```shell
  # 生成apiserver证书
  [root@master ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
  2018/12/27 09:51:56 [INFO] generate received request
  2018/12/27 09:51:56 [INFO] received CSR
  2018/12/27 09:51:56 [INFO] generating key: rsa-2048
  2018/12/27 09:51:56 [INFO] encoded CSR
  2018/12/27 09:51:56 [INFO] signed certificate with serial number 399376216731194654868387199081648887334508501005
  2018/12/27 09:51:56 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
  websites. For more information see the Baseline Requirements for the Issuance and Management
  of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
  specifically, section 10.2.3 ("Information Requirements").
  [root@master ssl]# ls
  ca-config.json  ca.csr  ca-csr.json  ca-key.pem  ca.pem  server.csr  server-csr.json  kubernetes-key.pem  kubernetes.pem
  ```
  ```shell
  # 制作kube-proxy证书
  $ cat kube-proxy-csr.json
  {
    "CN": "system:kube-proxy",
    "hosts": [],
    "key": {
      "algo": "rsa",
      "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "L": "BeiJing",
        "ST": "BeiJing",
        "O": "k8s",
        "OU": "System"
      }
    ]
  }
  ```
  ```shell
  # 生成kube-proxy证书
  [root@master ssl]# cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
  2018/12/27 09:52:40 [INFO] generate received request
  2018/12/27 09:52:40 [INFO] received CSR
  2018/12/27 09:52:40 [INFO] generating key: rsa-2048
  2018/12/27 09:52:40 [INFO] encoded CSR
  2018/12/27 09:52:40 [INFO] signed certificate with serial number 633932731787505365511506755558794469389165123417
  2018/12/27 09:52:40 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
  websites. For more information see the Baseline Requirements for the Issuance and Management
  of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
  specifically, section 10.2.3 ("Information Requirements").
  [root@master ssl]# ls
  ca-config.json  ca-csr.json  ca.pem          kube-proxy-csr.json  kube-proxy.pem  kubernetes-csr.json  kubernetes.pem
  ca.csr          ca-key.pem   kube-proxy.csr  kube-proxy-key.pem   kubernetes.csr      kubernetes-key.pem
  ```
  ```shell
  # 最终生成以下证书文件
  $ ls *pem
  ca-key.pem  ca.pem  kube-proxy-key.pem  kube-proxy.pem  kubernetes-key.pem  kubernetes.pem
  ```
  ```shell
  # 部署apiserver组件
  #下载二进制包：|https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.13.md#v1133
  # 下载这个包（kubernetes-server-linux-amd64.tar.gz）就够了，包含了所需的所有组件。
  $ mkdir /opt/kubernetes-master/{bin,cfg,ssl} -p
  $ tar zxvf kubernetes-server-linux-amd64.tar.gz
  $ cd kubernetes/server/bin
  $ cp kube-apiserver kube-scheduler kube-controller-manager kubectl /opt/kubernetes-master/bin
  ```
  ```shell
  # 创建token文件，用途后面会讲到
  $ cat /opt/kubernetes-master/cfg/token.csv
  674c457d4dcf2eefe4920d7dbb6b0ddc,kubelet-bootstrap,10001,"system:kubelet-bootstrap"
  ```
  * 第一列：随机字符串，自己可生成
  * 第二列：用户名
  * 第三列：UID
  * 第四列：用户组
  ```shell
  # 创建Master共用文件
  $ cat /opt/kubernetes-master/cfg/config
  KUBE_LOGTOSTDERR="--logtostderr=true"
  KUBE_LOG_LEVEL="--v=0"
  KUBE_ALLOW_PRIV="--allow-privileged=true"
  KUBE_MASTER="--master=http://127.0.0.1:8080"
  ```
  * KUBE_LOGTOSTDERR 启用日志
  * KUBE_LOG_LEVEL 日志等级
  * KUBE_ALLOW_PRIV 允许运行特权容器
  * KUBE_MASTER 连接本地apiserver

  ```shell
  # 创建apiserver配置文件
  $ cat /opt/kubernetes-master/cfg/apiserver
  KUBE_API_ADDRESS="--advertise-address=172.17.196.230 --bind-address=172.17.196.230  --insecure-bind-address=127.0.0.1"
  KUBE_API_PORT="--insecure-port=8080 --secure-port=6443"
  KUBE_ETCD_SERVERS="--etcd-servers=https://172.17.196.227:2379,https://172.17.196.228:2379,https://172.17.196.229:2379"
  KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16 --service-node-port-range=30000-50000"
  KUBE_ADMISSION_CONTROL="--admission-control=NamespaceLifecycle,LimitRanger,SecurityContextDeny,ServiceAccount,ResourceQuota,NodeRestriction --authorization-mode=RBAC,Node"
  KUBE_API_ARGS="--allow-privileged=true --kubelet-https=true --enable-bootstrap-token-auth --token-auth-file=/opt/kubernetes-master/cfg/token.csv --tls-cert-file=/opt/kubernetes-master/ssl/kubernetes/kubernetes.pem --tls-private-key-file=/opt/kubernetes-master/ssl/kubernetes/kubernetes-key.pem --client-ca-file=/opt/kubernetes-master/ssl/kubernetes/ca.pem  --service-account-key-file=/opt/kubernetes-master/ssl/kubernetes/ca-key.pem --etcd-cafile=/opt/kubernetes-master/ssl/etcd/ca.pem --etcd-certfile=/opt/kubernetes-master/ssl/etcd/etcd.pem  --etcd-keyfile=/opt/kubernetes-master/ssl/etcd/etcd-key.pem"
  ```
  * --bind-address 监听地址
  * --advertise-address 集群通告地址
  * --secure-port https安全端口
  * --etcd-servers etcd集群地址
  * --service-cluster-ip-range Service虚拟IP地址段
  * --service-node-port-range Service Node类型默认分配端口范围
  * --enable-admission-plugins 准入控制模块
  * --authorization-mode 认证授权，启用RBAC授权和节点自管理
  * --enable-bootstrap-token-auth 启用TLS bootstrap功能，后面会讲到
  * --kubelet-https kubelet启用https
  * --token-auth-file token文件
  * --allow-privileged 启用授权
  ```shell
  # systemd管理apiserver
  $ cat /usr/lib/systemd/system/kube-apiserver.service  
    [Unit]
    Description=Kubernetes API Service
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes
    After=network.target
    After=etcd.service

    [Service]
    EnvironmentFile=-/opt/kubernetes-master/cfg/config
    EnvironmentFile=-/opt/kubernetes-master/cfg/apiserver
    ExecStart=/opt/kubernetes-master/bin/kube-apiserver \
    	    $KUBE_LOGTOSTDERR \
    	    $KUBE_LOG_LEVEL \
    	    $KUBE_ETCD_SERVERS \
    	    $KUBE_API_ADDRESS \
    	    $KUBE_API_PORT \
    	    $KUBELET_PORT \
    	    $KUBE_ALLOW_PRIV \
    	    $KUBE_SERVICE_ADDRESSES \
    	    $KUBE_ADMISSION_CONTROL \
    	    $KUBE_API_ARGS
    Restart=on-failure
    Type=notify
    LimitNOFILE=65536

    [Install]
    WantedBy=multi-user.target
    ````
  ```shell
  # 启动
  $ systemctl daemon-reload
  $ systemctl enable kube-apiserver
  $ systemctl start kube-apiserver
  $ systemctl status kube-apiserver
  ```
  ```shell
  # 部署scheduler组件
  # 创建schduler配置文件
  $ cat /opt/kubernetes-master/cfg/kube-scheduler
  KUBE_SCHEDULER_ARGS="--leader-elect=true --address=127.0.0.1"
  ```
  * --leader-elect 当该组件启动多个时，自动选举（HA）
  ```shell
  # systemd管理schduler组件
  $ cat /usr/lib/systemd/system/kube-scheduler.service
    [Unit]
    Description=Kubernetes Scheduler Plugin
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes

    [Service]
    EnvironmentFile=-/opt/kubernetes-master/cfg/config
    EnvironmentFile=-/opt/kubernetes-master/cfg/scheduler
    ExecStart=/opt/kubernetes-master/bin/kube-scheduler \
    	    $KUBE_LOGTOSTDERR \
    	    $KUBE_LOG_LEVEL \
    	    $KUBE_MASTER \
    	    $KUBE_SCHEDULER_ARGS
    Restart=on-failure
    LimitNOFILE=65536

    [Install]
    WantedBy=multi-user.target
  ```
  ```shell
  # 启动
  $ systemctl daemon-reload
  $ systemctl enable kube-scheduler
  $ systemctl start kube-scheduler
  $ systemctl enable kube-scheduler
  ```
  ```shell
  # 部署controller-manager组件
  # 创建controller-manager配置文件
  $ cat /opt/kubernetes-master/cfg/kube-controller-manager
    KUBE_CONTROLLER_MANAGER_ARGS="--address=127.0.0.1 --service-cluster-ip-range=10.254.0.0/16 --leader-elect=true --cluster-name=kubernetes --cluster-signing-cert-file=/opt/kubernetes-master/ssl/kubernetes/ca.pem --cluster-signing-key-file=/opt/kubernetes-master/ssl/kubernetes/ca-key.pem  --service-account-private-key-file=/opt/kubernetes-master/ssl/kubernetes/ca-key.pem  --root-ca-file=/opt/kubernetes-master/ssl/kubernetes/ca.pem"
  ```
  * --service-cluster-ip-range Service虚拟IP地址段
  * --leader-elect 当该组件启动多个时，自动选举（HA）
  * --cluster-name 集群名称
  ```shell
  # systemd管理controller-manager组件
  $ cat /usr/lib/systemd/system/kube-controller-manager.service
    Description=Kubernetes Controller Manager
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes

    [Service]
    EnvironmentFile=-/opt/kubernetes-master/cfg/config
    EnvironmentFile=-/opt/kubernetes-master/cfg/controller-manager
    ExecStart=/opt/kubernetes-master/bin/kube-controller-manager \
    	    $KUBE_LOGTOSTDERR \
    	    $KUBE_LOG_LEVEL \
    	    $KUBE_MASTER \
    	    $KUBE_CONTROLLER_MANAGER_ARGS
    Restart=on-failure
    LimitNOFILE=65536

    [Install]
    WantedBy=multi-user.target
  ```
  ```shell
  # 启动
  $ systemctl daemon-reload
  $ systemctl enable kube-controller-manager
  $ systemctl start kube-controller-manager
  $ systemctl status kube-controller-manager
  ```
  ```shell
  # 设置环境变量
  vim /etc/profile
  PATH=/opt/kubernetes-master/bin:$PATH
  source /etc/profile
  ```
  ```shell
  # 所有组件都已经启动成功，通过kubectl工具查看当前集群组件状态
  $ kubectl get cs
  NAME                 STATUS    MESSAGE             ERROR
  scheduler            Healthy   ok                  
  etcd-0               Healthy   {"health":"true"}   
  etcd-2               Healthy   {"health":"true"}   
  etcd-1               Healthy   {"health":"true"}   
  controller-manager   Healthy   ok
