# 一拳搞定 Kubenetes \| 基础知识2

## 伸缩性

### 概念

在之前的模块中，我们创建了一个 [Deployment](https://kubernetes.io/docs/user-guide/deployments/)，然后通过 [Service](https://kubernetes.io/docs/user-guide/services/)让其可以开放访问。Deployment 仅为跑这个应用程序创建了一个 Pod。 当流量增加时，我们需要扩容应用程序满足用户需求。

**扩缩** 是通过改变 Deployment 中的副本数量来实现的。

扩展 Deployment 将创建新的 Pods，并将资源调度请求分配到有可用资源的节点上，收缩 会将 Pods 数量减少至所需的状态。Kubernetes 还支持 Pods 的[自动缩放](https://kubernetes.io/docs/user-guide/horizontal-pod-autoscaling/)，但这并不在本教程的讨论范围内。将 Pods 数量收缩到0也是可以的，但这会终止 Deployment 上所有已经部署的 Pods。

运行应用程序的多个实例需要在它们之间分配流量。服务 \(Service\)有一种负载均衡器类型，可以将网络流量均衡分配到外部可访问的 Pods 上。服务将会一直通过端点来监视 Pods 的运行，保证流量只分配到可用的 Pods 上。

好处：

一旦有了多个应用实例，就可以没有宕机地滚动更新。

{% embed url="https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/scale/scale-intro/" %}

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

## 滚动更新

用户希望应用程序始终可用，而开发人员则需要每天多次部署它们的新版本。在 Kubernetes 中，这些是通过滚动更新（Rolling Updates）完成的。 **滚动更新** 允许通过使用新的实例逐步更新 Pod 实例，零停机进行 Deployment 更新。新的 Pod 将在具有可用资源的节点上进行调度。

在前面的模块中，我们将应用程序扩展为运行多个实例。这是在不影响应用程序可用性的情况下执行更新的要求。默认情况下，更新期间不可用的 pod 的最大值和可以创建的新 pod 数都是 1。这两个选项都可以配置为（pod）数字或百分比。 在 Kubernetes 中，更新是经过版本控制的，任何 Deployment 更新都可以恢复到以前的（稳定）版本。

> 我们可以追求随时发布，然后通过灰度去验证，success 则滚动发布所有。fail 则回滚灰度的到之前版本，这些都是可以做到自动化。

滚动更新允许以下操作：

* 将应用程序从一个环境提升到另一个环境（通过容器镜像更新）
* 回滚到以前的版本
* 持续集成和持续交付应用程序，无需停机

{% embed url="https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/update/update-intro/" %}



