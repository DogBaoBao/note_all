# 一拳搞定 Kubenetes \| 基本操作

## 创建 deployment

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ kb create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
deployment.apps/hello-minikube created
```

控制台效果

![](../../.gitbook/assets/image%20%2814%29.png)

查看：

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
hello-minikube   1/1     1            1           17m
```

## 创建 service

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ kubectl expose deployment hello-minikube --type=NodePort --port=8778
service/hello-minikube exposed
```

控制台效果：

![](../../.gitbook/assets/image%20%2810%29.png)

查看：

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ kubectl get services
NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
hello-minikube   NodePort    10.96.74.40   <none>        8778:31259/TCP   5m41s
kubernetes       ClusterIP   10.96.0.1     <none>        443/TCP          22h
```

服务类型查看如下地址：

{% embed url="https://kubernetes.io/zh/docs/concepts/services-networking/service/\#publishing-services-service-types" %}

## 查看 Pod

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
hello-minikube-797f975945-hqnfk   1/1     Running   0          15m
```

注意 `STATUS` 会从 `ContainerCreating`  到 `Running`

控制台效果：

![](../../.gitbook/assets/image%20%2812%29.png)

## 查看暴露的地址

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ minikube service hello-minikube --url
http://192.168.64.3:31259
```

去访问发现是不通的。

![](../../.gitbook/assets/image%20%2811%29.png)



