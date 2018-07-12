---
title: "Kubernetes in action(读书 2)--Pod（WIP）"
date: 2018-07-09T00:00:47+08:00
draft: false
tags: ["Kubernetes","book"]
author: "Peter Yang"
---
# Pod 简介
Pod 在K8s里面是最基础的部署单元，Pod里面可以包含一个或多个Container, 而Pod里面的Container永远是一块部署的，不会存在同一个Pod里面的Container部署在不同的机器这种情况。同一个Pod内部的Container共享一个IP和端口空间，Container之间可以用localhost来访问，所有的Pod通过一个简单的网络连接， 我理解就是所有的Pod都在一个局域网里（？），不需要NAT，所以一个Pod可以通过另一个Pod的IP地址访问到它，而不用关心另一个Pod会在哪个Node上运行（如下图）。

![][1]


所以逻辑上Pod感觉像个VM，这些在Pod里面的Container感觉像VM中的进程一样，他们共享一个IP地址和端口空间，并且可以访问同样的Volumn

# 用YAML创建Pod

K8s里面的resource一般都可以用YAML或者是Json描述文件来创建， 在创建之前， 我们可以找一个已经存在的Pod，然后看下他的YAML是啥样的,在k8s集群执行这个命令

	$ kubectl get po $pod_name -o yaml


[1]: /img/pod_network.png
