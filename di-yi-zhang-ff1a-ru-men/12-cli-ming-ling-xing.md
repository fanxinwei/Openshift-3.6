## 创建一个项目 {#developers-cli-creating-a-project}

要创建应用程序，必须创建一个新项目并指定源的位置。OpenShift Container Platform根据该位置构建新的部署。

1. 从CLI登录OpenShift容器平台：

   * 使用用户名和密码：

     ```
     $ oc login -u=
     <
     username
     >
      -p=
     <
     password
     >
      --server=
     <
     your-openshift-server
     >
      --insecure-skip-tls-verify
     ```

   * 使用oauth令牌：

     ```
     $ oc login 
     <
     https://api.your-openshift-server.com
     >
      --token=
     <
     tokenID
     >
     ```

2. 创建一个新项目：

   ```
   $ oc new-project 
   <
   projectname
   >
    --description="
   <
   description
   >
   " --display-name="
   <
   display_name
   >
   "
   ```

创建新项目后，您将自动切换到新的项目命名空间。

## 创建应用程序 {#developers-cli-creating-an-application}

要从您的分支库中的代码创建一个新的应用程序：

1. 通过指定代码的来源创建应用程序：

   ```
   $ oc new-app openshift/ruby-20-centos7~https://github.com/
   <
   your_github_username
   >
   /ruby-ex
   ```

   OpenShift Container Platform找到匹配的构建器映像（在这种情况下为**ruby-20-centos7**），然后为应用程序（映像流，构建配置，部署配置，服务）创建资源。它也安排构建。

2. 跟踪构建的进度：

   ```
   $ oc logs -f bc / ruby​​-ex
   ```

3. 一旦构建完成并且生成的映像已成功推送到注册表，请检查应用程序的状态：

   ```
   $ oc状态
   ```

   或者您可以从Web控制台查看构建。

创建应用程序可能需要一些时间。您可以在Web控制台的“概述”页面上查看正在创建的新资源，并观察构建和部署的进度。您还可以使用该`oc get pods`命令检查pod何时启动并运行，或者`oc get builds`查看构建统计信息的命令。

当Ruby pod被创建时，它的状态显示为挂起。Ruby pod然后启动并显示其新分配的IP地址。当Ruby pod运行时，构建完成。

该`oc status`命令告诉您服务正在运行的IP地址;它部署到的默认端口是8080。

## 创建路由 {#developers-cli-create-route}

OpenShift Container Platform路径以主机名公开一个服务，以便外部客户端可以通过名称访问它。创建到您的新应用程序的路由：

1. 提供以下服务`ruby-ex`：

   ```
   $ oc expose service ruby-ex
   ```

2. 查看您的新路线：

   ```
   $ oc get route
   ```

3. 复制路线位置，例如`ruby-ex-my-test.example.openshiftapps.com`。

## 查看运行的应用程序 {#developers-cli-view-running-app}

要查看您的新应用程序，请将您复制的路线位置（在上一节中）粘贴到Web浏览器的地址栏中，然后按Enter键。

示例`ruby-ex`应用程序是一个简单的欢迎屏幕，并包含有关如何部署代码更改，管理应用程序和其他开发资源的详细信息。

接下来，通过GitHub webhook触发器配置自动化构建，以便您的分支存储库中的代码更改导致您的应用程序重建。

## 配置自动构建 {#configuring-automated-builds}

您从[OpenShift Container Platform GitHub存储库中分析](https://github.com/openshift/ruby-ex)了此应用程序的源代码。因此，只要将代码更改推送到分支存储库，就可以使用webhook自动触发应用程序的重建。

为您的应用程序设置一个webhook：

1. 查看触发器部分`BuildConfig`以验证是否存在GitHub webhook触发器：

   ```
   $ oc edit bc / ruby​​-ex
   ```

   你应该看到类似如下：

   ```
   triggers
   - github:
       secret: Q1tGY0i9f1ZFihQbX07S
       type: GitHubb
   ```

   秘密确保只有您和您的存储库才能触发构建。

2. 运行以下命令来显示与您的关联的webhook URL`BuildConfig`：

   ```
   $ oc describe bc ruby-ex
   ```

3. 复制上述命令输出的GitHub webhook有效内容URL。

4. 在GitHub上导航到您的分支存储库，然后单击**Settings**。

5. 单击**Webhooks＆Services**。

6. 单击**添加webhook**。

7. 将您的webhook URL粘贴到**有效载荷URL**字段中。

8. 单击**添加webhook**以保存。

GitHub现在尝试向您的OpenShift Container Platform服务器发送ping有效内容，以确保通信成功。如果您看到一个绿色的勾号出现在您的webhook URL旁边，那么它被正确配置。将鼠标悬停在复选标记上，以查看上次投递的状态。

下一次将代码更改推送到您的分支存储库时，您的应用程序将自动重建。

## 更改代码 {#developers-cli-writing-a-code-change}

要在本地工作，然后将更改推送到您的应用程序：

1. 在本地机器上，使用文本编辑器更改示例应用程序的文件来源_**ruby-ex / config.ru**_

2. 进行代码更改，从应用程序中可以看到。例如：在第229行，将标题更改`Welcome to your Ruby application on OpenShift`为`This is my Awesome OpenShift Application`，然后保存更改。

3. 提交git中的更改，并将更改推到您的分支仓库。

   如果您的Webhook配置正确，您的应用程序将根据您的更改立即重建。重建成功后，使用之前创建的路由查看更新的应用程序。

现在，您需要做的只是推送代码更新，OpenShift Container Platform处理其余部分。

### 手动重建图像 {#developers-cli-manually-rebuilding-images}

如果您的Webhook不工作，或者构建失败，并且您不想在重新启动构建之前更改代码，则可能会发现手动重建映像很有用。根据您最近对您的分支库进行的更改，手动重建映像：

```
$ oc start-build ruby​​-ex
```

## 故障排除 {#developers-cli-troubleshooting}

**改变项目**

虽然`oc new-project`命令会自动将您当前的项目设置为您刚创建的项目，但您可以随时通过运行以下方式更改项目：

```
$ oc project 
<
project-name
>
```

查看项目列表：

```
$ oc get projects
```

**手动触发构建**

如果构建未自动启动，则启动构建并流式传输日志：

```
$ oc start-build ruby​​-ex -follow
```

或者，不要`--follow`在上述命令中包含，而是在触发构建之后发出以下命令，`n`要跟踪的构建的编号在哪里：

```
$ oc logs -f build / ruby​​-ex-n
```



