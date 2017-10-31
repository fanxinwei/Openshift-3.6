# Web控制台

## 概述 {#overview}

OpenShift Container Platform Web控制台是可从Web浏览器访问的用户界面。开发人员可以使用Web控制台进行显示、浏览和管理[项目](https://docs.openshift.com/container-platform/3.6/architecture/core_concepts/projects_and_users.html#projects)的内容。

> ![](/assets/提示3%.png) 为获得最佳体验，请使用支持[WebSockets](http://caniuse.com/#feat=websockets)的Web浏览器

## CLI下载 {#web-console-cli-downloads}

您可以从Web控制台的“**命令行工具”**页面下载并解压CLI，以在Linux，MacOSX和Windows客户端上使用。

## 项目概述 {#project-overviews}

[登录](https://docs.openshift.com/container-platform/3.6/dev_guide/authentication.html#dev-guide-authentication)之后，Web控制台为开发者提供当前选择的[项目](https://docs.openshift.com/container-platform/3.6/dev_guide/projects.html#dev-guide-projects)情况：



图2. Web Console项目情况

【1】允许您在有权访问的[项目之间](https://docs.openshift.com/container-platform/3.6/dev_guide/projects.html#view-projects)[切换](https://docs.openshift.com/container-platform/3.6/dev_guide/projects.html#view-projects)。

【2】[使用仓库](https://docs.openshift.com/container-platform/3.6/dev_guide/application_lifecycle/new_app.html#using-the-web-console-na)或[使用模板](https://docs.openshift.com/container-platform/3.6/dev_guide/templates.html#creating-from-templates-using-the-web-console)创建新的应用程序。

【3】“Overview**”**选项：（当前示例图），采用高级视图方式可视化项目的内容。

【4】“Application”选项：浏览，并可对deployments，pods，services和routes执行操作。

【5】"Build"选项：浏览，并可对builds和image stream（镜像流）执行操作

【6】“Resource”选项：查看您当前的配额消耗和其他资源。

【7】“Storage”选项：查看持久性卷声明并为应用程序请求存储。

【8】”Monitoring“选项：查看builds，pod和deployments的日志，以及项目中所有对象的事件通知。

