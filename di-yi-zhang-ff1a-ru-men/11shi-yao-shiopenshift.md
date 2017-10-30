# 使用Web控制台创建和构建镜像

## 概述 {#developers-console-before-you-begin}

该入门指南将引导您在OpenShift Container Platform上采取的最简单的方法创建示例项目。虽然启动镜像有多种不同的方法，但本节侧重于最快速和最简单的方法。

如果这是您阅读的文档的第一部分，并且您不熟悉OpenShift容器平台版本3（v3）的核心概念，则可能需要首先阅读[新内容](https://docs.openshift.com/container-platform/3.6/release_notes/index.html#release-notes-index)。此版本的OpenShift容器平台与版本2（v2）有着显著不同。

OpenShift Container Platform3 为开发人员提供了一套[语言](https://docs.openshift.com/container-platform/3.5/using_images/s2i_images/index.html#using-images-s2i-images-index)和[数据库](https://docs.openshift.com/container-platform/3.5/using_images/db_images/index.html#using-images-db-images-index)，可让您启动应用程序开发。

| 语言 | 实现和教程 |
| :--- | :--- |
| Ruby | [Rails](https://github.com/openshift/rails-ex) |
| Python | [Django](https://github.com/openshift/django-ex) |
| Node.js | [Node.js](https://github.com/openshift/nodejs-ex) |
| PHP | [CakePHP](https://github.com/openshift/cakephp-ex) |
| Perl | [Dancer](https://github.com/openshift/dancer-ex) |
| Java |  |

OpenShift Container Platform提供的其他镜像包括：

* [MySQL](https://github.com/openshift/mysql)

* [MongoDB](https://github.com/openshift/mongodb)

* [PostgreSQL](https://github.com/openshift/postgresql)

* [Jenkins](https://github.com/openshift/jenkins)

## 在示例项目开始之前的准备 {#developers-console-before-you-begin}

* 您必须能够访问OpenShift Container Platform的正在运行的实例。如果您没有访问权限，请与您的集群管理员联系。

* 您的实例必须由集群管理员使用[即时应用模板](https://docs.openshift.com/container-platform/3.5/dev_guide/templates.html#using-the-instantapp-templates)和[构建器映像](https://docs.openshift.com/container-platform/3.5/using_images/s2i_images/index.html#using-images-s2i-images-index)进行预配置。如果它们不可用，请联系集群管理员“[下载默认映像流和模板”](https://docs.openshift.com/container-platform/3.5/install_config/imagestreams_templates.html#install-config-imagestreams-templates)主题。

* 您必须[下载并安装](https://docs.openshift.com/container-platform/3.5/cli_reference/get_started_cli.html#cli-reference-get-started-cli)OpenShift Container Platform CLI。

## 建立Ruby分支仓库 {#developers-console-video}

1.登录到GitHub，访问[Ruby示例](https://github.com/openshift/ruby-ex)页面。

> ![](/assets/提示3%.png)本节采用Ruby示例，您也可以使用[OpenShift Container Platform GitHub项目中提供的](https://www.gitbook.com/book/fanxinwei/openshift_3-6/edit#)任何语言示例进行实验

1. 建立你的分支仓库

2. 复制 clone URL在你的分支仓库中。

3. 将仓库克隆到本地计算机。

## 参考视频 {#developers-console-video}

[https://access.redhat.com/videos/2480801](https://access.redhat.com/videos/2480801)

## 创建一个项目 {#developers-console-creating-a-project}

要创建一个应用程序，您必须先创建一个新项目，然后选择一个InstantApp模板。之后，OpenShift Container Platform开始构建并创建一个新的部署。

1. 访问浏览器中的OpenShift Container Platform Web控制台。Web控制台使用自签名证书，因此如果出现提示，请继续进入浏览器警告。

2. 使用管理员向您推荐的用户名和密码进行登录。

3. 创建新项目，请单击**新建项目**。

4. 输入新项目的唯一名称，显示名称和说明。

5. 单击**创建**。

加载Web控制台的欢迎画面。

## 创建一个应用程序 {#developers-console-creating-an-application}

“选择镜像”或“模板”页面可让您从公开可访问的git存储库或模板创建选项：

1. 如果创建新项目没有自动将您重定向到“选择图像”或“模板”页面，则可能需要单击“Add to project**”。**
2. 单击Browe，然后从下拉列表中选择**ruby**。

3. 点击ruby:latest构建镜像。

4. 键入一个**名称**为您的应用程序，并指定**Git仓库URL：**[`https://github.com/<your_github_username>/ruby-ex.git`](https://github.com/<your_github_username>/ruby-ex.git)。

5. （可选）单击**Show advanced routing, build, and deployment options**，但默认情况下，此示例应用程序会自动创建路由，Webhook触发器和构建更改触发器。

6. 单击**创建。**

> ![](/assets/提示3%.png)创建后，可以通过单击**Brose、Builds**，然后单击**Actions**，**Edit**或**Edit YAML**，可从Web控制台修改其中一些设置。

创建应用程序可能需要一些时间。您可以在Web控制台的“Overview”页面上查看正在创建的新资源，并观察构建和部署的进度。

当Ruby pod被创建时，它的状态显示为“挂起”。Ruby pod然后启动并显示其新分配的IP地址。当Ruby pod显示为“running”时，构建完成。

## 查看正在运行的应用程序 {#developers-console-view-app}

如果您的DNS配置正确，则可以使用Web浏览器访问新的应用程序。

查看您的新应用程序：

1. 在Web控制台中，查看“overview”页面以确定应用程序的Web地址。例如，在**SERVICE：RUBY-EX下，**您应该看到类似于如下：`ruby-ex-my-test.example.openshiftapps.com`。

2. 访问您的新应用程序的网址。

## 配置自动build {#developers-console-configure-auto-builds}

您从[OpenShift Container Platform GitHub仓库](https://github.com/openshift/ruby-ex)中下载了该应用程序的源代码。因此，只要将代码更改推送到分支仓库，就可以使用webhook自动触发应用程序的重新构建。

为您的应用程序设置一个webhook：

1. 从Web控制台导航到您包含应用程序的项目。

2. 单击**Browse**，然后单击**Builds**。

3. 单击您的构建名称，然后单击**Configuration**选项。

4. 点击**GitHub webhook URL**![](data:image/jpg;base64,/9j/4AAQSkZJRgABAQEAYABgAAD//gATQ3JlYXRlZCB3aXRoIEdJTVD/2wBDAAMCAgMCAgMDAwMEAwMEBQgFBQQEBQoHBwYIDAoMDAsKCwsNDhIQDQ4RDgsLEBYQERMUFRUVDA8XGBYUGBIUFRT/2wBDAQMEBAUEBQkFBQkUDQsNFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBQUFBT/wgARCAAbAB0DAREAAhEBAxEB/8QAGgABAAIDAQAAAAAAAAAAAAAABQYIAAEDBP/EABQBAQAAAAAAAAAAAAAAAAAAAAD/2gAMAwEAAhADEAAAAbBBY2RgkBU8sSaHSIjwSCnMw95//8QAHhAAAQQCAwEAAAAAAAAAAAAABAACAwUGFQERJBT/2gAIAQEAAQUCEHgkF2lEmDiSMvehUE7xYvzPZ2OOkEDD5I5BO8bcTDCIgFJ+7InKSyJgfty1ty0K/myX/8QAFBEBAAAAAAAAAAAAAAAAAAAAQP/aAAgBAwEBPwEH/8QAFBEBAAAAAAAAAAAAAAAAAAAAQP/aAAgBAgEBPwEH/8QALhAAAgEDAgIHCQEAAAAAAAAAAQIDAAQREjEFEyEzUoKiscEUIiMyNEFhcnOT/9oACAEBAAY/AoWaGNmKAklR09FfVcP/ANEpXSGFlYZDBRg1DyByc5zy+jO1QfzXyq34W97dRWcocGOKUgY0k7bVbcPvIGhljh9xjpwwXAOxO2RVv3vSoP0XyqG54ci2lzG2dT6pARgjGNX5pLi4uIpNEbRqsUJTcqe0ezVv3vSmjSTCIdIGBtXW+EV1vhFN7T8TR8v2r//EACAQAQABBAICAwAAAAAAAAAAAAERACExQWHwENFRcZH/2gAIAQEAAT8hUsacpuVpJhN67okHHcHCNYM07FZMfddk0q5iI0iMoLm22hylkIbnZeYmcF/BOnaU/CQmo1McrOqlGc0BCrPj++ISl4lUCxqu/wBNd/pq5mLgnOI+Cv/aAAwDAQACAAMAAAAQAgEAkAEk/8QAFBEBAAAAAAAAAAAAAAAAAAAAQP/aAAgBAwEBPxAH/8QAFBEBAAAAAAAAAAAAAAAAAAAAQP/aAAgBAgEBPxAH/8QAHhABAQADAAIDAQAAAAAAAAAAAREAITFBcRBRYYH/2gAIAQEAAT8QNWtxooFVVVe4mBGI1MCwYEtQCIiImkc9eePUGypeV+8ntwNBUhoFWhMLE8jcjWjWFQjglBDQBTrGezIKf3Ud2OyEET0UY3ZA5ruiIAdVdZbvHo4w5UKpYBtb822/yah3/qvmvM//2Q== "复制")旁边的**URL**来复制你的webhook的URL。

5. 浏览到GitHub上的分支存储库，然后单击**设置**。

6. 单击**Webhooks＆Services**。

7. 单击**add** **webhook**。

8. 将您的Webhook URL粘贴到**Payload URL**字段中。

9. 单击**add** **webhook**以保存。

GitHub之后将尝试向您的OpenShift Container Platform服务器发送ping包，以确保连接成功。如果您看到一个绿色的勾号出现在您的webhook URL旁边，那么它被正确配置。将鼠标悬停在复选标记上，以查看上次上传的状态。

当将代码更改推送到您的分支存储库时，您的应用程序将自动构建。

## 代码更改 {#developers-console-write-code-change}

在本地修改后将更改推送到您的应用程序：

1. 在本地机器上，使用文本编辑器更改示例应用程序的文件_**ruby-ex / config.ru**_

2. 修改从应用程序中可以直接看到的代码。例如：在第229行，将标题`Welcome to your Ruby application on OpenShift`更改为`This is my Awesome OpenShift Application`，然后保存更改。

3. 提交git中的更改，并将更改推到您的分支。

   如果您的Webhook配置正确，您的应用程序将根据您的更改立即重建。重建成功后，使用之前创建的路由查看更新的应用程序。

现在，您需要做的只是推送代码更新，OpenShift Container Platform处理其余部分。

### 手动重建镜像 {#developers-console-manually-rebuild-images}

手动重建镜像很有用，当Webhook没有开启、或者build失败并且您不想在重新启动构建之前更改代码。根据您最近对您的分支库进行的更改，手动重建镜像如下：

1. 单击**Browse**选项，然后单击**Builds**。

2. 找到您的构建，然后单击**Start Build**。



