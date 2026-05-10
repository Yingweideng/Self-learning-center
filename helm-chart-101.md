# Helm Chart 101：从零到一的完全指南

---

### **什么是 Helm？**

Helm 是 Kubernetes 生态中的包管理器，由云原生计算基金会（CNCF）维护的毕业项目。它解决了裸写 Kubernetes YAML 时面临的三大痛点：环境配置重复、缺乏版本追踪、难以回滚。Helm 将一组 Kubernetes 资源清单打包为一个**Chart（图表）**，通过模板引擎实现一次编写、多处部署。简单来说，Helm 之于 Kubernetes 就如同 apt 之于 Ubuntu、yum 之于 CentOS。

Helm v3（2019 年底发布）彻底移除了服务端组件 Tiller，直接使用你的 kubeconfig 凭据与 Kubernetes API 通信，既简化了架构也消除了 v2 时代的重大安全隐患。目前 Helm v2 已停止维护，所有新项目都应使用 Helm v3。

### **三大核心概念**

要掌握 Helm，需要理解三个基石：Chart、Release 和 Repository。

**Chart** 是一个自包含的 Kubernetes 应用包，它包含了你部署一个应用所需的所有 YAML 模板和配置默认值。一个 Chart 可能包含 Deployment、Service、ConfigMap、Ingress 等资源的模板定义。

**Release** 是 Chart 的一次运行实例。同一个 Chart 可以在同一个集群中被安装多次，每次安装都会创建一个独立的 Release，拥有自己的名称和版本历史。比如你用同一个 Nginx Chart 部署了"前端"和"后端"两个环境，它们就是两个不同的 Release。

**Repository** 是 Chart 的仓库，用于存储和分发 Chart 包。社区最知名的仓库是 Bitnami，你可以在 [Artifact Hub](https://artifacthub.io/) 上搜索数百个公开的 Chart。

---

### **安装 Helm**

Helm 的安装非常简单，无需服务端组件。请根据你的操作系统选择安装方式。

macOS 用户使用 Homebrew：

```bash
brew install helm
```

Linux 用户使用官方安装脚本：

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

Windows 用户使用 Chocolatey：

```bash
choco install kubernetes-helm
```

安装后验证版本：

```bash
helm version
```

Helm 直接复用你已有的 `kubeconfig` 配置，只要 `kubectl` 能连上集群，Helm 就可以工作，无需额外初始化。

---

### **Chart 目录结构详解**

一个标准的 Chart 目录结构如下：

```
mychart/
├── Chart.yaml          # Chart 元信息：名称、版本、描述
├── values.yaml         # 默认配置值
├── charts/             # 子 Chart（依赖）
├── templates/          # Kubernetes 清单模板
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl    # 可复用的模板辅助函数
│   ├── NOTES.txt       # 安装后显示的帮助信息
│   └── tests/
│       └── test-connection.yaml
└── .helmignore         # 类似 .gitignore，打包时忽略文件
```

**Chart.yaml** 是 Chart 的身份证，核心字段如下：

```yaml
apiVersion: v2           # v2 对应 Helm 3，v1 对应旧版本
name: nginx-chart        # Chart 名称
description: My First Helm Chart
type: application        # application 或 library
version: 0.1.0           # Chart 包版本（遵循语义化版本）
appVersion: "1.16.0"     # 内部应用的版本号
maintainers:
  - email: dev@example.com
    name: devops-team
```

**values.yaml** 是配置的"司令部"，所有可变的参数都在这里声明默认值：

```yaml
replicaCount: 1
image:
  repository: nginx
  tag: "1.16.0"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  limits:
    cpu: 500m
    memory: 512Mi
```

**templates/** 目录中的文件使用 Go 模板语法编写，支持条件判断、循环、变量引用等能力。模板通过 `{{ }}` 的方式引用 values.yaml 中的配置。

**charts/** 目录存放子 Chart 依赖。如果你的应用依赖 MySQL，可以将 MySQL Chart 放到这个目录下。

**charts/_helpers.tpl** 是模板辅助函数库，用于定义可在整个 Chart 中复用的命名模板片段。

---

### **Helm 操作命令速查**

以下是日常使用频率最高的 Helm 命令：

```bash
# 仓库管理
helm repo add bitnami https://charts.bitnami.com/bitnami  # 添加仓库
helm repo update                                            # 更新仓库
helm repo list                                              # 查看仓库列表

# 搜索与查看
helm search repo bitnami                                    # 搜索仓库中的 Chart
helm search hub nginx                                       # 搜索 Artifact Hub
helm show chart bitnami/nginx                               # 查看 Chart 基本信息
helm show values bitnami/nginx                              # 查看 Chart 默认值
helm show all bitnami/nginx                                 # 查看 Chart 全部信息

# 安装与部署
helm install my-release bitnami/nginx                       # 安装 Chart
helm install my-release ./mychart                           # 从本地目录安装
helm install my-release ./mychart -f prod-values.yaml       # 使用自定义值文件
helm install my-release ./mychart --set image.tag=v2        # 命令行覆盖值

# 查看与管理 Release
helm list                                                   # 列出已部署的 Release
helm list -n my-namespace                                   # 指定命名空间
helm status my-release                                      # 查看 Release 状态
helm history my-release                                     # 查看 Release 历史版本

# 升级与回滚
helm upgrade my-release ./mychart                           # 升级 Release
helm upgrade --install my-release ./mychart                 # 安装或升级（CI/CD 推荐）
helm rollback my-release 1                                  # 回滚到修订版 1
helm rollback my-release                                    # 回滚到上一个版本

# 卸载
helm uninstall my-release                                   # 卸载 Release
helm uninstall my-release --keep-history                    # 卸载但保留历史记录

# 验证与调试
helm lint ./mychart                                         # 检查 Chart 语法
helm template ./mychart                                     # 本地渲染模板
helm install --dry-run --debug my-release ./mychart         # 试运行并打印渲染结果
helm get manifest my-release                                # 获取已安装的清单
helm get values my-release                                  # 获取当前生效的值
```

---

### **创建你的第一个 Chart**

我们来从头创建一个 Nginx Chart，亲手体验完整的流程。

Step 1：使用 `helm create` 命令创建脚手架：

```bash
helm create nginx-chart
cd nginx-chart
```

Step 2：删除默认模板文件，从零构建：

```bash
rm -rf templates/*
```

Step 3：创建 `templates/configmap.yaml`，用 Go 模板语法引入 Release 名称：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-index-html-configmap
data:
  index.html: |
    <html>
      <h1>Welcome</h1>
      <p>Deployed in {{ .Values.env.name }} Environment via Helm</p>
    </html>
```

Step 4：创建 `templates/deployment.yaml`，模板化多个参数：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
  labels:
    app: nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-index-file
              mountPath: /usr/share/nginx/html/
      volumes:
        - name: nginx-index-file
          configMap:
            name: {{ .Release.Name }}-index-html-configmap
```

Step 5：创建 `templates/service.yaml`：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-service
spec:
  selector:
    app: nginx
  type: {{ .Values.service.type }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 80
```

Step 6：更新 `values.yaml`，填充你需要的默认值：

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.16.0"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
env:
  name: dev
```

Step 7：验证 Chart 的正确性：

```bash
helm lint .
helm template .
```

Step 8：部署到集群：

```bash
helm install frontend .
```

Step 9：查看部署结果：

```bash
helm list
kubectl get pods,configmap,service
```

---

### **模板语法与内置对象**

Helm 使用 Go 模板引擎，所有的模板指令包裹在 `{{ }}` 中。这是你最常用的几个内置对象：

**Release 对象**——与当前部署版本相关的信息：

| 表达式 | 含义 |
|---|---|
| `{{ .Release.Name }}` | Release 名称 |
| `{{ .Release.Namespace }}` | 目标命名空间 |
| `{{ .Release.Revision }}` | 修订版本号 |
| `{{ .Release.IsInstall }}` | 是否为安装操作 |
| `{{ .Release.IsUpgrade }}` | 是否为升级操作 |
| `{{ .Release.Service }}` | 始终为 "Helm" |

**Chart 对象**——来自 C



































































































































































































































