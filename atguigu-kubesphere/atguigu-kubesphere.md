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
echo "vm.swappiness=0" >> /etc/sysctl.conf
sysctl -p /etc/sysctl.conf
sed -i 's$/dev/mapper/centos-swap$#/dev/mapper/centos-swap$g' /etc/fstab
# sed -ri 's/.*swap.*/#&/' /etc/fstab
# 查看swap结果
# free -m
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

Console: http://172.31.30.131:30880
Account: admin
Password: P@88w0rd  -> Passw0rd

NOTES：
  1. After you log into the console, please check the
     monitoring status of service components in
     "Cluster Management". If any service is not
     ready, please wait patiently until all components
     are up and running.
  2. Please change the default password after login.

#####################################################
https://kubesphere.io             2021-10-24 16:43:11
#####################################################
INFO[16:43:23 CST] Installation is complete.

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

## DevOps 系统

1. 以 `admin` 身份登录控制台，点击左上角的**平台管理**，选择**集群管理**。

2. 点击**自定义资源 CRD**，在搜索栏中输入 `clusterconfiguration`，点击搜索结果查看其详细页面。

   信息

   自定义资源定义 (CRD) 允许用户在不增加额外 API 服务器的情况下创建一种新的资源类型，用户可以像使用其他 Kubernetes 原生对象一样使用这些自定义资源。

3. 在**资源列表**中，点击 `ks-installer` 右侧的 ![img](https://kubesphere.com.cn/images/docs/zh-cn/enable-pluggable-components/kubesphere-devops-system/three-dots.png)，选择**编辑配置文件**。

4. 在该 YAML 文件中，搜寻到 `devops`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的**更新**，保存配置。



## 日志系统



## 事件系统

## 告警系统

## 审计日志

## 服务网格

## 网络策略

## Metrics Server

## 服务拓扑图

## 容器组 IP 池

