#《Kubernetes基础教程》- 云原生应用架构实战手册

第一章 架构

https://jimmysong.io/kubernetes-handbook/concepts/

https://lib.jimmysong.io/kubernetes-handbook/

### 核心组件

- Etcd
- apiserver
- controller manager  负责维护集群的状态，比如故障检测、自动扩展、滚动更新等;
- scheduler
- kubelet 负责维护容器的生命周期，同时也负责 Volume（CSI）和网络（CNI）的管理；
- Container runtime 负责镜像管理以及 Pod 和容器的真正运行（CRI）；
- Kube-proxy

### 核心插件

- CoreDNS
- Ingress Controller
- Prometheus
- Dashbord
- Fedrtation

### 分层设计

- 核心层
- 应用层
- 管理层
- 接口层
- 生态系统
  - 外部：日志、监控、配置管理、CI、CD、Workflow、FaaS、OTS 应用、ChatOps 等
  - 内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等





Kubernetes 中所有的配置都是通过 API 对象的 spec 去设置的，也就是用户通过配置系统的理想状态来改变系统，这是 Kubernetes 重要设计理念之一，即所有的操作都是声明式（Declarative）的而不是命令式（Imperative）的。



### 核心技术概念和API对象

API对象是k8s中的管理操作单元，每个API对象都有3大类属性：

- metadata  元数据，用来标识API对象，每个API至少以下3个元数据
  - namespace
  - name
  - uid
- spec 规范，描述用户期望集群中系统达到的理想状态
- status 状态



#### Pod

Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。Pod 对多容器的支持是 K8 最基础的设计理念。

Pod 是 Kubernetes 集群中所有业务类型的基础，可以看作运行在 Kubernetes 集群中的小机器人，不同类型的业务就需要不同类型的小机器人去执行。Pod类型对应关系如下

- long-running（长期伺服型） ---- Deployment
- batch（批处理型）—— Job
- Node-daemon（节点后台支撑型）—— DaenomSet
- stateful application（有状态应用型）—— StatefulSet



#### RC\RS\Deployment

- RC 过期了
- RS 不单独使用，用作Deployment的理想状态参数使用
- Deployment  来对所有长期伺服型的的业务的管理，都会通过 Deployment 来管理
  - 创建
  - 更新
  - 滚动更新

#### Service

解决如何访问Pod服务的问题

clusterIP服务是k8s默认服务（Service），集群内的应用可以访问，集群外部无法访问！

k8s三种外部访问service方式：

- NodePort：引导外部流量到pod的最原始方式。
- LoadBalancer：是服务暴漏到internet的标准方式
- Ingress

k8s五种外部访问pod方式：

- hostNetwork
- hostPort
- NodePort
- LoadBalancer
- Ingress

#### Job

job任务成功完成的标志由spec.completions 策略决定

- 单pod型任务：有一个pod成功就标志完成
- 定数成功型任务：保证有N个任务全部成功
- 工作队列型任务：根据应用确认的全局成功而标志成功

#### DaemonSet

典型的后台支撑型服务包括，存储，日志和监控等在每个节点上支持 Kubernetes 集群运行的服务。



#### StatefulSet

对于 StatefulSet 中的 Pod，每个 Pod 挂载自己独立的存储，如果一个 Pod 出现故障，从其他节点启动一个同样名字的 Pod，要挂载上原来 Pod 的存储继续以它的状态提供服务。

StatefulSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性。(保持稳定的映射关系)



#### Federation

联合集群服务就是为提供跨 Region 跨服务商 Kubernetes 集群服务而设计的。



#### Volume

同Docker的存储卷，只是作用范围不同。Docker的作用于一个容器，而k8s的存储卷的生命周期和作用范围是一个pod。每个pod声明的存储卷由pod中的所有容器共享。

支持的存储卷类型非常多

- 支持公有云平台存储卷，AWS、Google、Azure等
- 支持分布式存储，GlusterFS、Ceph
- 本地目录，emptyDir\hostPath\NFS
- 逻辑存储，PVC（Persistent Volume Claim）。使用这种存储，使得存储的使用者可以忽略（屏蔽）后台的实际存储技术，而将有关存储实际技术的配置交给存储管理员通过PV （Persistent Volume）来配置



#### PV\PVC 

- pv -- 持久存储卷
- pvc -- 持久存储卷声明

存储的 PV 和 PVC 的这种关系，跟计算的 Node 和 Pod 的关系是非常类似的；PV 和 Node 是资源的提供者，根据集群的基础设施变化而变化，由 Kubernetes 集群管理员配置；而 PVC 和 Pod 是资源的使用者，根据业务服务的需求变化而变化，有 Kubernetes 集群的使用者即服务的管理员来配置。

Node -- pod

PV -- PVC



#### Node

提供k8s集群的计算能力，最初也叫Minion

统一特征是上面要运行kubelet管理节点上运行的容器



#### Secret

密钥对象

- 用来保存和传递密码、密钥、认证凭证等敏感信息
- 好处是，可以避免把敏感信息写到配置文件 （意图明确、避免重复、减少暴露机会）



#### User Account\ Service Account

用户账户：对应人的身份，与namespace无关

服务账户：对应一个运行程序的身份，与namespace有关



#### Namespace

提供虚拟的隔离作用

默认有两

- default
- kube-system



#### RBAC访问授权

RBAC 权限与角色关联

ABAC 权限与用户关联



### Etcd解析

用于保存集群所有的网络配置和对象的状态信息

整个k8s系统中，两个服务用etcd协同和存储配置

- 网络插件 flannel  （其他网络插件也需要，我们用calico）存储网络配置信息
- k8s本身，包括各种对象的状态和元信息配置

Etcd V2 和 V3 之间的数据结构完全不同，互不兼容，也就是说使用 V2 版本的 API 创建的数据只能使用 V2 的 API 访问，V3 的版本的 API 创建的数据只能使用 V3 的 API 访问。



### 开放接口

k8s作为云原生应用的基础调度平台，相当于操作系统，为了便于系统扩展，提供接口。 Linux 、JVM啥的优秀框架 都会应用类似的设计模式

#### CRI

容器运行时接口  Container Runtime Interface

#### CNI

容器网络接口 Container Network Interface

#### CSI

容器存储接口 Container Storage Interface





### 资源对象

分类

- 资源对象：Pod、RS、RC、Deployment、StatefulSet、DaemonSet、CronJob、Node、Namespace、Service、Ingress、Label、CustomResourceDefinition、HorizontalPodAutoscaling
- 存储对象：Volume、PersistentVolume、Secret、ConfigMap
- 策略对象：SecurityContext、ResourceQuota、LimitRange
- 身份对象：ServiceAccount、Role、ClusterRole



在创建对象的.yaml文件中，需配置如下字段

- apiVersion：创建对象应用的API版本
- kind：对象类型
- metadata：帮助识别对象的唯一性
- spec：对象期望状态。 每类对象的spec格式不一致  

### POD解析

pod是k8s中可创建和部署的最小单位。pod中封装着应用容器、存储、网络ip，管理容器运行策略。

> Docker是k8s中最常用的容器运行时，但是pod也支持其他容器运行时

Pod使用方式，两种

- 一个pod运行一个容器
- 一个pod运行多个容器



#### init容器

在应用程序启动之前运行，包含一些应用镜像中不存在的实用工具或安装脚本

用途：

**init容器使用Linux Namespace，所以相对应用程序容器来说具有不同的文件系统视图，因此，能访问Secret，而应用程序不能**

**能提供一种简单的阻塞或延迟应用容器的启动方法**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

修改init容器的image字段，等价于重启该pod



#### Pause容器

又叫Infra容器

特点

- 镜像非常小
- 永远处于Pause状态

**解决容器之间的网络隔离问题，实现网络共享**

pause容器主要为每个业务容器提供以下功能：

- 在pod中担任linux命名空间共享的基础
- 启用pid命名空间，开启init进程

k8s中容器的PID=1的进程即为容器本身的业务进程



#### 安全策略

Pod安全策略是集群级别的资源，它能够控制pod运行的行为，以及它具有访问什么的能力。

获取策略列表 (hdf没有使用 貌似)

```sh
kubectl get psp
```

#### POD生命周期

##### pod的相位（phase）

- Pending 挂起
- Running 运行中——pod已绑定到一个节点，所有容器已被创建，至少一个容器正在运行，或处于启动或重启状态
- Succeeded 成功——pod中的所有容器都被成功终止，并且不会再重启
- Failed 失败 —— pod中的所有容器都停止了，并且至少有一个容器因为失败终止
- Unknown 未知 —— 无法获取pod状态，通常由于pod所在主机通信失败

##### 容器探针

- ExecAction
- TCPSocketAction
- HTTPGetAction

##### pod重启策略 

podSpec 中 restartPolicy字段，适用于Pod中所有容器

- Always （默认）
- OnFailure
- Never

#### Pod hook

由kubelet发起，当容器中的**进程启动前或启动中的进程终止之前**运行，包含在容器生命周期之中。可以同时为pod中所有的容器都配置hook

hooksystemctl start docker类型

- exec：执行命令
- http：调接口

#### Pod Preset

预设， PodPreset资源对象，是用来在pod被创建的时候向其中注入额外的运行时需求的API资源。可以使用label selector指定为哪些pod应用pod Preset。（相当于变量，模板作者不关心细节）

让一批容器在启动的时候注入一些信息，比如event、volume、volume mount和环境变量，又不想改变pod的templete



每个pod可以匹配0~N个PodPreset



#### Pod中断与PDB（pod中断预算）

适用于构建高可用程序和集群升级、自动扩容等

类型

- 自愿中断
- 非自愿中断

### 集群资源管理

