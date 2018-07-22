---
title: "Kubernetes in action(读书 2)--Pod（WIP）"
date: 2018-07-13T00:00:47+08:00
draft: false
tags: ["Kubernetes","book"]
author: "Peter Yang"
---
# Pod 简介
Pod 在K8s里面是最基础的部署单元，Pod里面可以包含一个或多个Container, 而Pod里面的Container永远是一块部署的，不会存在同一个Pod里面的Container部署在不同的机器这种情况。同一个Pod内部的Container共享一个IP和端口空间，Container之间可以用localhost来访问，所有的Pod通过一个简单的网络连接， 我理解就是所有的Pod都在一个局域网里（？），不需要NAT，所以一个Pod可以通过另一个Pod的IP地址访问到它，而不用关心另一个Pod会在哪个Node上运行（如下图）。

![][1]


所以逻辑上Pod感觉像个VM，这些在Pod里面的Container感觉像VM中的进程一样，他们共享一个IP地址和端口空间，并且可以访问同样的Volumn

# 用YAML创建Pod

K8s里面的resource一般都可以用YAML(推荐)或者是Json描述文件来创建.这些描述文件一般都包含一下几个部分

* Metadata 包括名字，命名空间，标签及其他的元数据
* Spec 包含对Pod内容的描述，例如container的镜像，Volumn 或者其他的数据
* Status，这就是Pod运行时的一些信息， 在定义Pod时是不需要的，但查看已经运行的Pod可以看到这块信息，包括Container的status，IP等信息

下面是一个最简单的YAML

	apiVersion: v1 #这个指定的是K8s API的版本
	kind: Pod   #这个文件描述的是Pod
	metadata:
	name: kubia-manual #Pod的名字
	spec:
	containers:
	- image: luksa/kubia #指定镜像（可以有多个）
	name: kubia	#container的名字（可省略）
	ports:
	- containerPort: 8080 #暴露的端口（可省略）
	protocol: TCP
 
上边指定了端口，但这个只是明确的表示这个Pod会监听的端口，实际上省略这个`containerPort` 并不会对Client端连接产生任何的影响，只不过别人想用的时候，要用`kubectl explain` 命令找到你暴露的端口

下边是几个跟pod有关的`kubectl`命令

	kubectl get po $podName -o ymal  #以ymal的方式展示pod的详细信息
	kubectl get pods   #显示当前所有的pod
	kubectl logs $podName #显示Pod里面container的log
	kubectl port-forward $podName 8888:8080 #将本地的8888端口映射到Pod内部的8080 端口，所以执行完这个命令后，访问localhost:8888 会 访问到 Pod内部的8080 端口,如下图所示

![][2]

# 给Pod打标签
实际上K8s里面所有的资源和组件都可以打标签，打标签的目的就是为了更容易的找到想找的东西， 当你的系统比较大或者复杂的时候正确的打标签就非常有必要，否则会很难管理。

简单的说标签(`label`)就是就是任意的键值对，你可以把这键值对attach到任意的resource上，然后可以用`label selector`做选择，一个resource上边可能有很多个标签。可以在创建的资源的时候给资源打标签， 当然也可以update已有资源的标签。下面就是创建Pod的时候指定了标签

	apiVersion: v1
	kind: Pod
	metadata:
	name: kubia-manual
	labels:  #标签
		creation_method: manual #当创建这个pod的时候，这俩标签就会attach到这个pod上
		env: pod
	spec:
	containers:
	- image: luksa/kubia
	name: kubia
	ports:
	- containerPort: 8080
	protocol: TCP

用kubectl创建上边的pod

	kubectl apply -f kubia-manual-with-labels.yaml

然后下面几个是跟标签有关的命令
	
	kubectl get po --show-labels #列出所有的pods，会列出pod的label
	kubectl label po kubia-manual creation_method=manual ，这个是给kubia-manual这个pod打上一个标签 creation_method, 值是manul
	kubectl label po kubia-manual-v2 env=debug --overwrite #这是覆盖 pod kubia-manual-v2 的 env 标签将值改成debug

# 用标签选择Pod
K8s提供了用标签选择Pod的灵活方式,`label selector` 可以提供让你选择一些符合特定标签的pods进行操作，一个`label selector`就是一个选择条件,可以有以下的使用方式

* 选择包含(或者不包含)某些标签的pod
* 选择包含某些标签和值的pod
* 选择包含某些标签但是值不等于某个给定值的pod

下面用命令展示一下`label selector`的用法 

	kubectl get po -l env #选择有env这个标签的pod
	kubectl get po -l '!env' #选择不含有env这个标签的pod
	kubectl get po -l creation_method=manual #选择creation_method是manual的pod
	kubectl get po -l app=pc,rel=beta #选择app=pc并且rel=beta的pod（多个pod同时使用的情况）

# 选择node执行
刚才是说了怎么用`label`来选择pod了， 但其实k8s中所有的resource 都可以加标签，也可以用`label selector` 来选择，比如说我集群中有些虚拟机配置比较好(有gpu 等) 但有些是比较老旧的机器， 这是侯可以用`label` 来给这些node打标签， 当要调度某些应用的时候可以指定部署到哪种类型的机器上

	kubectl label node gke-kubia-85f6-node-0rrx gpu=true #给node gke-kubia-85f6-node-0rrx 打上标签 gpu=true

现在比如说我有个挖矿的应用需要部署到有gpu的node上去，就可以用下面这种方式去指定部署到哪种node上

	apiVersion: v1
	kind: Pod
	metadata:
		name: kubia-gpu
	spec:
		nodeSelector:
			gpu: "true"  #这个nodeSelector可以指定这个pod会schedule到有gpu的node上面去
	containers:
	- image: luksa/kubia
		name: kubia


[1]: /img/pod_network.png
[2]: /img/pod_port_forward.png
