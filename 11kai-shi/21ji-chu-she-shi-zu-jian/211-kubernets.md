# Kubernetes {#kubernetes}

## 概述 {#overview}

在OpenShift Container Platform中，Kubernetes可以通过一组容器或主机来管理集装箱化应用程序，并提供部署、维护和应用程序扩展的机制。Kubernetes可以打包，实例化和运行容器化应用程序。

Kubernetes集群由一个或多个masters和一组nodes组成。您可以为主机配置[高可用性](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#high-availability-masters)（[high availability](https://docs.openshift.com/container-platform/3.6/architecture/infrastructure_components/kubernetes_infrastructure.html#high-availability-masters)，HA）以确避免集群单点失效的情况发生。

> ![](/assets/提示3%.png)OpenShift Container Platform3.6使用Kubernetes 1.6版本和Docker 1.12版本。

## Masters {#master}

master为一个或者多个主机，包括API服务器，控制器管理器服务器和**etcd**。master管理其Kubernetes集群中的[nodes](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#node)，并调度[pod](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#pods)在nodes上运行。

表1. master组件

| 组件 | 描述 |
| :--- | :--- |
| API服务器 | Kubernetes API服务器提供验证和配置pod的数据，服务和复制控制器。此外，它还可以将pod分配给node，并将pod信息与配置同步。可以作为独立进程运行。 |
| etcd | etcd存储持续的master状态，其他组件将监视etcd的变动以保持所需状态。可以选择将etcd配置为高可用性，通常采用2n + 1对等服务进行部署。 |
| 控制器管理服务器 | 控制器管理器服务器查看etcd的状态**，**更改复制控制器对象，然后使用API​​强制执行所需的状态。该组件可以作为独立进程运行。通常在一个集群下创建一个控制器管理器服务器 |
| HAProxy | 可选，在配置多个主节点高可用性的环境时使用，可将请求负载均匀分配到多个主节点。[高级安装方法](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#install-config-install-advanced-install)可以为您配置HAProxy的`native`方法。或者您可以使用该`native`方法，但得预先配置自己的负载均衡器。 |

### 主节点的高可用性配置 {#high-availability-masters}

在单个主节点的配置中，虽然当主节点或其任何服务失败时，运行的应用程序的可用仍将保留。但是主节点发生故障将降低系统响应或创建新应用程序的能力。您可以选择为主节点配置高可用性（HA）——即多个主节点，以确保集群不会发生单点故障。

> ![](/assets/提示3%.png)不支持在安装后从单个主群集迁移到多个主群。

当使用`native`HAProxy的HA方法时，主节点组件需具有以下特性：

| 角色 | 模式 | 其他 |
| :--- | :--- | :--- |
| etcd | 主动 - 主动 | 通过负载均衡实现冗余部署 |
| API服务器 | 主动 - 主动 | 由HAProxy管理 |
| 控制器管理服务器 | 主动 - 被动 | 在一个时间允许一个实例被选举为控制器管理服务器 |
| HAProxy | 主动 - 被动 | 分发负载在多个主节点 |

## Nodes（节点） {#node}

节点为容器提供运行的环境。Kubernetes集群中的每个节点可以被master进行管理。节点还具有运行pod的所需服务，包括Docker服务，[kubelet](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#kubelet)和[服务代理](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#service-proxy)。

OpenShift Container Platform可从云提供商，物理系统或虚拟系统创建节点。Kubernetes与这些[节点对象](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#node-object-definition)进行交互。master通过节点对象采集的信息来验证节点是否正常运行，当节点通过master持续的运行状况检查后才能被使用。可参考[Kubernetes文档](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/admin/node.md#node-management)，里面提供了更多对节点管理的信息。

管理员可以使用CLI[管理](https://docs.openshift.com/container-platform/3.5/admin_guide/manage_nodes.html#admin-guide-manage-nodes)OpenShift Container Platform实例中的[节点](https://docs.openshift.com/container-platform/3.5/admin_guide/manage_nodes.html#admin-guide-manage-nodes)。要在启动节点服务器时定义完整的配置和安全选项，请使用[专用节点配置文件](https://docs.openshift.com/container-platform/3.5/install_config/master_node_configuration.html#install-config-master-node-configuration)。

> 、![](/assets/警告3.5%.png)建议的最大节点数为300

### Kubelet {#kubelet}

每个节点都有一个kubelet来更新由容器清单（container manifest）指定的节点，容器清单是描述一个pod的YAML文件。kubelet使用一组清单来确保其容器启动并继续运行。示例清单可以在[Kubernetes文档](https://cloud.google.com/compute/docs/containers/container_vms#container_manifest)找到。

### 服务代理 {#service-proxy}

每个节点还运行一个简单的网络代理，以反映在该节点上的API中定义的服务。允许节点在一组后端执行简单的TCP和UDP流转发。

### 节点对象定义 {#node-object-definition}

以下是Kubernetes中的节点对象定义示例：

```
apiVersion: v1     【1】
kind: Node         【2】
metadata:
  creationTimestamp: null
  labels:          【3】
    kubernetes.io/hostname: node1.example.com
  name: node1.example.com     【4】
spec:
  externalID: node1.example.com    【5】
status:
  nodeInfo:
    bootID: ""
    containerRuntimeVersion: ""
    kernelVersion: ""
    kubeProxyVersion: ""
    kubeletVersion: ""
    machineID: ""
    osImage: ""
    systemUUID: ""
```

【1】`apiVersion`定义要使用的API版本

【2】将`kind`设置为`Node`为节点对象的定义

【3】`metadata.labels`列出已添加到节点的任何[标签](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#labels)

【4】`metadata.name`是定义节点对象名称的必需值

【5】`spec.externalID`定义可以到达节点的完全限定域名。`metadata.name`为空时将被设置为默认值

更多细节请参考[REST API](https://docs.openshift.com/container-platform/3.5/rest_api/kubernetes_v1.html#rest-api-kubernetes-v1)

