# Google Cloud + Helm + Kubernetes DevOps 操作手册

> 版本：v1.0  
> 最后更新：2026-05-11  
> 作者：DevOps 团队

---

## 目录

1. [概述](#概述)
2. [Google Cloud GKE 最佳实践](#google-cloud-gke-最佳实践)
3. [Helm Chart 部署与管理](#helm-chart-部署与管理)
4. [Kubernetes 日常维护](#kubernetes-日常维护)
5. [监控方案](#监控方案)
6. [常见疑难问题排查](#常见疑难问题排查)
7. [附录：常用命令速查](#附录常用命令速查)

---

## 概述

本手册旨在为 DevOps 团队提供 Google Cloud Platform (GCP)、Helm Chart 和 Kubernetes (GKE) 的综合操作指南，涵盖日常维护流程、监控方案以及常见疑难问题的解决方法。

### 技术栈

- **云平台**: Google Cloud Platform (GCP)
- **容器编排**: Google Kubernetes Engine (GKE)
- **包管理**: Helm 3.x
- **监控**: Cloud Monitoring, Cloud Logging, Managed Prometheus
- **CI/CD**: Cloud Build, GitHub Actions

---

## Google Cloud GKE 最佳实践

### 1.1 集群设计与规划

#### 集群模式选择

| 模式 | 适用场景 | 特点 |
|------|----------|------|
| **Autopilot** | 标准化工作负载、减少运维负担 | Google 管理节点、自动扩缩容、按 Pod 资源计费 |
| **Standard** | 需要高度定制、特殊硬件需求 | 完全控制节点配置、支持自定义镜像 |

#### 集群配置建议

```yaml
# 推荐配置示例
集群版本：使用稳定版本（Regular 或 Stable 渠道）
区域：多区域部署（Regional Cluster）提高可用性
节点池：至少 2 个节点池（系统池 + 应用池）
自动扩缩容：启用 Cluster Autoscaler
自动升级：启用自动升级并配置维护窗口
```

#### 网络配置

```bash
# 创建集群时推荐的网络配置
gcloud container clusters create-auto my-cluster \
  --region us-central1 \
  --network my-vpc \
  --subnetwork my-subnet \
  --enable-private-nodes \
  --enable-master-authorized-networks \
  --master-authorized-networks 10.0.0.0/8
```

### 1.2 安全最佳实践

#### RBAC 权限管理

```yaml
# 最小权限原则 - 创建只读角色
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

```bash
# 绑定角色到用户
kubectl create rolebinding dev-read-binding \
  --role=pod-reader \
  --user=dev-team@example.com
```

#### 工作负载身份（Workload Identity）

```bash
# 启用工作负载身份
gcloud container clusters update CLUSTER_NAME \
  --workload-pool PROJECT_ID.svc.id.goog

# 创建 Kubernetes 服务账号
kubectl create serviceaccount my-app-sa \
  --namespace my-namespace

# 绑定 Google 服务账号
gcloud iam service-accounts add-iam-policy-binding \
  GSA_NAME@PROJECT_ID.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "serviceAccount:PROJECT_ID.svc.id.goog[my-namespace/my-app-sa]"
```

#### 集群加固措施

- ✅ 启用私有集群（Private Cluster）
- ✅ 配置主授权网络（Master Authorized Networks）
- ✅ 启用二进制授权（Binary Authorization）
- ✅ 启用工作负载漏洞扫描
- ✅ 使用 Shielded Nodes
- ✅ 启用 GKE Sandbox（高安全需求场景）

### 1.3 成本优化

#### 资源请求与限制

```yaml
# 必须设置 resources.requests 和 resources.limits
resources:
  requests:
    memory: "256Mi"
    cpu: "250m"
  limits:
    memory: "512Mi"
    cpu: "500m"
```

#### 使用 Spot VMs

```bash
# 创建使用 Spot VMs 的节点池
gcloud container node-pools create spot-pool \
  --cluster=my-cluster \
  --spot \
  --num-nodes=3 \
  --machine-type=e2-standard-4
```

#### 成本监控

```bash
# 查看 GKE 成本分配
gcloud billing accounts list
# 在 Cloud Console 中查看 Cost Allocation 报告
```

### 1.4 集群升级策略

#### 升级流程

---

## Helm Chart 部署与管理

### 2.1 Helm 基础操作

#### 仓库管理

```bash
# 添加仓库
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# 更新仓库索引
helm repo update

# 搜索 Chart
helm search repo nginx
helm search repo --versions
```

#### 发布管理

```bash
# 安装 Chart
helm install my-release bitnami/nginx \
  --namespace production \
  --create-namespace \
  --values values.yaml

# 升级发布
helm upgrade my-release bitnami/nginx \
  --namespace production \
  --values values.yaml \
  --atomic \
  --timeout 10m

# 回滚
helm rollback my-release 1 --namespace production

# 查看历史
helm history my-release --namespace production

# 卸载
helm uninstall my-release --namespace production
```

### 2.2 最佳实践

#### 使用 --atomic 标志

```bash
# 确保失败时自动回滚
helm upgrade --install my-release ./mychart \
  --atomic \
  --timeout 10m \
  --wait
```

#### 版本锁定

```yaml
# Chart.yaml 中指定依赖版本
dependencies:
  - name: postgresql
    version: "13.2.0"  # 使用精确版本
    repository: https://charts.bitnami.com/bitnami
```

#### 多环境配置

```bash
# 使用不同 values 文件
helm install my-release ./mychart \
  --values values.yaml \
  --values values-production.yaml \
  --values values-prod-secrets.yaml
```

### 2.3 CI/CD 集成

#### GitHub Actions 示例

```yaml
name: Deploy to GKE

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: v3.14.0
    
    - name: Authenticate to GCP
      uses: google-github-actions/auth@v2
      with:
        credentials_json: ${{ secrets.GCP_CREDENTIALS }}
    
    - name: Setup kubectl
      uses: google-github-actions/setup-kubectl@v2
    
    - name: Deploy with Helm
      run: |
        helm upgrade --install my-app ./charts/my-app \
          --namespace production \
          --atomic \
          --timeout 10m
```

---

## Kubernetes 日常维护

### 3.1 日常检查清单

#### 每日检查

```bash
# 1. 检查节点状态
kubectl get nodes
kubectl get nodes -o wide

# 2. 检查 Pod 状态
kubectl get pods --all-namespaces | grep -v Running
kubectl get pods --all-namespaces --field-selector=status.phase!=Running

# 3. 检查资源使用
kubectl top nodes
kubectl top pods --all-namespaces

# 4. 检查事件
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -50

# 5. 检查 PVC 状态
kubectl get pvc --all-namespaces

# 6. 检查证书过期
kubectl get certificates --all-namespaces
```

#### 每周检查

```bash
# 1. 检查集群版本
gcloud container clusters describe my-cluster --region us-central1

# 2. 检查节点池状态
gcloud container node-pools list --cluster my-cluster --region us-central1

# 3. 检查资源配额
kubectl describe quota --all-namespaces

# 4. 检查 LimitRange
kubectl describe limitrange --all-namespaces

# 5. 检查 HPA 状态
kubectl get hpa --all-namespaces

# 6. 检查网络策略
kubectl get networkpolicies --all-namespaces
```

#### 每月检查

```bash
# 1. 审计 RBAC 权限
kubectl auth can-i --list --all-namespaces

# 2. 检查未使用的资源
kubectl get all --all-namespaces | grep -E "0/0|0m|0Mi"

# 3. 检查存储使用
kubectl get pv,pvc --all-namespaces

# 4. 审查 Secret
kubectl get secrets --all-namespaces

# 5. 检查 ConfigMap
kubectl get configmaps --all-namespaces
```

### 3.2 备份与恢复

#### 使用 Velero 备份

```bash
# 安装 Velero
velero install \
  --provider gcp \
  --plugins velero/velero-plugin-for-gcp:v1.8.0 \
  --bucket my-backup-bucket \
  --secret-file ./credentials-velero \
  --backup-location-config region=us-central1

# 创建备份
velero backup create daily-backup --include-namespaces production

# 恢复备份
velero restore create --from-backup daily-backup
```

### 3.3 日志管理

```bash
# 查看 Pod 日志
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> -c <container-name>
kubectl logs <pod-name> -n <namespace> --previous  # 查看重启前日志
kubectl logs <pod-name> -n <namespace> -f  # 实时跟踪

# 查看最近 N 行
kubectl logs <pod-name> -n <namespace> --tail=100

# 查看特定时间范围
kubectl logs <pod-name> -n <namespace> --since=1h
```

---

## 监控方案

### 4.1 Cloud Monitoring 集成

#### 关键指标与告警阈值

| 类别 | 指标 | 告警阈值 |
|------|------|----------|
| **节点** | CPU 使用率 | > 80% 持续 5 分钟 |
| **节点** | 内存使用率 | > 85% 持续 5 分钟 |
| **节点** | 磁盘使用率 | > 80% |
| **Pod** | 重启次数 | > 3 次/小时 |
| **Pod** | CPU 节流 | > 10% |
| **应用** | 请求延迟 P99 | > 500ms |
| **应用** | 错误率 | > 1% |

### 4.2 Managed Prometheus

```bash
# 启用 Managed Prometheus
gcloud container clusters update my-cluster \
  --region us-central1 \
  --enable-managed-prometheus
```

#### 创建 PodMonitoring 资源

```yaml
apiVersion: monitoring.googleapis.com/v1
kind: PodMonitoring
metadata:
  name: my-app-monitoring
  namespace: production
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: metrics
    interval: 30s
```

### 4.3 自定义监控指标

```yaml
# Deployment 中添加 annotations
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

---

## 常见疑难问题排查

### 5.1 Pod 问题排查

#### Pod 处于 Pending 状态

**排查步骤**:

```bash
# 1. 查看 Pod 详情
kubectl describe pod <pod-name> -n <namespace>

# 2. 检查事件
kubectl get events -n <namespace> --sort-by='.lastTimestamp'

# 3. 检查资源配额
kubectl describe quota -n <namespace>

# 4. 检查节点资源
kubectl describe nodes | grep -A 5 "Allocated resources"
```

**常见原因及解决**:

| 原因 | 解决方案 |
|------|----------|
| 资源不足 | 增加节点、减少资源请求、启用 Cluster Autoscaler |
| 节点选择器不匹配 | 检查 nodeSelector/affinity 配置 |
| PVC 未绑定 | 检查 StorageClass 和 PV 可用性 |
| 资源配额限制 | 调整 ResourceQuota 或优化资源请求 |

#### Pod 处于 CrashLoopBackOff 状态

**排查步骤**:

```bash
# 1. 查看 Pod 日志
kubectl logs <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous

# 2. 查看 Pod 详情
kubectl describe pod <pod-name> -n <namespace>

# 3. 检查存活探针
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 10 livenessProbe

# 4. 检查资源限制
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 5 resources
```

**常见原因及解决**:

| 原因 | 解决方案 |
|------|----------|
| 应用启动失败 | 检查应用日志、配置文件、依赖服务 |
| 内存不足 (OOMKilled) | 增加内存限制、优化应用内存使用 |
| 存活探针失败 | 调整探针参数、修复应用健康检查 |
| 配置错误 | 检查 ConfigMap、Secret、环境变量 |

#### Pod 处于 ImagePullBackOff 状态

**排查步骤**:

```bash
# 1. 查看 Pod 事件
kubectl describe pod <pod-name> -n <namespace>

# 2. 检查镜像名称
kubectl get pod <pod-name> -n <namespace> -o yaml | grep image

# 3. 检查 Secret
kubectl get secrets -n <namespace>
```

**常见原因及解决**:

| 原因 | 解决方案 |
|------|----------|
| 镜像名称错误 | 修正镜像名称和标签 |
| 认证失败 | 创建/更新 imagePullSecrets |
| 网络问题 | 检查节点网络、防火墙规则 |
| 镜像不存在 | 确认镜像已推送到仓库 |

### 5.2 节点问题排查

#### 节点处于 NotReady 状态

**排查步骤**:

```bash
# 1. 查看节点详情
kubectl describe node <node-name>

# 2. 检查节点条件
kubectl get node <node-name> -o yaml | grep -A 10 conditions

# 3. SSH 到节点检查
gcloud compute ssh <node-name> --zone <zone>

# 4. 检查 kubelet 状态
sudo systemctl status kubelet
sudo journalctl -u kubelet -f

# 5. 检查容器运行时
sudo systemctl status containerd
```

**常见原因及解决**:

| 原因 | 解决方案 |
|------|----------|
| kubelet 停止 | 重启 kubelet: `sudo systemctl restart kubelet` |
| 容器运行时故障 | 重启 containerd: `sudo systemctl restart containerd` |
| 网络插件问题 | 重启 CNI Pod、检查 CNI 配置 |
| 资源耗尽 | 清理磁盘、释放内存 |
| 证书过期 | 轮换证书、重新加入集群 |

### 5.3 存储问题排查

#### PVC 处于 Pending 状态

**排查步骤**:

```bash
# 1. 查看 PVC 详情
kubectl describe pvc <pvc-name> -n <namespace>

# 2. 检查 StorageClass
kubectl get storageclass
kubectl describe storageclass <storageclass-name>

# 3. 检查 PV
kubectl get pv
kubectl describe pv <pv-name>
```

**常见原因及解决**:

| 原因 | 解决方案 |
|------|----------|
| StorageClass 不存在 | 创建 StorageClass 或使用现有 |
| 配额限制 | 调整 StorageQuota 或清理未使用 PVC |
| 动态供应失败 | 检查云提供商权限、配额 |
| 访问模式不匹配 | 调整 accessModes 配置 |

### 5.4 Helm 常见问题

#### Error: cannot re-use a name that is still in use

**解决方案**:
```bash
# 检查现有发布
helm list -A

# 升级而不是安装
helm upgrade --install my-release ./mychart

# 或卸载后重新安装
helm uninstall my-release
helm install my-release ./mychart
```

#### Error: release not found

**解决方案**:
```bash
# 检查所有发布（包括失败的）
helm list -A --all

# 使用 upgrade --install
helm upgrade --install my-release ./mychart
```

#### Error: template parse error

**解决方案**:
```bash
# 调试模板渲染
helm template my-release ./mychart --debug

# 检查语法
helm lint ./mychart

# 验证 values
helm install my-release ./mychart --dry-run --debug
```

#### Error: Kubernetes cluster unreachable

**解决方案**:
```bash
# 检查 kubeconfig
kubectl config current-context
kubectl cluster-info

# 设置正确上下文
kubectl config use-context <context-name>

# 验证连接
gcloud container clusters get-credentials <cluster-name> --region <region>
```

### 5.5 自动扩缩容问题

#### Cluster Autoscaler 不扩容

**排查步骤**:

```bash
# 1. 查看 Autoscaler 日志
kubectl logs -n kube-system -l app=cluster-autoscaler

# 2. 检查不可调度的 Pod
kubectl get pods --all-namespaces -o wide | grep Pending

# 3. 查看事件
kubectl get events --sort-by='.lastTimestamp' | grep -i autoscaler

# 4. 检查节点池限制
gcloud container node-pools describe <pool-name> --cluster <cluster>
```

**常见原因**:
- PodDisruptionBudget 限制过严
- Pod 有本地存储
- 节点池达到最大大小
- 区域 VM 库存不足
- Pod 有 `cluster-autoscaler.kubernetes.io/safe-to-evict: "false"` 注解

#### HPA 不工作

**排查步骤**:

```bash
# 1. 检查 HPA 状态
kubectl get hpa -n <namespace>
kubectl describe hpa <hpa-name> -n <namespace>

# 2. 检查 Metrics Server
kubectl get pods -n kube-system | grep metrics-server
kubectl top pods

# 3. 检查资源请求
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 5 resources
```

---

## 附录：常用命令速查

### GKE 集群管理

```bash
# 创建集群
gcloud container clusters create-auto CLUSTER_NAME --region REGION

# 获取凭证
gcloud container clusters get-credentials CLUSTER_NAME --region REGION

# 列出集群
gcloud container clusters list

# 描述集群
gcloud container clusters describe CLUSTER_NAME --region REGION

# 删除集群
gcloud container clusters delete CLUSTER_NAME --region REGION
```

### kubectl 常用命令

```bash
# 资源操作
kubectl get <resource> [-n namespace]
kubectl describe <resource> <name> [-n namespace]
kubectl apply -f <file.yaml>
kubectl delete <resource> <name> [-n namespace]

# 调试
kubectl logs <pod> [-c container] [-f] [--previous]
kubectl exec -it <pod> -- /bin/sh
kubectl port-forward <pod> <local-port>:<pod-port>
kubectl cp <pod>:<path> <local-path>

# 故障排查
kubectl top nodes/pods
kubectl get events --sort-by='.lastTimestamp'
kubectl auth can-i <verb> <resource>
```

### Helm 常用命令

```bash
# 仓库管理
helm repo add <name> <url>
helm repo update
helm search repo <keyword>

# 发布管理
helm install <name> <chart>
helm upgrade <name> <chart>
helm uninstall <name>
helm list [-A]
helm history <name>
helm rollback <name> <revision>

# 调试
helm lint <chart>
helm template <name> <chart> --debug
helm install <name> <chart> --dry-run --debug
```

### 监控与日志

```bash
# Cloud Monitoring
gcloud monitoring policie

```bash
# 1. 检查可用版本
gcloud container get-server-config --region us-central1

# 2. 升级控制平面
gcloud container clusters upgrade my-cluster \
  --cluster-version=1.28 \
  --region us-central1

# 3. 升级节点池
gcloud container clusters upgrade my-cluster \
  --node-pool=default-pool \
  --cluster-version=1.28 \
  --region us-central1
```

#### 维护窗口配置

```bash
gcloud container clusters update my-cluster \
  --maintenance-window START_TIME \
  --maintenance-window-duration HOURS \
  --region us-central1
```
