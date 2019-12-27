## Kubernetes

### 介绍说明
1. 发展历史
  - 公有云类型说明
  - 资源管理器对比
  - K8S 其优势
2. K8S 组件说明
  - Borg 组件说明
  - K8S 结构说明
    - 网络结构
    - 组件结构
3. K8S 中的一些关键字解释

### 基础概念
1. Pod 概念
  - 自助式 Pod
  - 管理器管理的 Pod
    - RS、RC
    - deployment
    - HPA
    - StatefullSet
    - DaemonSet
    - Job，Cronjob
  - 服务发现
  - Pod 协同
2. 网络通讯模式
  - 网络通讯模式说明
  - 组件通讯模式说明

### Kubernetes 安装
1. 系统初始化
2. Kubeadm 部署安装
3. 常见问题分析

### 资源清单
1. K8S 中资源的概念
  - 什么是资源
  - 名称空间级别的资源
  - 集群级别的资源
2. 资源清单 -- yam 语法格式
3. 通过资源清单编写 Pod
4. Pod 的生命周期
  - initc
  - Pod phase
  - 容器探针
    - livenessProbe
    - readinessProbe
  - Pod hook
  - 重启策略

### Pod 控制器
1. Pod 控制器说明
  - 什么是控制器
  - 控制器类型说明
    - ReplicationController 和 ReplicaSet
    - Deployment
    - DaemonSet
    - Job
    - CronJob
    - StatefulSet
    - Horizontal Pod Autoscaling

### 服务发现
1. Service 原理
  - Service 含义
  - Service 常见分类
    - ClusterIP 
    - NodePort
    - ExternalName
  - Service 实现方式
    - userspace 
    - iptables
    - ipvs
2. Ingress
  - Nginx
    - HTTP 代理访问
    - HTTPS 代理访问
    - 使用 cookie 实现会话关联
    - BasicAuth
    - Nginx 进行重写

### 存储
1. configMap
  - 定义概念
  - 创建 configMap
    - 使用目录创建
    - 使用文件创建
    - 使用字面值创建
  - Pod 中使用 configMap
    - ConfigMap 来替代环境变量
    - ConfigMap 设置命令行参数
    - 通过数据卷插件使用 ConfigMap
  - configMap 热更新
    - 实现演示
    - 更新触发说明
2. Secret
  - 定义概念
    - 概念说明
    - 分类
  - service Account
  - Opaque Secret
    - 特殊说明
    - 创建
    - 使用
      - Secret 挂载到 Volume
      - Secret 导出到环境变量中
    - kubernetes.io/dockerconfigjson
3. volume
  - 定义概念
    - 卷的类型
  - emptyDir
    - 说明
    - 用途假设
    - 实验演示
  - hostPath
    - 说明
    - 用途说明
    - 实验演示
4. PV
  - 概念解释
    - PV
    - PVC
    - 类型说明
  - PV
    - 后端类型
    - PV访问模式说明
    - 回收策略
    - 状态
    - 实例演示
  - PVC
    - PVC 实战演示
  
### 调度器
1. 调度器概念
  - 概念
  - 调度过程
  - 自定义调度器
2. 调度亲和性
  - nodeAffinity
    - preferredDuringSchedulingIgnoredDuringExecution
    - requiredDuringSchedulingIgnoredDuringExecution
  - podAntiAffinity
    - preferredDuringSchedulingIgnoredDuringExecution
    - requiredDuringSchedulingIgnoredDuringExecution
  - 亲和性运算符
3. 污点
  - 污点概念
  - Taint
    - 组成
    - 污点的设置、查看和去除
  - Tolerations
    - tolerations 设置演示
4. 固定节点调度
  - PodName 指定调度
  - 标签选择器调度

### 集群安全机制
1. 机制说明
2. 认证
  - HTTP Token
  - HTTP Base
  - HTTPS
3. 鉴权
  - AlwaysDeny
  - AlwaysAllow
  - ABAC
  - Webbook
  - RBAC
    - RBAC
    - Role and ClusterRole
    - RoleBinding and ClusterRoleBinding
    - Resources
    - to Subjects
    - 创建一个系统用户管理 k8s dev 名称空间：重要实验
4. 准入控制

### HELM
1. HELM概念
  - HELM 概念说明
  - 组将构成
  - HELM 部署
  - HELM 自定义
2. HELM 部署实例
  - HELM 部署 dashboard
  - metrics-server
    - HPA 演示
    - 资源限制
      - Pod
      - 名称空间
  - Prometheus
  - EFK

### 运维
1. Kubeadm 源码修改
2. Kubernetes 高可用构建

### 汇总
1. 介绍说明：前世今生 + Kubernetes 框架 + Kubernetes 关键字含义
2. 基础概念：什么是 Pod + 控制器类型 + K8S 网络通讯模式
3. Kubernetes：构建 K8S 集群
4. **资源清单：资源 +  掌握资源清单的语法 + 编写 Pod + 掌握 Pod 的生命周期**
5. Pod 控制器：掌握各种控制器的特点以及使用定义方式
6. 服务发现：掌握 SVC 原理机器构建方式
7. 存储：掌握多种存储类型的特点，并且能够在不同环境中选择合适的存储方案（有自己的见解）
8. 调度器：掌握调度器原理，能够根据要求把 Pod 定义到想要的节点运行
9. 安全：集群的认证 + 鉴权 + 访问控制 + 原理及其流程
10. HELM： linux yum + 掌握 HELM 原理 +  HELM 模版自定义 + HELM 部署一些常用插件
11. 运维： 修改 Kubeadm 达到证书可用期限为 10 年 + 能够构建高可用的 Kubernetes 集群