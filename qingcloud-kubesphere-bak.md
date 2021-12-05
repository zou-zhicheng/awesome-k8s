# 安装

```shell
export KKZONE=cn
curl -sfL https://get-kk.kubesphere.io | VERSION=v1.2.0 sh -
chmod +x kk
yum -y install conntrack
./kk create cluster --with-kubernetes v1.21.5 --with-kubesphere v3.2.0
```

安装结果

```shell
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://172.16.0.2:30880
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
https://kubesphere.io             2021-12-01 17:07:58
#####################################################
INFO[17:08:06 CST] Installation is complete.

Please check the result using the command:

       kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

## 主页

http://139.198.43.205:30880

# 启用可插拔组件

[启用可插拔组件 (kubesphere.com.cn)](https://kubesphere.com.cn/docs/pluggable-components/)

## 应用商店

## devops

### 将 SonarQube 集成到流水线

[将 SonarQube 集成到流水线 (kubesphere.com.cn)](https://kubesphere.com.cn/docs/devops-user-guide/how-to-integrate/sonarqube/)

```shell
helm upgrade --install sonarqube sonarqube --repo https://charts.kubesphere.io/main -n kubesphere-devops-system  --create-namespace --set service.type=NodePort
```

执行结果

```shell
Release "sonarqube" does not exist. Installing it now.
NAME: sonarqube
LAST DEPLOYED: Sun Dec  5 22:25:47 2021
NAMESPACE: kubesphere-devops-system
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace kubesphere-devops-system -o jsonpath="{.spec.ports[0].nodePort}" services sonarqube-sonarqube)
  export NODE_IP=$(kubectl get nodes --namespace kubesphere-devops-system -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP
```

端点

```shell
http://172.16.0.2:31619
http://139.198.43.205:31619
```

在 青云控制台 -> 安全 -> 安全组 中添加端口31619的访问策略

检查

```shell
$ kubectl get pod -n kubesphere-devops-system
NAME                                       READY   STATUS    RESTARTS   AGE
devops-jenkins-68b8949bb-7zwg4                 1/1     Running   0          84m
s2ioperator-0                              1/1     Running   1          84m
sonarqube-postgresql-0                     1/1     Running   0          5m31s
sonarqube-sonarqube-bb595d88b-97594        1/1     Running   2          5m31s

```



## 日志系统

## 事件系统

## 告警系统

## 审计日志

## 服务网格

## 网络策略

## metric server

## 服务拓扑图

## 容器组 IP 池

## KubeEdge



# 快速入门

## 企业空间管理和用户指南

[创建企业空间、项目、用户和平台角色 (kubesphere.com.cn)](https://kubesphere.com.cn/docs/quick-start/create-workspace-and-project/)

## 部署并访问 Bookinfo

[部署并访问 Bookinfo (kubesphere.com.cn)](https://kubesphere.com.cn/docs/quick-start/deploy-bookinfo-to-k8s/)

## 部署并访问 Bookinfo

[创建并部署 WordPress (kubesphere.com.cn)](https://kubesphere.com.cn/docs/quick-start/wordpress-deployment/)

# DevOps 用户指南

[DevOps 用户指南 (kubesphere.com.cn)](https://kubesphere.com.cn/docs/devops-user-guide/)

## docker hub

zouzhicheng / james / [zouzhicheng@foxmail.com](mailto:zouzhicheng@foxmail.com)

youzhibicheng / zzc /[youzhibicheng@163.com](mailto:youzhibicheng@163.com)

## github

zouzhicheng / james / [zouzhicheng@foxmail.com](mailto:zouzhicheng@foxmail.com)

代码库

[GitHub - zou-zhicheng/devops-java-sample: SpringBoot demo for DevOps on KubeSphere](https://github.com/zou-zhicheng/devops-java-sample.git)








