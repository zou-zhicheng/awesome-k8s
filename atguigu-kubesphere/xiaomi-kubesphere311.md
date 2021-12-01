# 说明

在小米笔记本上通过虚拟机安装websphere v3.1.1

# 安装操作系统

centos7.9 最小化安装

调整资源大小

安装必要软件

```
yum -y update
yum -y install wget vim
```

关闭selinux

```bash
# 将 SELinux 设置为 disabled 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

关闭防火墙

```bash
systemctl stop firewalld
systemctl disable firewalld
```

查看并关闭swap分区

```bash
# 关闭swap
swapoff -a
cp /etc/sysctl.conf /etc/sysctl.conf.bak
echo "vm.swappiness=0" >> /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
cp /etc/fstab /etc/fstab.bak
sed -i 's$/dev/mapper/centos-swap$#/dev/mapper/centos-swap$g' /etc/fstab
# sed -ri 's/.*swap.*/#&/' /etc/fstab
# 查看swap结果
free -m
#               total        used        free      shared  buff/cache   available
# Mem:          24345         409       23786          11         149       23657
# Swap:             0           0           0
```

# 单节点部署kubesphere

[在 Linux 上以 All-in-One 模式安装 KubeSphere](https://kubesphere.com.cn/docs/quick-start/all-in-one-on-linux/)

## 安装依赖组件

```bash
yum -y install ebtables socat ipset conntrack
```

## 下载 KubeKey

```bash
cd /opt
export KKZONE=cn
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.1.1 sh -
chmod +x kk
```

## 开始安装

```bash
./kk create cluster --with-kubernetes v1.20.4 --with-kubesphere v3.1.1
```

安装成功

```
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://192.168.127.129:30880
Account: admin
Password: P@88w0rd -> Passw0rd

NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components
     are up and running.
  2. Please change the default password after login.

#####################################################
https://kubesphere.io             2021-11-29 22:54:06
#####################################################
INFO[22:54:16 CST] Installation is complete.

Please check the result using the command:

       kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

# 启用可插拔组件

## 应用商店

1. 以 `admin` 身份登录控制台，点击左上角的**平台管理**，选择**集群管理**。
2. 点击**自定义资源 CRD**，在搜索栏中输入 `clusterconfiguration`，点击结果查看其详细页面。
3. 在**资源列表**中，点击 `ks-installer` 右侧的 ![img](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-app-store/three-dots.png)，选择**编辑配置文件**。
4. 在该 YAML 文件中，搜寻到 `openpitrix`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**更新**，保存配置。

### 验证组件的安装

在您登录控制台后，如果您能看到页面左上角的**应用商店**以及其中的应用，则说明安装成功。

- 您可以在不登录控制台的情况下直接访问 `<节点 IP 地址>:30880/apps` 进入应用商店。

## DevOps 系统

1. 以 `admin` 身份登录控制台，点击左上角的**平台管理**，选择**集群管理**。

2. 点击**自定义资源 CRD**，在搜索栏中输入 `clusterconfiguration`，点击搜索结果查看其详细页面。
   
   信息
   
   自定义资源定义 (CRD) 允许用户在不增加额外 API 服务器的情况下创建一种新的资源类型，用户可以像使用其他 Kubernetes 原生对象一样使用这些自定义资源。

3. 在**资源列表**中，点击 `ks-installer` 右侧的 ![img](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-devops-system/three-dots.png)，选择**编辑配置文件**。

4. 在该 YAML 文件中，搜寻到 `devops`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**更新**，保存配置。

### 验证组件的安装

#### 在仪表盘中验证

进入 平台管理 -> 集群管理  -> 服务组件 -> devops，检查 **DevOps** 标签页中的所有组件都处于**健康**状态。

#### 使用kubectl验证组件的安装

执行以下命令来检查容器组的状态：

```shell
kubectl get pod -n kubesphere-devops-system
```

如果组件运行成功，输出结果如下：

```shell
NAME                          READY   STATUS    RESTARTS   AGE
devops-jenkins-5cbbfbb975-hjnll   1/1     Running   0          40m
s2ioperator-0                 1/1     Running   0          41m
```

## 日志系统

1. 以 `admin` 身份登录控制台。点击左上角的**平台管理**，选择**集群管理**。

2. 点击 **CRD**，在搜索栏中输入 `clusterconfiguration`。点击结果查看其详细页面。

3. 在**自定义资源**中，点击 `ks-installer` 右侧的 ![](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-logging-system/three-dots.png)，选择**编辑 YAML**。

4. 在该 YAML 文件中，搜寻到 `logging`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**确定**，保存配置。

### 验证组件的安装

#### 仪表盘中验证

进入**系统组件**，检查**日志**标签页中的所有组件都处于**健康**状态。

#### kubectl验证

执行以下命令来检查容器组的状态：

```shell
kubectl get pod -n kubesphere-logging-system
```

如果组件运行成功，输出结果如下：

```shell
NAME                                          READY   STATUS    RESTARTS   AGE
elasticsearch-logging-data-0                  1/1     Running   0          87m
elasticsearch-logging-data-1                  1/1     Running   0          85m
elasticsearch-logging-discovery-0             1/1     Running   0          87m
fluent-bit-bsw6p                              1/1     Running   0          40m
fluent-bit-smb65                              1/1     Running   0          40m
fluent-bit-zdz8b                              1/1     Running   0          40m
fluentbit-operator-9b69495b-bbx54             1/1     Running   0          40m
logsidecar-injector-deploy-667c6c9579-cs4t6   2/2     Running   0          38m
logsidecar-injector-deploy-667c6c9579-klnmf   2/2     Running   0          38m
```

## 事件系统

1. 以 `admin` 身份登录控制台。点击左上角的**平台管理**，选择**集群管理**。

2. 点击 **CRD**，在搜索栏中输入 `clusterconfiguration`。点击结果查看其详细页面。

3. 在**自定义资源**中，点击 `ks-installer` 右侧的 ![](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-events/three-dots.png)，选择**编辑 YAML**。

4. 在该 YAML 文件中，搜寻到 `events`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**确定**，保存配置。

### 验证组件的安装

#### 在仪表盘中验证

验证您可以使用右下角**工具箱**中的**资源事件查询**功能。

#### kubectl验证

执行以下命令来检查容器组的状态：

```shell
kubectl get pod -n kubesphere-logging-system
```

如果组件运行成功，输出结果如下：

```shell
NAME                                          READY   STATUS    RESTARTS   AGE
elasticsearch-logging-data-0                  1/1     Running   0          155m
elasticsearch-logging-data-1                  1/1     Running   0          154m
elasticsearch-logging-discovery-0             1/1     Running   0          155m
fluent-bit-bsw6p                              1/1     Running   0          108m
fluent-bit-smb65                              1/1     Running   0          108m
fluent-bit-zdz8b                              1/1     Running   0          108m
fluentbit-operator-9b69495b-bbx54             1/1     Running   0          109m
ks-events-exporter-5cb959c74b-gx4hw           2/2     Running   0          7m55s
ks-events-operator-7d46fcccc9-4mdzv           1/1     Running   0          8m
ks-events-ruler-8445457946-cl529              2/2     Running   0          7m55s
ks-events-ruler-8445457946-gzlm9              2/2     Running   0          7m55s
logsidecar-injector-deploy-667c6c9579-cs4t6   2/2     Running   0          106m
logsidecar-injector-deploy-667c6c9579-klnmf   2/2     Running   0          106m
```

## 告警系统

1. 使用 `admin` 用户登录控制台。点击左上角的**平台管理**，选择**集群管理**。

2. 点击 **CRD**，在搜索栏中输入 `clusterconfiguration`。点击结果查看其详细页面。

3. 在**自定义资源**中，点击 `ks-installer` 右侧的 ![](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-alerting/three-dots.png)，选择**编辑 YAML**。

4. 在该 YAML 文件中，搜寻到 `alerting`，将 `enabled` 的 `false` 更改为 `true`。完成后，点击右下角的**确定**，保存配置。

### 验证组件的安装

如果您在**集群管理**页面可以看到**告警消息**和**告警策略**，说明安装成功，因为安装组件之后才会显示这两部分。

## 审计日志

1. 以 `admin` 身份登录控制台。点击左上角的**平台管理**，选择**集群管理**。

2. 点击 **CRD**，在搜索栏中输入 `clusterconfiguration`，点击搜索结果查看其详细页面。

3. 在**自定义资源**中，点击 `ks-installer` 右侧的 ![](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-auditing-logs/three-dots.png)，选择**编辑 YAML**。

4. 在该 YAML 文件中，搜寻到 `auditing`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**确定**，保存配置。

### 验证组件的安装

#### 仪表盘中验证

验证您可以使用右下角**工具箱**中的**审计日志查询**功能。

#### kubectl验证

执行以下命令来检查容器组的状态：

```shell
kubectl get pod -n kubesphere-logging-system
```

如果组件运行成功，输出结果如下：

```shell
NAME                                                              READY   STATUS      RESTARTS   AGE
elasticsearch-logging-curator-elasticsearch-curator-159872n9g9g   0/1     Completed   0          2d10h
elasticsearch-logging-curator-elasticsearch-curator-159880tzb7x   0/1     Completed   0          34h
elasticsearch-logging-curator-elasticsearch-curator-1598898q8w7   0/1     Completed   0          10h
elasticsearch-logging-data-0                                      1/1     Running     1          2d20h
elasticsearch-logging-data-1                                      1/1     Running     1          2d20h
elasticsearch-logging-discovery-0                                 1/1     Running     1          2d20h
fluent-bit-6v5fs                                                  1/1     Running     1          2d20h
fluentbit-operator-5bf7687b88-44mhq                               1/1     Running     1          2d20h
kube-auditing-operator-7574bd6f96-p4jvv                           1/1     Running     1          2d20h
kube-auditing-webhook-deploy-6dfb46bb6c-hkhmx                     1/1     Running     1          2d20h
kube-auditing-webhook-deploy-6dfb46bb6c-jp77q                     1/1     Running     1          2d20h
```

## 服务网格

1. 以 `admin` 身份登录控制台。点击左上角的**平台管理**，选择**集群管理**。

2. 点击 **CRD**，在搜索栏中输入 `clusterconfiguration`。点击结果查看其详细页面。

3. 在**自定义资源**中，点击 `ks-installer` 右侧的 ![](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-service-mesh/three-dots.png)，选择**编辑 YAML**。

4. 在该 YAML 文件中，搜寻到 `servicemesh`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**确定**，保存配置。

### 验证组件的安装

#### 仪表盘中验证

进入**系统组件**，检查 **Istio** 标签页中的所有组件都处于**健康**状态。

#### kubectl验证

执行以下命令来检查容器组的状态：

```shell
kubectl get pod -n istio-system
```

如果组件运行成功，输出结果可能如下：

```shell
NAME                                    READY   STATUS    RESTARTS   AGE
istio-ingressgateway-78dbc5fbfd-f4cwt   1/1     Running   0          9m5s
istiod-1-6-10-7db56f875b-mbj5p          1/1     Running   0          10m
jaeger-collector-76bf54b467-k8blr       1/1     Running   0          6m48s
jaeger-operator-7559f9d455-89hqm        1/1     Running   0          7m
jaeger-query-b478c5655-4lzrn            2/2     Running   0          6m48s
kiali-f9f7d6f9f-gfsfl                   1/1     Running   0          4m1s
kiali-operator-7d5dc9d766-qpkb6         1/1     Running   0          6m53s
```

## 网络策略

### 验证组件的安装

#### 仪表盘中验证

#### kubectl验证

## Metrics Server

### 验证组件的安装

#### 仪表盘中验证

#### kubectl验证

## 服务拓扑图

### 验证组件的安装

#### 仪表盘中验证

#### kubectl验证

## 容器组 IP 池

### 验证组件的安装

#### 仪表盘中验证

#### kubectl验证

# 卸载可插拔组件



# 创建企业空间、项目、帐户和角色

[创建企业空间、项目、帐户和角色 (kubesphere.io)](https://v3-1.docs.kubesphere.io/zh/docs/quick-start/create-workspace-and-project/)

KubeSphere 的多租户系统分**三个**层级，即集群、企业空间和项目。KubeSphere 中的项目等同于 Kubernetes 的[命名空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/)。

### 步骤 1：创建帐户

一开始，系统默认只有一个帐户 `admin`，具有 `platform-admin` 角色。在本步骤中，您将创建一个帐户 `user-manager`，然后使用 `user-manager` 创建新帐户。

创建如下四个新帐户，这些帐户将在其他的教程中使用。

| 帐户                | 角色                   | 描述                                                                                 |
| ----------------- | -------------------- | ---------------------------------------------------------------------------------- |
| `ws-manager`      | `workspaces-manager` | 创建和管理所有企业空间。                                                                       |
| `ws-admin`        | `platform-regular`   | 管理指定企业空间中的所有资源（在此示例中，此帐户用于邀请新成员加入该企业空间）。                                           |
| `project-admin`   | `platform-regular`   | 创建和管理项目以及 DevOps 工程，并邀请新成员加入项目。                                                    |
| `project-regular` | `platform-regular`   | `project-regular` 将由 `project-admin` 邀请至项目或 DevOps 工程。该帐户将用于在指定项目中创建工作负载、流水线和其他资源。 |

### 步骤 2：创建企业空间

在本步骤中，您需要使用上一个步骤中创建的帐户 `ws-manager` 创建一个企业空间。作为管理项目、DevOps 工程和组织成员的基本逻辑单元，**企业空间是 KubeSphere 多租户系统的基础。**



### 步骤 3：创建项目

KubeSphere 中的项目与 Kubernetes 中的命名空间相同，为资源提供了虚拟隔离。

### 步骤 4：创建角色

### 步骤 5：创建 DevOps 工程（可选）

# 部署并访问 Bookinfo

[部署并访问 Bookinfo (kubesphere.io)](https://v3-1.docs.kubesphere.io/zh/docs/quick-start/deploy-bookinfo-to-k8s/)

[Istio](https://istio.io/) 为微服务提供了强大的流量管理功能。

# 创建并部署 WordPress

[创建并部署 WordPress (kubesphere.io)](https://v3-1.docs.kubesphere.io/zh/docs/quick-start/wordpress-deployment/)

# devops用户指南

[DevOps 用户指南 (kubesphere.io)](https://v3-1.docs.kubesphere.io/zh/docs/devops-user-guide/)

## 使用 Jenkinsfile 创建流水线

Jenkinsfile 将整个工作流存储为代码，因此它是代码审查和流水线迭代过程的基础。

 [Docker Hub](https://hub.docker.com/) 帐户 

```
zouzhicheng / james / zouzhicheng@foxmail.com
```

[GitHub](https://github.com/) 帐户

```
zouzhicheng / james / zouzhicheng@foxmail.com
```
