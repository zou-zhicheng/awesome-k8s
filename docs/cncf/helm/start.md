# 1. check structure
create a demo sample to check the structure   
```bash
# create
$ helm create hello-helm
# check the structure
# 资源创建顺序如何确定?
$ tree hello-helm/
hello-helm/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files

```

# 2. create basic demo
## 2.1 basic demo
```bash
$ helm create mychart
Creating mychart

$ tree mychart
mychart
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files

# remove all template
$ rm -rf mychart/templates/*

# create a sample configmap template
# for detailed context, please check below
$ vim mychart/templates/configmap.yaml

# install
$ helm install helmchart mychart/
NAME: helmchart
LAST DEPLOYED: Mon May  9 15:09:32 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

# list
$ helm ls
NAME   	    NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
helmchart	default  	1       	2022-05-09 15:09:32.942448489 +0800 CST	deployed	mychart-0.1.0	1.16.0

# get manifest
$ helm get manifest helmchart
---
# Source: helmchart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvaue: "Hello Helm"

# kubectl get configmap
$ kubectl get cm
NAME                DATA   AGE
mychart-configmap   1      3m13s

# delete
$ helm delete mychart
release "mychart" uninstalled
```

content of `mychart/templates/configmap.yaml`   
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mychart-configmap
data:
  myvaue: "Hello Helm"
```

## 2.2 引入模板
```bash
# create a sample configmap template
# for detailed context, please check below
$ vim mychart/templates/configmap.yaml

# install
$ helm install helmchart2 mychart/
NAME: helmchart2
LAST DEPLOYED: Mon May  9 15:28:21 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None

# list
$ helm ls
NAME      	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
helmchart2	default  	1       	2022-05-09 15:28:21.695592789 +0800 CST	deployed	mychart-0.1.0	1.16.0     

# get manifest
$ helm get manifest helmchart2
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart2-configmap
data:
  myvaue: "Hello Helm"

```

content of `mychart/templates/configmap.yaml`  
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvaue: "Hello Helm"
```

## 2.3 调试 dry-run
 
```bash
# 不实际安装  
$ helm install --dry-run --debug helmchart3 mychart/ 
install.go:178: [debug] Original chart version: ""
install.go:195: [debug] CHART PATH: /home/u7000029163/Workspace/helm-sample/mychart

NAME: helmchart3
LAST DEPLOYED: Mon May  9 15:34:17 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
autoscaling:
  enabled: false
  maxReplicas: 100
  minReplicas: 1
  targetCPUUtilizationPercentage: 80
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: nginx
  tag: ""
imagePullSecrets: []
ingress:
  annotations: {}
  className: ""
  enabled: false
  hosts:
  - host: chart-example.local
    paths:
    - path: /
      pathType: ImplementationSpecific
  tls: []
nameOverride: ""
nodeSelector: {}
podAnnotations: {}
podSecurityContext: {}
replicaCount: 1
resources: {}
securityContext: {}
service:
  port: 80
  type: ClusterIP
serviceAccount:
  annotations: {}
  create: true
  name: ""
tolerations: []

HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart3-configmap
data:
  myvaue: "Hello Helm"


# 检查, 并没有 helmchart3
$ helm ls
NAME      	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
helmchart2	default  	1       	2022-05-09 15:28:21.695592789 +0800 CST	deployed	mychart-0.1.0	1.16.0
```

# 3. 内置对象
一些内置对象   

- {{ .Values }}
- {{ .Chart }}
- {{ .Capabilities }}
- {{ .Release }}


示例   
```bash
# change configmap
$ vim mychart/templates/configmap.yaml

$ helm install --dry-run  helmchart4 mychart/ 
NAME: helmchart4
LAST DEPLOYED: Mon May  9 15:43:07 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart4-configmap
data:
  myvaue: "Hello Helm"
  repository: nginx

# use --set, --set的优先级最高
$ helm install --dry-run  helmchart4 mychart/ --set image.repository=kubernetes
NAME: helmchart4
LAST DEPLOYED: Mon May  9 15:45:25 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart4-configmap
data:
  myvaue: "Hello Helm"
  repository: kubernetes

```

content of `mychart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvaue: "Hello Helm"
  repository: {{ .Values.image.repository }}
```


# 5. 模板函数与管道
## 5.1 模板函数
模板函数来源

- Go语言本身定义
- Sprig模板库定义

格式   
{{ 模板函数名 参数列表 }}   

```bash
# change configmap
$ vim mychart/templates/configmap.yaml

# dry-run install
# repository: "nginx"
$ helm install --dry-run  helmchart5 mychart/ 
NAME: helmchart5
LAST DEPLOYED: Mon May  9 17:04:11 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart5-configmap
data:
  myvaue: "Hello Helm"
  repository: "nginx"
```

content of `mychart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvaue: "Hello Helm"
  repository: {{ quote .Values.image.repository }}
```

## 5.2 管道
```bash
# change configmap
$ vim mychart/templates/configmap.yaml

# dry-run install
# repository: "NGINXNGINXNGINX"
$ helm install --dry-run  helmchart6 mychart/ 
NAME: helmchart6
LAST DEPLOYED: Mon May  9 17:17:02 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart6-configmap
data:
  myvaue: "HELLO HELM"
  repository: "NGINXNGINXNGINX"

# use --set
$ helm install --dry-run  helmchart6 mychart/ --set myvalue="hello world"
NAME: helmchart6
LAST DEPLOYED: Mon May  9 17:17:54 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart6-configmap
data:
  myvaue: "HELLO WORLD"
  repository: "NGINXNGINXNGINX"
```

content of `mychart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvaue: {{ .Values.myvalue | default "Hello Helm" | quote | upper }}
  repository: {{ .Values.image.repository  | repeat 3 | quote | upper }}
```

## 6. 控制流程
## 6.1 if/else
```bash
{{ if PIPELINE }}
  # do something
{{ else if PIPELINE }}
  # do something
{{ else }}
  # default case
{{ end }}
```

结果为`false`

- 布尔值false
- 数字0
- 空字符串
- nil/null
- 空集合(map/slice/tuple/dict/array)

运算符

- eq
- neq
- lt
- gt
- ...

```bash
# change configmap
$ vim mychart/templates/configmap.yaml

# dry-run install
# 请注意nginx3的空格
$ helm install --dry-run  helmchart7 mychart/ 
NAME: helmchart7
LAST DEPLOYED: Mon May  9 17:53:01 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart7-configmap
data:
  nginx1: true
  nginx2: true
  
  nginx3: true


```

content of `mychart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{ if eq .Values.image.repository "nginx" }}nginx1: true{{ end }}
  {{- if eq .Values.image.repository "nginx" }}
  nginx2: true
  {{- end }}
  {{ if eq .Values.image.repository "nginx" }}
  nginx3: true
  {{ end }}
```

## 6.2 with
限定作用范围   
```bash
# change configmap
$ vim mychart/templates/configmap.yaml

# dry-run install
$ helm install --dry-run  helmchart8 mychart/ 
NAME: helmchart8
LAST DEPLOYED: Mon May  9 17:58:26 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart8-configmap
data:
  enabled: false
```

content of `mychart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- with .Values.ingress }}
  {{- if eq .enabled false }}
  enabled: false
  {{- end }}
  {{- end }}
```

## 6.3 range
```bash
# change values.yaml
$ vim mychart/values.yaml  

# change configmap.yaml
$ vim mychart/templates/configmap.yaml

# dry-run install
$ helm install --dry-run  helmchart9 mychart/ 
NAME: helmchart9
LAST DEPLOYED: Mon May  9 18:59:20 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart9-configmap
data:
  toppings: |-
    - "Mushrooms"
    - "Cheese"
    - "Peppers"
    - "Onions"
  sizes: |-
    - small
    - medium
    - large
```

add content to `mychart/values.yaml`
```yaml
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions    
```

content of `mychart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  toppings: |-
    {{- range .Values.pizzaToppings }}
    - {{ . | title | quote }}
    {{- end }}   
  sizes: |-
    {{- range tuple "small" "medium" "large" }}
    - {{ . }}
    {{- end }}     
```

**ABOUT "|-"**   
The toppings: |- line is declaring a multi-line string. So our list of toppings is actually not a YAML list. It's a big string. Why would we do this? Because the data in ConfigMaps data is composed of key/value pairs, where both the key and the value are simple strings. To understand why this is the case, take a look at the Kubernetes ConfigMap docs. For us, though, this detail doesn't matter much.   
>>The |- marker in YAML takes a multi-line string. This can be a useful technique for embedding big blocks of data inside of your manifests, as exemplified here.

## 6.4 其他
## 6.5 参考
https://helm.sh/docs/chart_template_guide/control_structures/   

# 7. 变量
## 7.1 示例
```bash
# add values to values.yaml

# change configmap.yaml
$ vim mychart/templates/configmap.yaml

# dry-run install
$ helm install --dry-run  helmchart10 mychart/ 
NAME: helmchart10
LAST DEPLOYED: Mon May  9 19:12:57 2022
NAMESPACE: default
STATUS: pending-install
REVISION: 1
TEST SUITE: None
HOOKS:
MANIFEST:
---
# Source: mychart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: helmchart10-configmap
data:
  releaseName: helmchart10
  courses: |-
    - java: "Spring"
    - javascript: "Nodejs"
    - k8s: "Devops"
    - python: "Django" 
  toppings: |-
    - 0: "Mushrooms"
    - 1: "Cheese"
    - 2: "Peppers"
    - 3: "Onions"
```

add content to `mychart/values.yaml`
```yaml
courses:
  k8s: devops
  python: django
  java: spring
  javascript: nodejs

pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions
```

content of `mychart/templates/configmap.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $releaseName := .Release.Name }}
  releaseName: {{ $releaseName }} 
  courses: |-
    {{- range $key, $value := .Values.courses }}
    - {{ $key }}: {{ $value | title | quote }}
    {{- end }}  
  toppings: |-
    {{- range $index, $topping := .Values.pizzaToppings }}
    - {{ $index }}: {{ $topping | title | quote }}
    {{- end }}  
```
## 7.2 参考
https://helm.sh/docs/chart_template_guide/variables/   

# 8. 命名模板
