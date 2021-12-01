# secret-volume

```shell
$ kubectl create secret generic user --from-file=username.txt
$ kubectl create secret generic pass --from-file=password.txt
$ kubectl get secrets
```

也可以直接通过编写 YAML 文件的方式来创建这个 Secret 对象，比如：secret.yaml

运行

```shell
$ kubectl apply -f secret-volume.yaml
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
test-projected-volume   1/1     Running   0          34s

# 当 Pod 变成 Running 状态之后，我们再验证一下这些 Secret 对象是不是已经在容器里了：
$ kubectl exec -it test-projected-volume -- /bin/sh
$ ls /projected-volume/
user
pass
$ cat /projected-volume/user
root
$ cat /projected-volume/pass
1f2d1e2e67df
```

更重要的是，像这样通过挂载方式进入到容器里的 Secret，一旦其对应的 Etcd 里的数据被更新，这些 Volume 里的文件内容，同样也会被更新。其实，这是 kubelet 组件在定时维护这些 Volume。

需要注意的是，这个更新可能会有一定的延时。所以在编写应用程序时，在发起数据库连接的代码处写好重试和超时的逻辑，绝对是个好习惯。

# configmap

```shell
$ kubectl create configmap ui-config --from-file=configmap-ui.properties

# 从.properties文件创建ConfigMap
$ kubectl get configmap ui-config
NAME        DATA   AGE
ui-config   1      10s

# 查看这个ConfigMap里保存的信息(data)
$ kubectl get configmaps ui-config -o yaml
apiVersion: v1
data:
  configmap-ui.properties: |-
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
kind: ConfigMap
metadata:
  creationTimestamp: "2021-08-22T07:40:43Z"
  name: ui-config
  namespace: default
  resourceVersion: "153195"
  uid: 20f133ed-6de7-4028-86fb-2f7184a21da5
```

# downward API

让 Pod 里的容器能够直接获取到这个 Pod API 对象本身的信息。

```shell
$ kubectl apply -f downwardapi-volume.yaml
$ kubectl logs test-downwardapi-volume
cluster="test-cluster1"
rack="rack-22"
zone="us-est-coast
```

Downward API 能够获取到的信息，一定是 Pod 里的容器进程启动之前就能够确定下来的信息