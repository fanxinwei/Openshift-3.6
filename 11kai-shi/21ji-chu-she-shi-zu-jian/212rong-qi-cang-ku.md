# 容器仓库 {#容器仓库}

## 概述 {#overview}

OpenShift Container Platform可以使用多种镜像源，包括Docker Hub，由第三方运行的私有仓库以及OpenShift Container Platform仓库。

## OpenShift容器仓库 {#integrated-openshift-registry}

OpenShift Container Platform提供了一个集成的容器仓库，该仓库的名称为_OpenShift Container Registry_（OCR），可以根据需要增加自动配置新的镜像仓库。这为用户提供了一个内置的应用程序[构建](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/builds_and_image_streams.html#builds)位置以推送镜像。

无论何时将新镜像推送到OCR，ORC将通知OpenShift Container Platform新镜像的所有信息例如命名空间，名称和镜像元数据。OpenShift Container Platform根据需求创建新的[构建](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/builds_and_image_streams.html#builds)和[部署](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations)。

## 第三方仓库管理机构 {#third-party-registries}

OpenShift Container Platform可以使用来自第三方仓库创建容器，但这些仓库可能无法提供与集成OpenShift Container Platform仓库相同的镜像通知支持。在这种情况下，OpenShift Container Platform将在创建imagetream时从远程仓库中获取标签。刷新获取的标签就像运行一样简单`oc import-image <stream>`。当检测到新镜像时，会发生先前描述的构建和部署反应。

### 认证 {#authentication}

OpenShift Container Platform可以与仓库进行通信，以使用用户提供的凭据访问私有映像存储库。这将允许OpenShift从私有仓库中下载镜像。点击[认证](https://docs.openshift.com/container-platform/3.6/architecture/additional_concepts/authentication.html#architecture-additional-concepts-authentication)含有更多的信息。

