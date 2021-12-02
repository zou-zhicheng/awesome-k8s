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

# 启用可插拔组件

[启用可插拔组件 (kubesphere.io)](https://kubesphere.io/zh/docs/pluggable-components/overview/)

# 创建企业空间、项目、用户和平台角色

[创建企业空间、项目、用户和平台角色 (kubesphere.io)](https://kubesphere.io/zh/docs/quick-start/create-workspace-and-project/)

# 部署并访问 Bookinfo

[部署并访问 Bookinfo (kubesphere.io)](https://kubesphere.io/zh/docs/quick-start/deploy-bookinfo-to-k8s/)

# 创建并部署 WordPress

[创建并部署 WordPress (kubesphere.io)](https://kubesphere.io/zh/docs/quick-start/wordpress-deployment/)

# DevOps 用户指南

[DevOps 用户指南 (kubesphere.io)](https://kubesphere.io/zh/docs/devops-user-guide/)

# 项目用户指南

[项目用户指南 (kubesphere.io)](https://kubesphere.io/zh/docs/project-user-guide/)



## 