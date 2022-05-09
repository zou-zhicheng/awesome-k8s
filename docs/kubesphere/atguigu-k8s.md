# 说明

尚硅谷《云原生Java架构师的第一课K8s+Docker+KubeSphere+DevOps》

# 架构

![k8s-architecture](images\k8s-architecture.png)

kube-proxy

[kube-proxy](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kube-proxy/) 是集群中每个节点上运行的网络代理， 实现 Kubernetes [服务（Service）](https://kubernetes.io/zh/docs/concepts/services-networking/service/) 概念的一部分。

# 使用kubeadm安装kubernetes集群

## 网络说明

不要选择如下网段

```
192.168.0.0/16
172.16.0.0/16
172.17.0.0/16
```

建议选择

```
172.31.0.0/16
```

在vmware上是这样配置的 编辑->虚拟网络编辑器->NAT模式

![vmware](images\vmware.PNG)

## 安装Docker

```bash
# 配置yum源
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装docker
yum install -y docker-ce-20.10.7 docker-ce-cli-20.10.7  containerd.io-1.4.6

# 启动
systemctl enable docker --now

# 配置加速
# 登录 阿里云 -> 产品 -> 容器与中间件 -> 容器镜像服务ACR -> 管理控制台 -> 镜像工具 -> 镜像加速器
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://4wpnbb5u.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```



## 安装kubeadm

### 1、基础环境

```bash
#各个机器设置自己的域名
hostnamectl set-hostname xxxx

# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
# 最好设置成disable
sudo setenforce 0
# sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config

#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab

#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### 2、安装kubelet、kubeadm、kubectl

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF


sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes

sudo systemctl enable --now kubelet
```

## 使用kubeadm引导集群

### 1、下载各个机器需要的镜像

```bash
# 在所有的节点上运行
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF
   
chmod +x ./images.sh && ./images.sh
```

### 2、初始化主节点

```bash
# 所有机器添加master域名映射，以下需要修改为自己的
echo "172.31.30.128  cluster-endpoint" >> /etc/hosts
echo "172.31.30.128  k8s-master" >> /etc/hosts
echo "172.31.30.129  k8s-node1" >> /etc/hosts
echo "172.31.30.130  k8s-node2" >> /etc/hosts

# 主节点初始化
# 只在主节点运行
# 如果要修改相关网络，确保所有网络范围不重叠
kubeadm init \
--apiserver-advertise-address=172.31.30.128 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
```

得到最终结果

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

# 部署网络插件的提示, 稍后再使用
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join cluster-endpoint:6443 --token rhkqfe.qc7v4pcbosl8g5p6 \
    --discovery-token-ca-cert-hash sha256:1d980074c333dc27e5d8da4158412e5719388cb522819021bbc39660b32ad608 \
    --control-plane

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join cluster-endpoint:6443 --token rhkqfe.qc7v4pcbosl8g5p6 \
    --discovery-token-ca-cert-hash sha256:1d980074c333dc27e5d8da4158412e5719388cb522819021bbc39660b32ad608
```

执行上面提示中的命令

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

查看集群相关信息

```bash
# 查看集群所有节点
kubectl get nodes

# 根据配置文件，给集群创建资源
kubectl apply -f xxxx.yaml

# 查看集群部署了哪些应用？
docker ps   ===   kubectl get pods -A
# 运行中的应用在docker里面叫容器，在k8s里面叫Pod
kubectl get pods -A
```

### 3、部署网络插件

之前的提示

```bash
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/
```

目前使用calico网络插件

```bash
# 只在master运行
curl https://docs.projectcalico.org/manifests/calico.yaml -O

kubectl apply -f calico.yaml
```

### 4、加入node节点

之前的提示

```bash
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join cluster-endpoint:6443 --token rhkqfe.qc7v4pcbosl8g5p6 \
    --discovery-token-ca-cert-hash sha256:1d980074c333dc27e5d8da4158412e5719388cb522819021bbc39660b32ad608
```

直接在node上执行

```bash
# 默认的，这个命令是24小时有效
kubeadm join cluster-endpoint:6443 --token rhkqfe.qc7v4pcbosl8g5p6 \
    --discovery-token-ca-cert-hash sha256:1d980074c333dc27e5d8da4158412e5719388cb522819021bbc39660b32ad608
    
# 生成新令牌
# 一定要在master执行
kubeadm token create --print-join-command

# 在控制平面(master)上查看结果
# 验证集群节点状态
kubectl get nodes
```

#### 错误

执行出错

```bash
# kubeadm join cluster-endpoint:6443 --token rhkqfe.qc7v4pcbosl8g5p6 \
>     --discovery-token-ca-cert-hash sha256:1d980074c333dc27e5d8da4158412e5719388cb522819021bbc39660b32ad608
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
        [WARNING Hostname]: hostname "k8s-node2" could not be reached
        [WARNING Hostname]: hostname "k8s-node2": lookup k8s-node2 on 172.31.30.2:53: no such host
error execution phase preflight: couldn't validate the identity of the API Server: Get "https://cluster-endpoint:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s": dial tcp 172.31.30.128:6443: connect: no route to host
To see the stack trace of this error execute with --v=5 or higher
```

解决方案

```bash
# 1. 添加/etc/hosts
echo "172.31.30.128  k8s-master" >> /etc/hosts
echo "172.31.30.129  k8s-node1" >> /etc/hosts
echo "172.31.30.130  k8s-node2" >> /etc/hosts

# 2. 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld
```

#### 结果

命令运行之后的结果如下

```bash
# kubeadm join cluster-endpoint:6443 --token rhkqfe.qc7v4pcbosl8g5p6 \
>     --discovery-token-ca-cert-hash sha256:1d980074c333dc27e5d8da4158412e5719388cb522819021bbc39660b32ad608
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

执行查看命令结果如下

```bash
# 在主节点运行
# kubectl get nodes
NAME         STATUS   ROLES                  AGE     VERSION
k8s-master   Ready    control-plane,master   7h46m   v1.20.9
k8s-node1    Ready    <none>                 2m40s   v1.20.9
k8s-node2    Ready    <none>                 43s     v1.20.9
```

#### 关机自恢复

关机之后, 会自动恢复

## 部署dashboard

```bash
# 所有命令都需要在master节点执行
# kubernetes官方提供的可视化界面
# https://github.com/kubernetes/dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

# 设置访问端口
# 注意: 在修改文件中将 type: ClusterIP 改为 type: NodePort
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

# 查看dashboard端口
# kubectl get svc -A |grep kubernetes-dashboard
kubernetes-dashboard   dashboard-metrics-scraper   ClusterIP   10.96.112.56    <none>        8000/TCP                 7m2s
kubernetes-dashboard   kubernetes-dashboard        NodePort    10.96.105.167   <none>        443:30459/TCP            7m2s


# 在云计算环境下, 找到端口，在安全组放行

# 访问
# https://集群任意IP:端口      
https://172.31.30.128:30459
```

登录的时候需要身份验证

### 创建账号访问

创建访问账号，准备一个yaml文件；

```
vim dashboard-user.yaml
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

运行

```bash
kubectl apply -f dashboard-user.yaml
```

### 令牌访问

```bash
# 获取访问令牌
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

结果如下

```
eyJhbGciOiJSUzI1NiIsImtpZCI6ImNYMGlrWUlVbGVZNWYwZWQ3Uy1DR21GQndQb2p3cHdmZUJaamlyQXBjaXcifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLXNjcWt3Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI4YTllNmYxNy00YjhjLTRlZTctYTU2Mi1jNGMwMzMzMzMyNTUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.J36Pofh8QG8hOhA7o_qQtklF3wMtQ_j2VfJucfhUYj4dLRqh7bxJ5ezW4VrgpobYNQlULF1wHS49XVsoFQnzmsD8GO5fmqP7U3q_t8jrfuH7aILiThCFxh3eALbGjjzfihlC9guoH9lrT4--1O9FBF1lAeeB3xzMT0GTcq4a4xXkzNuBXcjrD4UIxdsOPaRnVqUz2ISbML-B90_AFCEGrXSu-qY33_LonyG3cJimGitAhv3C-gAg9K4PzPuWgIij6q-40ejr1riORh7kxaN6AQFLUrBGRlpM9F29KMYeZ1Avp6Dr8FzX6xBc1DtxB-p775aew3OEvyWyTbd9kB3O5g
```

## 部署ingress

### 部署

 实际上，Ingress相当于一个7层的负载均衡器，是kubernetes对反向代理的一个抽象，它的工作原理类似于Nginx，可以理解成在Ingress里建立诸多映射规则，Ingress Controller通过监听这些配置规则并转化成Nginx的反向代理配置 , 然后对外部提供服务。

两个核心概念：

ingress：kubernetes中的一个对象，作用是定义请求如何转发到service的规则
ingress controller：具体实现反向代理及负载均衡的程序，对ingress定义的规则进行解析，根据配置的规则来实现请求转发，实现方式有很多，比如Nginx, Contour, Haproxy等等

```bash
# 下载
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.47.0/deploy/static/provider/baremetal/deploy.yaml
# 修改镜像
vim deploy.yaml
# 将image的值改为如下值：
# 原值 k8s.gcr.io/ingress-nginx/controller:v0.46.0
registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/ingress-nginx-controller:v0.46.0
# 安装
kubectl apply -f deploy.yaml
# 检查安装的结果
kubectl get pod,svc -n ingress-nginx
# kubectl get pod,svc -n ingress-nginx
# 可以看到ingress通过NodePort暴露了流量
# 直接访问 http://172.31.30.128:30446
# 直接访问 https://172.31.30.128:31416
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-2gxnz        0/1     Completed   0          104s
pod/ingress-nginx-admission-patch-hhkl4         0/1     Completed   0          104s
pod/ingress-nginx-controller-65bf56f7fc-x66ks   1/1     Running     0          104s

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.96.194.2     <none>        80:30446/TCP,443:31416/TCP   104s
service/ingress-nginx-controller-admission   ClusterIP   10.96.224.131   <none>        443/TCP                      104s
# 最后别忘记把svc暴露的端口要放行
```

### 实验

![ingress-nginx-lab](images\ingress-nginx-lab.png)

#### **准备service和pod**

创建 ingress-test.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ingress-test

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-server
  namespace: ingress-test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: hello-server
  template:
    metadata:
      labels:
        app: hello-server
    spec:
      containers:
      - name: hello-server
        image: registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/hello-server
        ports:
        - containerPort: 9000

---

apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
  namespace: ingress-test
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - image: nginx
        name: nginx

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-demo
  name: nginx-demo
  namespace: ingress-test
spec:
  selector:
    app: nginx-demo
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 80

---

apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-server
  name: hello-server
  namespace: ingress-test
spec:
  selector:
    app: hello-server
  ports:
  - port: 8000
    protocol: TCP
    targetPort: 9000
```

运行

```bash
# 创建
[root@master ~]# kubectl create -f ingress-test.yaml

# 查看
[root@master ~]# kubectl get pod,svc -n ingress-test
NAME                                READY   STATUS              RESTARTS   AGE
pod/hello-server-6cbb679d85-nngcr   1/1     Running             0          16s
pod/hello-server-6cbb679d85-wlr6n   0/1     ContainerCreating   0          16s
pod/nginx-demo-7d56b74b84-fjxm4     1/1     Running             0          16s
pod/nginx-demo-7d56b74b84-rbwdb     0/1     ContainerCreating   0          16s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/hello-server   ClusterIP   10.96.134.147   <none>        8000/TCP   16s
service/nginx-demo     ClusterIP   10.96.207.53    <none>        8000/TCP   16s
```

#### 域名访问

创建 ingress-demo.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: ingress-host-bar
  namespace: ingress-test
spec:
  ingressClassName: nginx
  rules:
  - host: "hello.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: hello-server
            port:
              number: 8000
  - host: "demo.atguigu.com"
    http:
      paths:
      - pathType: Prefix
        path: "/nginx"  # 把请求会转给下面的服务，下面的服务一定要能处理这个路径，不能处理就是404
        backend:
          service:
            name: nginx-demo  ## java，比如使用路径重写，去掉前缀nginx
            port:
              number: 8000
```



```bash
# 创建
[root@master ~]# kubectl create -f ingress-demo.yaml
Error from server (InternalError): error when creating "ingress-demo.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1beta1/ingresses?timeout=10s": dial tcp 10.96.224.131:443: connect: connection refused


# 查看
[root@master ~]# kubectl get ing ingress-http -n dev
NAME           HOSTS                                  ADDRESS   PORTS   AGE
ingress-http   nginx.itheima.com,tomcat.itheima.com             80      22s

# 查看详情
[root@master ~]# kubectl describe ing ingress-http  -n dev
...
Rules:
Host                Path  Backends
----                ----  --------
nginx.itheima.com   / nginx-service:80 (10.244.1.96:80,10.244.1.97:80,10.244.2.112:80)
tomcat.itheima.com  / tomcat-service:8080(10.244.1.94:8080,10.244.1.95:8080,10.244.2.111:8080)
...

# 接下来,在本地电脑上配置host文件,解析上面的两个域名到192.168.109.100(master)上
# 然后,就可以分别访问tomcat.itheima.com:32240  和  nginx.itheima.com:32240 查看效果了

```



### 错误: 访问出错

访问 http://172.31.30.128:30446 和 访问 http://172.31.30.128:31416 都出现错误, 并没有显示出来nginx的错误标识来

### 错误: 创建ingress出错

创建的时候出错

```bash
[root@master ~]# kubectl create -f ingress-demo.yaml
Error from server (InternalError): error when creating "ingress-demo.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1beta1/ingresses?timeout=10s": dial tcp 10.96.224.131:443: connect: connection refused
```

检查如下

```bash
[root@k8s-master ~]# kubectl get pod,svc -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-2gxnz        0/1     Completed   0          60m
pod/ingress-nginx-admission-patch-hhkl4         0/1     Completed   0          60m
pod/ingress-nginx-controller-65bf56f7fc-x66ks   1/1     Running     0          60m

NAME                                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx-controller             NodePort    10.96.194.2     <none>        80:30446/TCP,443:31416/TCP   60m
service/ingress-nginx-controller-admission   ClusterIP   10.96.224.131   <none>        443/TCP                      60m
```

从这里可以看到10.96.224.131的ClusterIP所对应的service

[kubernetes1.20搭建ingress遇到failed calling webhook “validate.nginx.ingress.kubernetes.io“_Ackerly Xu的博客-CSDN博客](https://blog.csdn.net/sishen1994/article/details/112424856)

[双栈 部署ingress-nginx【附源码】_juestnow_51CTO博客](https://blog.51cto.com/juestnow/2493608)

## 部署存储

```bash
# 所有机器安装
yum install -y nfs-utils

# nfs主节点
echo "/nfs/data/ *(insecure,rw,sync,no_root_squash)" > /etc/exports
mkdir -p /nfs/data
systemctl enable rpcbind --now
systemctl enable nfs-server --now
# 配置生效
exportfs -r

# nfs从节点
showmount -e 172.31.0.4
# 执行以下命令挂载 nfs 服务器上的共享目录到本机路径 /root/nfsmount
mkdir -p /nfs/data
mount -t nfs 172.31.0.4:/nfs/data /nfs/data
# 写入一个测试文件
echo "hello nfs server" > /nfs/data/test.txt
```

### 原生方式挂载

### PV/PVC挂载

