# K8S_Webhook结合CRD进阶玩法

本篇非K8S Webhook入门，入门请参考之前的文章：**2021-08-29_从0到1开发K8S_Webhook最佳实践**

本章主要讲两个内容

1：对资源进行复杂修改后，怎么快速得到Json Patch

2：结合CRD自定义资源实现Webhook动态读取数据

所有代码都在（V2版本）：https://github.com/scriptwang/admission-webhook-example

## 背景

在之前的文章**2021-08-29_从0到1开发K8S_Webhook最佳实践**实现了对资源进行简单修改，也就是添加几个label，核心代码如下

```go
func updateLabels(target map[string]string, added map[string]string) (patch []patchOperation) {
	values := make(map[string]string)
	for key, value := range added {
		if target == nil || target[key] == "" {
			values[key] = value
		}
	}
	patch = append(patch, patchOperation{
		Op:    "add",
		Path:  "/metadata/labels",
		Value: values,
	})
	return patch
}

added = map[string]string{
    "app.kubernetes.io/name":     "not_available" ,
    "app.kubernetes.io/instance":  "not_available" ,
}
```

逻辑很简单，判断target，也就是已经有的里面是否包含本次需要添加的（added中的值），包含则不添加，反之则添加，最后封装成了`patchOperation`切片，`patchOperation`也就是一个简单的结构体

```go
type patchOperation struct {
	Op    string      `json:"op"`
	Path  string      `json:"path"`
	Value interface{} `json:"value,omitempty"`
}
```

其中的`Path:  "/metadata/labels"`是需要自己判断的，为啥要这么写Patch？这其实是json相关内容了，详情参考：http://jsonpatch.com/

那么问题来了，简单的Patch可以这样操作，那么复杂的Patch呢？比如增加一个`initContainer`，Patch例子如下

```json
[
    {
        "op": "add",
        "path": "/spec/template/spec/initContainers",
        "value": [
            {
                "command": [
                    "/bin/sh",
                    "-c",
                    " echo 'init' \u0026\u0026 sleep 100 "
                ],
                "image": "busybox",
                "name": "init",
                "resources": {
                    "requests": {
                        "cpu": "200m",
                        "memory": "400Mi"
                    }
                }
            }
        ]
    }
]
```

难道要手动拼接？拼接这个Json Patch就够喝一壶了，有没有啥简单办法？答案就是jsondiff



## jsondiff找不同？

github：https://github.com/wI2L/jsondiff

> **jsondiff** is a Go package for computing the *diff* between two JSON documents as a series of [RFC6902](https://tools.ietf.org/html/rfc6902) (JSON Patch) operations, which is particularly suitable to create the patch response of a Kubernetes Mutating Webhook for example.

简介都说了非常适合用来创建Kubernetes  Webhook返回的Json Patch

使用方式非常的人性，下面以添加一个initContainer为例

```go
//旧对象
newDeploy := deploy.DeepCopy()
//深克隆一个新对象
newPodSpec := &newDeploy.Spec.Template.Spec

//下面是对新对象进行一顿操作
//如果没有initContainer则新加一个
var initContainer *corev1.Container
if len(newPodSpec.InitContainers) == 0 {
    newPodSpec.InitContainers = []corev1.Container{
        {
            Name:    "init",
            Image:   "busybox",
            Command: []string{"/bin/sh", "-c", " echo 'init' && sleep 100 "},
        },
    }
    initContainer = &newPodSpec.InitContainers[0]
    log.WriteString("\nmutate add initContainer sucess!")		
}

/********************************************************* 结束修改操作 */

// 比较新旧deploy的不同，返回不同的bytes
patch, err := jsondiff.Compare(deploy, newDeploy)
if err != nil {
    ...//错误处理
}

//打patch，patchBytes就是我们需要的了
patchBytes, err := json.MarshalIndent(patch, "", "    ")
if err != nil {
    ...//错误处理
}

//打印出来看一下
fmt.Println(string(patchBytes))
```

打印出来就是下面这样的

```json
[
    {
        "op": "add",
        "path": "/spec/template/spec/initContainers",
        "value": [
            {
                "command": [
                    "/bin/sh",
                    "-c",
                    " echo 'init' \u0026\u0026 sleep 100 "
                ],
                "image": "busybox",
                "name": "init",
                "resources": {
                    "requests": {
                        "cpu": "200m",
                        "memory": "400Mi"
                    }
                }
            }
        ]
    }
]
```

有感觉了么，不用手动拼接Json Patch，直接深克隆一个对象出来，直接修改新对象的值（这比手动拼接Json Patch爽多了？），然后对比新老对象，jsondiff会自动把不同找出来生产Json Patch，

## json转yaml？

全是json看着不爽？我怎么知道我修改的地方转成yaml最后到底对不对？那就需要一个json和yaml互转的工具了，它就是yaml：https://github.com/ghodss/yaml

> YAML marshaling and unmarshaling support for Go

上面不是有了深克隆并且添加了initContainer的新对象么，直接把该对象转换成json，然后在转换成yaml打印出来

```go
//旧对象
newDeploy := deploy.DeepCopy()
//深克隆一个新对象
newPodSpec := &newDeploy.Spec.Template.Spec
//对新对象一顿修改操作
...
//转换json
bytes, err := json.Marshal(newPodSpec)
if err == nil {
    //转换yaml
    yamlStr, err := yaml.JSONToYAML(bytes)
    if err == nil {
        fmt.Println(string(yamlStr))
    }
}
```

打印出来如下，看到initContainers了吧，这看起来就非常直观了，比满屏幕的没有格式化的Json看起来舒服多了

```yaml
containers:
- command:
  - /bin/sleep
  - infinity
  image: busybox
  imagePullPolicy: IfNotPresent
  name: sleep
  resources: {}
  terminationMessagePath: /dev/termination-log
  terminationMessagePolicy: File
dnsPolicy: ClusterFirst
initContainers:
- command:
  - /bin/sh
  - -c
  - ' echo ''init'' && sleep 100 '
  image: busybox
  name: init
  resources:
    requests:
      cpu: 200m
      memory: 400Mi
restartPolicy: Always
schedulerName: default-scheduler
securityContext: {}
terminationGracePeriodSeconds: 30
```

## CRD？

CRD全称为CustomResourceDefinition，即自定义资源

可参考：https://kubernetes.io/zh/docs/concepts/extend-kubernetes/api-extension/custom-resources/

那么什么是自定义资源？在K8S中。Pod、Deployment、Service都是内建资源，如果这些资源不满足我们的需求，那么就可以自定义资源，假设场景如下：

需要在Pod部署的时候添加一个initContainer，并且根据需要动态的设置该initContainer的资源（CPU、内存），所以资源设置不能再Webhook里面写死，需要从外部动态读取，这个时候就需要CRD（也可以从别的地方读取，比如数据库，本质上Webhook是一个第三方WebServer，既然都用K8S了，那就直接定义CRD吧，相当于变相从K8S的ETCD里面读取数据）

整体逻辑为：在创建CRD的时候同步设置Webhook中CPU、内存变量的值，修改同理，删除的时候变回默认值。这样在Deployment创建的时候根据CPU、内存的值去修改相应的资源

### 定义CRD

要实现上面的需求，那么需要定义一个全局的CRD叫QoS，如下，详细定义在K8S文档都有说明（上面的链接），此处不在赘述

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # 名字必需与下面的 spec 字段匹配，并且格式为 '<名称的复数形式>.<组名>'
  name: qoss.stable.example.com
spec:
  # 组名称，用于 REST API: /apis/<组>/<版本>
  group: stable.example.com
  # 列举此 CustomResourceDefinition 所支持的版本
  versions:
    - name: v1
      # 每个版本都可以通过 served 标志来独立启用或禁止
      served: true
      # 其中一个且只有一个版本必需被标记为存储版本
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cpu:
                  type: integer
                memory:
                  type: integer
  # 可以是 Namespaced 或 Cluster
  scope: Cluster
  names:
    # 名称的复数形式，用于 URL：/apis/<组>/<版本>/<名称的复数形式>
    plural: qoss
    # 名称的单数形式，作为命令行使用时和显示时的别名
    singular: qos
    # kind 通常是单数形式的驼峰编码（CamelCased）形式。你的资源清单会使用这一形式。
    kind: QoS
    # shortNames 允许你在命令行使用较短的字符串来匹配资源
    shortNames:
    - qos
```

这相当于定义了一个类，然后我们创建一个该资源的实例

```yaml
apiVersion: "stable.example.com/v1"
kind: QoS
metadata:
  name: qos-default-policy
spec:
  cpu: 100
  memory: 100
```



### 定义MutatingWebhookConfiguration

相应的MutatingWebhookConfiguration也需要修改，增加对我们自定义资源的修改，创建，删除，更新QoS的时候都要触发Webhook

```yaml
apiVersion: admissionregistration.k8s.io/v1beta1
kind: MutatingWebhookConfiguration
metadata:
  name: mutating-webhook-example-cfg-debug
  labels:
    app: admission-webhook-example-debug
webhooks:
  - name: mutating-example.qikqiak.com.debug
    clientConfig:
      service:
        name: admission-webhook-example-svc-debug
        namespace: default
        path: "/mutate"
      caBundle: ${CA_BUNDLE}
    rules:
      - operations: [ "CREATE" ]
        apiGroups: ["apps", ""]
        apiVersions: ["v1"]
        resources: ["deployments"]
      # 主要在此处添加QoS的内容，创建，删除，更新的时候都要触发
      - operations: [ "CREATE","DETELE","UPDATE" ]
        apiGroups: ["stable.example.com", ""]
        apiVersions: ["v1"]
        resources: ["qoss"]
    namespaceSelector:
      matchLabels:
        admission-webhook-example: enabled
```



### 增加qos.go

增加一个文件代表CRD资源设置的值

```go
package main

type QoS struct {
	Spec QoSpec `json:"spec,omitempty"`
}

type QoSpec struct {
	Cpu    int64 `json:"cpu,omitempty"`
	Memory int64 `json:"memory,omitempty"`
}

var (
	QoSInst = &QoSpec{}
)

func (qos *QoSpec) getQoSpec() *QoSpec {
	//判断空对象
	if (*QoSInst == QoSpec{}) {
		//默认值
		return &QoSpec{
			Cpu:    200,
			Memory: 400,
		}
	} else {
		return QoSInst
	}
}

func (qos *QoSpec) setQoSpec(qoSpec *QoSpec) {
	QoSInst = qoSpec
}

```

对QoS的设置

```go
case "QoS":
		var qos QoS
		var raw []byte
		if req.Operation == "DELETE" {
			raw = req.OldObject.Raw
		} else {
			raw = req.Object.Raw
		}
		if err := json.Unmarshal(raw, &qos); err != nil {
			log.WriteString(fmt.Sprintf("\nCould not unmarshal raw object: %v", err))
			glog.Errorf(log.String())
			return &v1beta1.AdmissionResponse{
				Result: &metav1.Status{
					Message: err.Error(),
				},
			}
		}
		return mutateQoS(&qos, req.Operation, log)
		
//设置QoS
func mutateQoS(qos *QoS, operation v1beta1.Operation, log *bytes.Buffer) *v1beta1.AdmissionResponse {
	if operation == "DELETE" {
		//删除设置空对象
		QoSInst.setQoSpec(&QoSpec{})
        //其他的时候更新 
	} else {
		if qos != nil && (qos.Spec != QoSpec{}) {
			QoSInst.setQoSpec(&qos.Spec)
		}
	}
	return &v1beta1.AdmissionResponse{
		Allowed: true,
	}
}
```



### 验证

验证思路：新增、删除、修改QoS资源（`debug/crd/QoS.yaml`）后创建Deployment（`debug/sleep.yaml`），观察打印出来的Json Patch关于资源的设置是否正确