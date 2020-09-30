# 一拳搞定 Kubenetes \| 基础知识2

## 伸缩性

### 概念

在之前的模块中，我们创建了一个 [Deployment](https://kubernetes.io/docs/user-guide/deployments/)，然后通过 [Service](https://kubernetes.io/docs/user-guide/services/)让其可以开放访问。Deployment 仅为跑这个应用程序创建了一个 Pod。 当流量增加时，我们需要扩容应用程序满足用户需求。

**扩缩** 是通过改变 Deployment 中的副本数量来实现的。

扩展 Deployment 将创建新的 Pods，并将资源调度请求分配到有可用资源的节点上，收缩 会将 Pods 数量减少至所需的状态。Kubernetes 还支持 Pods 的[自动缩放](https://kubernetes.io/docs/user-guide/horizontal-pod-autoscaling/)，但这并不在本教程的讨论范围内。将 Pods 数量收缩到0也是可以的，但这会终止 Deployment 上所有已经部署的 Pods。

运行应用程序的多个实例需要在它们之间分配流量。服务 \(Service\)有一种负载均衡器类型，可以将网络流量均衡分配到外部可访问的 Pods 上。服务将会一直通过端点来监视 Pods 的运行，保证流量只分配到可用的 Pods 上。

好处：

一旦有了多个应用实例，就可以没有宕机地滚动更新。

### 实操

给 hello-minikube 扩到 2  个 pod：

```bash
tiechengdeMacBook-Pro-2:.kube tc$ kb scale --replicas=2 deployment/hello-minikube
deployment.apps/hello-minikube scaled
```

再查看 pods：

```bash
tiechengdeMacBook-Pro-2:.kube tc$ kb get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-797f975945-hqnfk   1/1     Running   0          18h
hello-minikube-797f975945-prwrg   1/1     Running   0          70s
```

可以看到已经运行了两个了。

我们再次发起 HTTP 请求：

```bash
tiechengdeMacBook-Pro-2:.kube tc$ curl http://192.168.64.3:31259


Hostname: hello-minikube-797f975945-prwrg

Pod Information:
        -no pod information available-

Server values:
        server_version=nginx: 1.13.3 - lua: 10008

Request Information:
        client_address=172.17.0.1
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://192.168.64.3:8080/

Request Headers:
        accept=*/*
        host=192.168.64.3:31259
        user-agent=curl/7.64.1

Request Body:
        -no body in request-
```

```bash
tiechengdeMacBook-Pro-2:.kube tc$ curl http://192.168.64.3:31259


Hostname: hello-minikube-797f975945-hqnfk

Pod Information:
        -no pod information available-

Server values:
        server_version=nginx: 1.13.3 - lua: 10008

Request Information:
        client_address=172.17.0.1
        method=GET
        real path=/
        query=
        request_version=1.1
        request_scheme=http
        request_uri=http://192.168.64.3:8080/

Request Headers:
        accept=*/*
        host=192.168.64.3:31259
        user-agent=curl/7.64.1

Request Body:
        -no body in request-
```

可以看到 Hostname 已经出现了 2 个 pod 的名称，证明这 2 个 pod 都开始提供服务。

