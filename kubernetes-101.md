# Kubernetes 101：基础知识与操作手册

> 本指南基于 Kubernetes 官方文档和最佳实践，涵盖从入门到实践的核心知识点

## 目录

1. [Kubernetes 概述](#kubernetes-概述)
2. [核心概念](#核心概念)
3. [集群架构](#集群架构)
4. [安装与配置](#安装与配置)
5. [常用命令速查](#常用命令速查)
6. [工作负载管理](#工作负载管理)
7. [服务与网络](#服务与网络)
8. [存储管理](#存储管理)
9. [配置管理](#配置管理)
10. [最佳实践](#最佳实践)

---

## Kubernetes 概述

### 什么是 Kubernetes？

Kubernetes（简称 K8s）是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，方便进行声明式配置和自动化。Kubernetes 建立在 Google 大规模运行生产工作负载十几年经验的基础上，结合了社区中最优秀的想法和实践。

K8s 这个缩写是因为 K 和 s 之间有 8 个字符。

### 为什么需要 Kubernetes？

容器是打包和运行应用程序的好方式。在生产环境中，你需要管理运行着应用程序的容器，并确保服务不会下线。Kubernetes 为你提供了一个可弹性运行分布式系统的框架。

**Kubernetes 为你提供：**

| 功能 | 说明 |
|------|------|
| **服务发现和负载均衡** | Kubernetes 可以使用 DNS 名称或自己的 IP 地址来暴露容器，自动负载均衡网络流量 |
| **存储编排** | 自动挂载存储系统，如本地存储、公共云存储等 |
| **自动部署和回滚** | 描述已部署容器的所需状态，以受控的速率将实际状态更改为期望状态 |
| **自动完成装箱计算** | 根据容器需要的 CPU 和内存，将容器调度到合适的节点上 |
| **自我修复** | 重新启动失败的容器、替换容器、杀死不响应用户定义的健康检查的容器 |
| **密钥与配置管理** | 存储和管理敏感信息，如密码、OAuth 令牌和 SSH 密钥 |
| **水平扩缩** | 使用简单的命令或根据 CPU 使用率自动对应用进行扩缩 |

### Kubernetes 不是什么

- **不是传统的 PaaS 系统**：Kubernetes 在容器级别运行，而非硬件级别
- **不限制支持的应用程序类型**：支持无状态、有状态和数据处理工作负载
- **不部署源代码，也不构建应用程序**：CI/CD 工作流取决于组织的技术要求
- **不提供应用程序级别的服务**：如中间件、数据库、缓存等（但可以在 K8s 上运行）
- **不是日志记录、监视或警报的解决方案**：但提供了收集和导出指标的机制

---

## 核心概念

### Pod

**Pod 是 Kubernetes 中最小的调度和部署单位**。一个 Pod 可以包含一个或多个容器，这些容器共享网络和存储。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### Deployment

Deployment 用于管理无状态应用，支持滚动更新和回滚。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Service

Service 定义了一组 Pod 的访问策略，提供稳定的 IP 地址和 DNS 名称。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

### StatefulSet

StatefulSet 用于管理有状态应用，每个 Pod 有唯一的标识和稳定的存储。

### DaemonSet

DaemonSet 确保每个（或某些）节点上运行一个 Pod 的副本，常用于日志收集、监控等。

### Job 和 CronJob

- **Job**：运行一次性任务，直到成功完成指定次数
- **CronJob**：基于时间表定期运行 Job

### ConfigMap 和 Secret

- **ConfigMap**：存储非敏感配置数据
- **Secret**：存储敏感信息（密码、令牌等），以 base64 编码存储

---

## 集群架构

### 控制平面组件（Control Plane）

| 组件 | 功能 |
|------|------|
| **kube-apiserver** | 公开 Kubernetes HTTP API 的核心组件服务器 |
| **etcd** | 具备一致性和高可用性的键值存储，用于所有 API 服务器的数据存储 |
| **kube-scheduler** | 查找尚未绑定到节点的 Pod，并将每个 Pod 分配给合适的节点 |
| **kube-controller-manager** | 运行控制器来实现 Kubernetes API 行为 |
| **cloud-controller-manager** | （可选）与底层云驱动集成 |

### Node 组件

| 组件 | 功能 |
|------|------|
| **kubelet** | 确保 Pod 及其容器正常运行 |
| **kube-proxy** | （可选）维护节点上的网络规则以实现 Service 的功能 |
| **容器运行时** | 负责运行容器的软件（如 containerd、CRI-O） |

### 插件（Addons）

- **DNS**：集群范围内的 DNS 解析
- **Web 界面（Dashboard）**：通过 Web 界面进行集群管理
- **容器资源监控**：收集和存储容器指标
- **集群级别日志**：将容器日志保存到中央日志存储

---

## 安装与配置

### 本地开发环境

#### 使用 kind（Kubernetes in Docker）

```bash
# 安装 kind
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# 创建集群
kind create cluster --name my-cluster

# 查看集群
kubectl cluster-info

# 删除集群
kind delete cluster --name my-cluster
```

#### 使用 Minikube

```bash
# 安装 Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# 启动集群
minikube start

# 查看状态
minikube status

# 停止集群
minikube stop
```

### 安装 kubectl

```bash
# macOS (Homebrew)
brew install kubectl

# Linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Windows (Chocolatey)
choco install kubernetes-cli

# 验证安装
kubectl version --client
```

### 配置集群访问

```bash
# 查看当前配置
kubectl config view

# 查看上下文列表
kubectl config get-contexts

# 切换上下文
kubectl config use-context my-cluster-name

# 设置命名空间
kubectl config set-context --current --namespace=my-namespace
```

---

## 常用命令速查

### 集群信息

```bash
# 查看集群信息
kubectl cluster-info

# 查看节点状态
kubectl get nodes
kubectl get nodes -o wide

# 查看节点详情
kubectl describe node <node-name>

# 查看节点资源使用
kubectl top node
kubectl top node <node-name>
```

### 命名空间管理

```bash
# 查看所有命名空间
kubectl get namespaces
kubectl get ns

# 创建命名空间
kubectl create namespace my-namespace

# 删除命名空间
kubectl delete namespace my-namespace

# 切换命名空间
kubectl config set-context --current --namespace=my-namespace
```

### Pod 管理

```bash
# 查看所有 Pod
kubectl get pods
kubectl get po

# 查看所有命名空间的 Pod
kubectl get pods --all-namespaces
kubectl get pods -A

# 查看 Pod 详情
kubectl describe pod <pod-name>

# 查看 Pod 的 YAML
kubectl get pod <pod-name> -o yaml

# 查看 Pod 日志
kubectl logs <pod-name>
kubectl logs -f <pod-name>          # 实时跟踪
kubectl logs <pod-name> -c <container-name>  # 多容器场景

# 进入 Pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- /bin/sh

# 删除 Pod
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> --now  # 立即删除
```

### Deployment 管理

```bash
# 查看 Deployments
kubectl get deployments
kubectl get deploy

# 创建 Deployment
kubectl create deployment nginx --image=nginx

# 更新镜像
kubectl set image deployment/nginx nginx=nginx:1.21

# 查看滚动更新状态
kubectl rollout status deployment/nginx

# 查看更新历史
kubectl rollout history deployment/nginx

# 回滚到上一个版本
kubectl rollout undo deployment/nginx

# 回滚到特定版本
kubectl rollout undo deployment/nginx --to-revision=2

# 扩缩容
kubectl scale deployment nginx --replicas=5

# 自动扩缩容
kubectl autoscale deployment nginx --min=2 --max=10 --cpu-percent=80

# 重启 Deployment
kubectl rollout restart deployment/nginx
```

### Service 管理

```bash
# 查看 Services
kubectl get services
kubectl get svc

# 创建 Service
kubectl expose deployment nginx --port=80 --type=NodePort

# 查看 Service 详情
kubectl describe service <service-name>

# 删除 Service
kubectl delete service <service-name>
```

### ConfigMap 和 Secret

```bash
# 创建 ConfigMap
kubectl create configmap my-config --from-literal=key1=value1 --from-literal=key2=value2
kubectl create configmap my-config --from-file=config.txt

# 查看 ConfigMap
kubectl get configmaps
kubectl describe configmap my-config

# 创建 Secret
kubectl create secret generic my-secret --from-literal=password=my-password
kubectl create secret generic my-secret --from-file=ssh-privatekey=~/.ssh/id_rsa

# 查看 Secret
kubectl get secrets
kubectl describe secret my-secret
```

### 资源操作

```bash
# 从文件创建资源
kubectl apply -f ./manifest.yaml
kubectl apply -f ./dir/  # 从目录创建

# 从 URL 创建
kubectl apply -f https://example.com/manifest.yaml

# 删除资源
kubectl delete -f ./manifest.yaml

# 编辑资源
kubectl edit deployment/nginx

# 查看资源标签
kubectl get pods --show-labels

# 添加标签
kubectl label pods my-pod new-label=awesome

# 移除标签
kubectl label pods my-pod new-label-
```

### 调试和故障排查

```bash
# 查看事件
kubectl get events
kubectl get events --sort-by=.metadata.creationTimestamp

# 查看警告事件
kubectl events --types=Warning

# 查看 Pod 的事件
kubectl describe pod <pod-name>

# 端口转发
kubectl port-forward <pod-name> 8080:80
kubectl port-forward svc/<service-name> 8080:80

# 复制文件
kubectl cp /tmp/foo <pod-name>:/tmp/bar
kubectl cp <pod-name>:/tmp/foo /tmp/bar

# 查看资源使用情况
kubectl top pods
kubectl top nodes
```

### 节点管理

```bash
# 标记节点为不可调度
kubectl cordon <node-name>

# 清空节点（驱逐 Pod）
kubectl drain <node-name>

# 标记节点为可调度
kubectl uncordon <node-name>

# 查看节点污点
kubectl get nodes -o custom-columns='NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value'

# 添加污点
kubectl taint nodes <node-name> key=value:NoSchedule

# 移除污点
kubectl taint nodes <node-name> key:NoSchedule-
```

### 其他实用命令

```bash
# 查看 API 资源
kubectl api-resources
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false

# 查看资源文档
kubectl explain pods
kubectl explain deployment.spec

# 比较配置差异
kubectl diff -f ./manifest.yaml

# 生成 YAML（不实际创建）
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml

# 强制删除（慎用）
kubectl delete pod <pod-name> --grace-period=0 --force
```

---

## 工作负载管理

### 运行无状态应用

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### 运行有状态应用

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### 运行定时任务

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"  # 每分钟执行一次
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

---

## 服务与网络

### Service 类型

| 类型 | 说明 |
|------|------|
| **ClusterIP** | 默认类型，仅在集群内部可访问 |
| **NodePort** | 在每个节点上开放一个端口，外部可通过 NodeIP:NodePort 访问 |
| **LoadBalancer** | 使用云提供商的负载均衡器，对外暴露服务 |
| **ExternalName** | 将服务映射到外部 DNS 名称 |

### 创建 Service

```yaml
# ClusterIP（默认）
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80

# NodePort
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007  # 范围：30000-32767

# LoadBalancer
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  tyname: my-storage
  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

### 动态存储制备

```yaml
# 创建 StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
allowVolumeExpansion: true
mountOptions:
  - debug
volumeBindingMode: WaitForFirstConsumer

# PVC 自动使用 StorageClass 创建 PV
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

---

## 配置管理

### ConfigMap

```yaml
# 创建 ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  key1: value1
  key2: value2
  config.yaml: |
    server:
      port: 8080
      host: localhost

# 在 Pod 中使用 ConfigMap
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    env:
    - name: KEY1
      valueFrom:
        configMapKeyRef:
          name: my-config
          key: key1
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: my-config
```

### Secret

```yaml
# 创建 Secret
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=  # base64 编码的 "admin"
  password: MWYyZDFlMmU2N2Rm  # base64 编码

# 或使用 stringData（自动 base64 编码）
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
stringData:
  username: admin
  password: mypassword

# 在 Pod 中使用 Secret
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

### 健康检查

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 3
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30
      periodSeconds: 10
```

---

## 最佳实践

### 1. 资源管理

**为容器设置资源请求和限制：**

```yaml
resources:
  requests:
    memory: "64Mi"
    cpu: "250m"
  limits:
    memory: "128Mi"
    cpu: "500m"
```

- **requests**：容器启动时保证的最小资源
- **limits**：容器可使用的最大资源

### 2. 标签和注解

**使用标签组织资源：**

```yaml
metadata:
  labels:
    app: myapp
    environment: production
    version: v1.0
    team: backend
```

**常用标签约定：**
- `app`：应用程序名称
- `version`：应用版本
- `environment`：环境（dev/staging/prod）
- `team`：负责团队

### 3. 命名空间隔离

**为不同环境创建命名空间：**

```bash
kubectl create namespace development
kubectl create namespace staging
kubectl create namespace production
```

### 4. 滚动更新策略

**配置滚动更新参数：**

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # 最多可超出的 Pod 数
      maxUnavailable: 0  # 最多不可用的 Pod 数
```

### 5. Pod 安全

**使用安全上下文：**

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  fsGroup: 2000
  capabilities:
    drop:
    - ALL
```

### 6. 镜像管理

**最佳实践：**
- 使用具体的镜像标签，避免使用 `latest`
- 使用私有镜像仓库
- 定期扫描镜像漏洞
- 使用多阶段构建减小镜像大小

```yaml
containers:
- name: myapp
  image: myregistry.com/myapp:v1.2.3  # 使用具体版本
  imagePullPolicy: IfNotPresent
```

### 7. 日志和监控

**日志最佳实践：**
- 将日志输出到 stdout/stderr
- 使用结构化日志（JSON 格式）
- 包含足够的上下文信息（trace ID、用户 ID 等）

**监控指标：**
- 应用性能指标（请求延迟、错误率等）
- 资源使用指标（CPU、内存等）
- 业务指标（用户数、订单数等）

### 8. 备份和恢复

**定期备份：**
- etcd 数据
- PersistentVolume 数据
- 配置文件和 Secret

**使用 Velero 等工具进行集群备份：**

```bash
# 安装 Velero
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.0.0 \
  --bucket my-backups \
  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://minio:9000

# 创建备份
velero backup create my-backup --include-namespaces production

# 恢复备份
velero restore create --from-backup my-backup
```

### 9. 高可用部署

**多副本部署：**

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  template:
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchLabels:
                  app: myapp
              topologyKey: kubernetes.io/hostname
```

### 10. 安全最佳实践

- **最小权限原则**：使用 RBAC 限制权限
- **网络策略**：限制 Pod 间通信
- **Secret 管理**：使用外部 Secret 管理工具（如 Vault）
- **镜像签名**：验证镜像来源
- **定期更新**：及时更新 Kubernetes 版本和补丁

---

## 故障排查指南

### 常见问题及解决方案

#### Pod 无法启动

```bash
# 查看 Pod 状态
kubectl get pod <pod-name>
kubectl describe pod <pod-name>

# 查看日志
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# 检查事件
kubectl get events --sort-by=.metadata.creationTimestamp
```

**常见原因：**
- 镜像拉取失败：检查镜像名称和凭证
- 资源不足：检查节点资源
- 配置错误：检查 ConfigMap/Secret
- 健康检查失败：检查探针配置

#### Service 无法访问

```bash
# 查看 Service 和 Endpoints
kubectl get svc <service-name>
kubectl get endpoints <service-name>

# 检查 Pod 标签是否匹配
kubectl get pods --show-labels

# 测试连通性
kubectl run test --rm -it --image=busybox -- /bin/sh
# 在容器内测试
wget -qO- http://<service-name>
```

#### 节点异常

```bash
# 查看节点状态
kubectl get nodes
kubectl describe node <node-name>

# 查看节点资源
kubectl top node

# 查看节点事件
kubectl get events --field-selector involvedObject.name=<node-name>
```

### 调试工具

```bash
# 使用 debug 命令创建调试容器
kubectl debug -it <pod-name> --image=busybox

# 临时容器排查
kubectl run debug-pod --rm -it --image=busybox -- /bin/sh

# 网络调试
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- /bin/bash
```

---

## 附录：快速参考卡片

### kubectl 自动补全

```bash
# Bash
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

# Zsh
source <(kubectl completion zsh)
echo '[[ $commands[kubectl] ]] && source <(kubectl completion zsh)' >> ~/.zshrc

# 使用别名
alias k=kubectl
complete -o default -F __start_kubectl k
```

### 常用输出格式

```bash
# YAML 格式
kubectl get pod <name> -o yaml

# JSON 格式
kubectl get pod <name> -o json

# 宽格式（显示更多信息）
kubectl get pods -o wide

# 仅名称
kubectl get pods -o name

# 自定义列
kubectl get pods -o custom-columns=NAME:.metadata.name,STATUS:.status.phase

# JSONPath
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

### 资源简称

| 全称 | 简称 |
|------|------|
| pods | po |
| deployments | deploy |
| services | svc |
| namespaces | ns |
| configmaps | cm |
| secrets | secret |
| persistentvolumeclaims | pvc |
| persistentvolumes | pv |
| ingress | ing |
| serviceaccounts | sa |

---

## 学习资源

### 官方文档
- [Kubernetes 官方文档](https://kubernetes.io/zh-cn/docs/)
- [kubectl 参考](https://kubernetes.io/zh-cn/docs/reference/kubectl/)
- [Kubernetes GitHub](https://github.com/kubernetes/kubernetes)

### 实践环境
- [Kubernetes 官方互动教程](https://kubernetes.io/zh-cn/docs/tutorials/kubernetes-basics/)
- [Killercoda Kubernetes 场景](https://killercoda.com/learn/kubernetes)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)

### 社区资源
- [Kubernetes 中文社区](https://www.kubernetes.org.cn/)
- [Cloud Native Computing Foundation](https://www.cncf.io/)
- [Kubernetes Slack](https://slack.k8s.io/)

---

*最后更新：2026 年 5 月*
*基于 Kubernetes v1.36 编写*














































































































































































































































































































































































































































































































































































































































































































































































































































































