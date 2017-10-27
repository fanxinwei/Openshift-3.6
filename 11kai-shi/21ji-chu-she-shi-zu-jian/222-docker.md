# 容器仓库 {#容器仓库}

## 概述 {#overview}

OpenShift Container Platform可以采用多种容器来源，包括Docker Hub，由第三方运行的私有仓库以及OpenShift Container Platform仓库。

## OpenShift容器仓库 {#integrated-openshift-registry}

OpenShift Container Platform提供了一个集成的容器仓库，该仓库的名称为_OpenShift Container Registry_（OCR），可以根据需要增加自动配置新的镜像仓库。这为用户提供了一个内置的应用程序[构建](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/builds_and_image_streams.html#builds)位置，以推送结果图像。

无论何时将新映像推送到OCR，注册表通知OpenShift容器平台关于新映像，传递所有关于它的信息，例如命名空间，名称和映像元数据。OpenShift容器平台的不同部分对新图像做出反应，创建新的[构建](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/builds_and_image_streams.html#builds)和[部署](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations)。

