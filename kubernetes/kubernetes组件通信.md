### kubernetes组件通信

- Kubernetes 多组件之间的通信原理：

  - apiserver 负责 etcd 存储的所有操作，且只有 apiserver 才直接操作 etcd 集群
apiserver 对内（集群中的其他组件）和对外（用户）提供统一的 REST API，其他组件均通过 apiserver 进行通信

  - controller manager、scheduler、kube-proxy 和 kubelet 等均通过 apiserver watch API 监测资源变化情况，并对资源作相应的操作
  - 所有需要更新资源状态的操作均通过 apiserver 的 REST API 进行
  - apiserver 也会直接调用 kubelet API（如 logs, exec, attach 等），默认不校验 kubelet 证书，但可以通过 --kubelet-certificate-authority 开启（而 GKE 通过 SSH 隧道保护它们之间的通信）

###### 比如最典型的创建 Pod 的流程：

  > ![](images/k8s-pod-process.png)

- 用户通过 REST API 创建一个 Pod
- apiserver 将其写入 etcd
- scheduluer 检测到未绑定 Node 的 Pod，开始调度并更新 Pod 的 Node 绑定
- kubelet 检测到有新的 Pod 调度过来，通过 container runtime 运行该 Pod
- kubelet 通过 container runtime 取到 Pod 状态，并更新到 apiserver 中
