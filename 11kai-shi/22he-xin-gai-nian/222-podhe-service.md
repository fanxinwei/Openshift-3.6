# Pods和Services {#pods和服务}

## Pods {#pods}

OpenShift Container Platform使用了Kubernetes中Pod的概念，Pod即一个主机，可以在上面部署一个或多个[容器](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/containers_and_images.html#containers)，Pods是可以被定义，部署和管理的最小的计算单元。

Pods粗略的看成一个虚拟机或者主机。每个pod都分配了自己的内部IP地址，因此拥有其整个端口的空间，pod中的容器可以共享其本地存储和网络。

Pods具有生命周期，当它们被定义后将被分配为在node上运行，直到其包含的容器退出或者由于某些其他原因被移除。根据制定的策略和退出代码，Pods可能在退出直接被删除，也可能保留其日志以便后续访问。

OpenShift Container Platform将pods视作基本不变的，在pod正在运行时，无法进行更改。OpenShift Container Platform会终止现有的pod再修改基础镜像、配置，或两者均修改。由于pod的创建时的状态并不是稳定的，因此，pod通常由更高级别的[控制器](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/deployments.html#replication-controllers)进行管理，而不是由用户直接管理。

> ![](/assets/警告3.5%.png)每个OpenShift Container Platform的pods建议最大个数为110个。

以下是提供长时间运行服务的pod的示例，实际上这是OpenShift Container Platform基础架构的一部分：集成容器镜像仓库。它展示了pod的许多特征，其中大部分在其他小节中进行了讨论，因此在这里仅作简要说明：

示例1. Pod对象定义（YAML）

```
apiVersion: v1
kind: Pod
metadata:
  annotations: { ... }
  labels:                                【1】 
    deployment: docker-registry-1
    deploymentconfig: docker-registry
    docker-registry: default
  generateName: docker-registry-1-       【2】
spec:
  containers:                            【3】
  - env:                                 【4】
    - name: OPENSHIFT_CA_DATA
      value: ...
    - name: OPENSHIFT_CERT_DATA
      value: ...
    - name: OPENSHIFT_INSECURE
      value: "false"
    - name: OPENSHIFT_KEY_DATA
      value: ...
    - name: OPENSHIFT_MASTER
      value: https://master.example.com:8443
    image: openshift/origin-docker-registry:v0.6.2 【5】
    imagePullPolicy: IfNotPresent
    name: registry
    ports:                         【6】     
    - containerPort: 5000
      protocol: TCP
    resources: {}
    securityContext: { ... }           【7】 
    volumeMounts:                      【8】
    - mountPath: /registry
      name: registry-storage
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-br6yz
      readOnly: true
  dnsPolicy: ClusterFirst
  imagePullSecrets:
  - name: default-dockercfg-at06w
  restartPolicy: Always
  serviceAccount: default               【9】
  volumes:                              【10】
  - emptyDir: {}
    name: registry-storage
  - name: default-token-br6yz
    secret:
      secretName: default-token-br6yz
```

> @提示：此处的pod定义不包括在创建pod并且其生命周期开始之后由OpenShift Container Platform自动填充的属性。[Kubernetes API文档](https://docs.openshift.com/container-platform/3.5/rest_api/kubernetes_v1.html#rest-api-kubernetes-v1)有pod REST API对象属性的完整信息，以及[Kubernetes POD文档](https://kubernetes.io/docs/concepts/workloads/pods/pod/)有pods的功能和用途的细节。

【1】Pod可以用一个或多个[标签](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#labels)，可以在单个操作中选择和管理Pod组。标签以密钥/值格式存储在`metadata`。在该示例中的一个标签是**docker-registry = default**。

【2】Pods在其[命名空间中](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/projects_and_users.html#namespaces)必须有一个唯一的名称。在pod定义中可以指定具有`generateName`属性的名称为基础名字，并且将自动添加随机字符以生成唯一的名称。

【3】`containers`给定一组容器的定义，但多数情况下只有一个。

【4】环境变量，将必需的值传递给其他的容器。

【5】pod中的每个容器都是从其自己的基于[Docker格式的容器镜像](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/containers_and_images.html#docker-images)实例化来的。

【6】容器可以绑定到将在pod的IP上可用的端口。

【7】OpenShift Container Platform定义了容器的[安全文件](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/authorization.html#security-context-constraints)，指定是否允许它们作为特权容器运行，以及选择的用户身份运行等等。默认安全文有权限限制，但是管理员可以根据需要进行修改。

【8】容器指定在了容器外部的存储卷的位置。在这个示例中，一个存储卷用于用于存储仓库数据，一个存储卷用于存储访问仓库对OpenShift Container Platform API进行请求的凭据。

【9】针对OpenShift Container PlatformAPI提出请求的脚本是一个常见的模式，它有一个`serviceAccount`字段用于指定应用程序在发出请求应验证哪个[服务帐户](https://docs.openshift.com/container-platform/3.5/dev_guide/service_accounts.html#dev-guide-service-accounts)用户。这样可以实现定制基础架构组件的细粒度访问控制。

【10】pod定义了包含的所有容器（一个或是多个）的可用的存储容量。它为容器仓库存储提供了一个临时卷，还提供了一个包含服务帐户凭据的`secret`卷。

> ![](/assets/提示3%.png)此处的pod的定义不包括在创建pod并且其生命周期开始之后由OpenShift Container Platform自动填充的属性。[Kubernetes API文档](https://docs.openshift.com/container-platform/3.5/rest_api/kubernetes_v1.html#rest-api-kubernetes-v1)有pod REST API对象属性的完整信息、功能和用途的细节。

## Init Container——初始化状态容器 {#pods-services-init-containers}

> ![](/assets/铅笔-3.5%.png.png)在kubernetes 1.3的POD中有两类容器：一类是系统容器（POD Container），一类是用户容器（User Container）；在用户容器中，又分成两类容器：一类是初始化容器（Init Container），一类是应用容器（App Container）。Init Container是做初始化工作的容器。可以有一个或多个，如果有多个，这些 Init Container 按照定义的顺序依次执行，只有所有的InitContainer 执行完后。真正的应用容器才启动。

一个[初始化容器](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)在APP Container运行之前就启动了。Init容器可以共享卷，执行网络操作并执行计算。Init容器还可以阻止或延迟app container的启动，直到满足某些先决条件。

当一个Pod被启动，而此时网络和卷还未被安装，init container就开始按顺序启动了。在下一个init container被调用之前，每个init container必须成功退出。如果init container无法启动或者启动失败而退出，则[`restartPolicy`](https://docs.openshift.com/container-platform/3.6/dev_guide/configmaps.html#consuming-configmap-in-pods)：

* `Always`- 尝试连续重新启动，（10s，20s，40s）的指数回退延迟，直到重新启动。

* `Never`- 不要尝试重新启动。豆荚立即失败并退出。

* `OnFailure`- 尝试以（10秒，20秒，40秒）的指数回退延迟在5分钟时间后重新启动。

在所有init容器成功之前，一个容器不能准备好。

有关[init初始化容器使用示例，](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/#examples)请参阅Kubernetes文档。

以下示例概述了一个具有两个init容器的简单Pod。第一个init容器等待，`myservice`第二个等待`mydb`。一旦两个容器都成功，Pod就会启动。

示例2.Init Container Container对象定义（YAML）

```
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
  - name: init-myservice     【1】
    image: busybox
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb          【2】
    image: busybox
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```

【1】指定`myservice`容器。

【2】指定`mydb`容器。

每个init container都有[一个应用程序容器](https://docs.openshift.com/container-platform/3.6/architecture/core_concepts/pods_and_services.html#example-pod-definition)所有[字段，](https://docs.openshift.com/container-platform/3.6/architecture/core_concepts/pods_and_services.html#example-pod-definition)除了[`readinessProbe`](https://docs.openshift.com/container-platform/3.6/dev_guide/application_health.html#container-health-checks-using-probes)。init container必须退出以使pod启动继续，并且不能定义完成之外的准备就绪。

init container可以包含在Pod上的[`activeDeadlineSeconds`](https://docs.openshift.com/container-platform/3.6/dev_guide/jobs.html#jobs-setting-maximum-duration)和在容器上的[`livenessProbe`](https://docs.openshift.com/container-platform/3.6/dev_guide/application_health.html#container-health-checks-using-probes)，以防止init container永远失败。

## Service {#services}

Kubernetes[服务](http://kubernetes.io/docs/user-guide/services)作为内部负载均衡器。它识别一组[pod](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#pods)，以便代理它接收的连接。在服务可用的情况下，可以任意添加备份pod或从服务中删除服务，使任何依赖于服务的内容都可以在一致的地址处引用。默认服务clusterIP地址来自OpenShift Container Platform内部网络，它们用于允许pod相互访问。

要允许对服务的外部访问，可以将额外的`externalIP`和群集[外部的](https://docs.openshift.com/container-platform/3.5/dev_guide/getting_traffic_into_cluster.html#using-externalIP)`ingressIP`地址分配给服务。这些地址也可以是提供[高可用性](https://docs.openshift.com/container-platform/3.5/admin_guide/high_availability.html#admin-guide-high-availability)访问服务的虚拟IP地址。

服务被分配一个IP地址和端口对，当被访问时，代理到适当的pod。服务使用标签选择器来查找在特定端口上提供特定网络服务的所有运行的容器。

像pod一样，服务是REST对象。以下示例显示了上述定义的服务的定义：

示例2.服务对象定义（YAML）

```
apiVersion: v1
kind: Service
metadata:
  name: docker-registry      【1】
spec:
  selector:                   【2】
    docker-registry: default
  portalIP: 172.30.136.123   【3】
  ports:
  - nodePort: 0
    port: 5000               【4】
    protocol: TCP
    targetPort: 5000          【5】
```

| 【1】服务名称docker-registry还用于构造一个带有服务IP的环境变量，该服务IP插入同一命名空间中的其他pod中。最大名称长度为63个字符。 |
| :--- |
| 【2】标签选择器标识所有pods，其中附加了**docker-registry = default**标签作为备份pod。 |
| 【3】服务的虚拟IP，从内部IP池创建时自动分配。 |
| 【4】端口服务侦听。 |
| 【5】服务转发连接的后台上的端口。 |

该[Kubernetes文件](http://kubernetes.io/docs/user-guide/services/)对服务的更多信息。

### 服务externalIPs {#service-externalip}

除了集群的内部IP地址，用户还可以配置[集群外部的](https://docs.openshift.com/container-platform/3.5/dev_guide/getting_traffic_into_cluster.html#getting-traffic-into-cluster)IP地址。管理员负责确保流量到达具有此IP的节点。

外部IP必须由[_**master-config.yaml**_](https://docs.openshift.com/container-platform/3.5/admin_guide/tcp_ingress_external_ports.html#unique-external-ips-ingress-traffic-configure-cluster)文件中配置的**ExternalIPNetworkCIDRs**范围，由集群管理员选择。当_**master-config.yaml**_更改时，主服务必须重新启动。

示例3. SampleIPNetworkCIDR /etc/origin/master/master-config.yaml

```
networkConfig：

  ExternalIPNetworkCIDR：172.47.0.0/24
```

示例4.服务externalIPs定义（JSON）

```
{
    "kind": "Service",
    "apiVersion": "v1",
    "metadata": {
        "name": "my-service"
    },
    "spec": {
        "selector": {
            "app": "MyApp"
        },
        "ports": [
            {
                "name": "http",
                "protocol": "TCP",
                "port": 80,
                "targetPort": 9376
            }
        ],
        "externalIPs" : [
            "80.11.12.10"     【1】         
        ]
    }
}
```

| 【1】 | **端口** | 暴露的外部IP地址列表。除内部IP地址外） |
| :--- | :--- | :--- |
|  |  |  |

### 服务 ingressIP {#service-ingressip}

在非云集群中，可以从地址池自动分配externalIP地址。这样就无需管理员手动分配。

池配置在_**/etc/origin/master/master-config.yaml**_文件中。更改此文件后，重新启动主服务。

将`ingressIPNetworkCIDR`被设置为`172.29.0.0/16`默认。如果集群环境尚未使用此专用范围，请使用默认范围或设置自定义范围。

> 如果您使用的是高可用性，则该范围必须小于256个地址。

示例5. ingressIPNetworkCIDR /etc/origin/master/master-config.yaml

```
networkConfig：

  ingressIPNetworkCIDR：172.29.0.0/16
```

### 服务NodePort {#service-nodeport}

设置服务`type=NodePort`将从标志配置的范围（默认值：30000-32767）分配一个端口，每个节点将代理该端口（每个节点上相同的端口号）到您的服务中。

所选端口将在服务配置`spec.ports[*].nodePort`中显示、

要指定一个自定义端口，只需将端口号放在nodePort字段中。自定义端口号必须在nodePorts的配置范围内。当'**master-config.yaml**'更改时，master服务必须重新启动。

示例6. servicesNodePortRange /etc/origin/master/master-config.yaml示例

```
kubernetesMasterConfig：

  servicesNodePortRange：“”
```

该服务将被可见当开启`<NodeIP>:spec.ports[].nodePort`和`spec.clusterIp:spec.ports[].port`

> @提示：设置nodePort是一种特权操作。

### 服务代理模式 {#service-proxy-mode}

OpenShift Container Platform提供两种不同的路由服务。默认模式是基于**iptables**，使用概率性**iptables**重写规则来分发端点pod之间的服务连接。另外一个老的模式是使用用户空间进程来接受传入的连接，然后在客户端和端点pod之代理流量。

基于**iptables**的实现效率更高，但它要求所有端点始终能够接受连接;用户空间模式实现较慢，但可以轮询尝试多个端点，直到找到可用的端点。如果您有良好的[准备状态检查](https://docs.openshift.com/container-platform/3.5/dev_guide/application_health.html#dev-guide-application-health)（或通常可靠的节点和pod），那么基于**iptables**的服务代理是最佳选择。否则，您可以在安装时启用基于用户空间的代理，或通过编辑节点配置文件部署群集后启用。

## 标签 {#labels}

标签用于组织、分组或选择API对象。例如，[pod](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#pods)通过标签“标记”，然后[服务](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#services)使用标签选择器来标识它们代理的pod。这使得服务可以引用，甚至将具有潜在不同容器的作为相关实体作整体处理。

大多数对象可以在其元数据中包含标签。因此，标签可用于分组任意相关的对象；例如，[可以](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations)对一个特定应用的所有[pod](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#pods)，[服务](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/pods_and_services.html#services)，[复制控制器](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/deployments.html#replication-controllers)和[部署配置](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations)进行分组。

标签是简单的键/值对，如以下示例所示：

```
labels:
  key1: value1
  key2: value2
```

考虑：

* 由**nginx**容器组成的pod，标签为**role = webserver**。

* 由**Apache httpd**容器组成的pod，具有相同的label**= webserver**标签。

定义为使用具有**role = webserver**标签的pod的服务或复制控制器将这两个pod视为同一组的一部分。

该[Kubernetes文件](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/user-guide/labels.md)对标签的更多信息。

## 端点 {#endpoints}

返回服务的服务器称为其端点，并由具有与服务名称相同的**端点**类型的对象指定。当服务由一个pod返回时，这些pod通常由标签选择器指定，OpenShift Container Platform会自动创建指向这些pod的端点对象。

在某些情况下，您可能需要创建一个服务，但是是在OpenShift Container Platform集群中的外部主机返回服务。在这种情况下，您可以省略`selector`服务中的字段，并[手动创建端点对象](https://docs.openshift.com/container-platform/3.5/dev_guide/integrating_external_services.html#dev-guide-integrating-external-services)。

请注意，OpenShift Container Platform不会让大多数用户手动创建一个端点对象，该对象指向[为pod和服务IP保留的网络块中的IP地址](https://docs.openshift.com/container-platform/3.5/install_config/configuring_sdn.html#configuring-the-pod-network-on-masters)。只有拥有[资源](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/authorization.html#evaluating-authorization)[权限](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/authorization.html#evaluating-authorization)[集群管理员](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/authorization.html#roles)或其他用在户[`createendpoints/restricted`](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/authorization.html#evaluating-authorization)才能创建此类端点对象。

