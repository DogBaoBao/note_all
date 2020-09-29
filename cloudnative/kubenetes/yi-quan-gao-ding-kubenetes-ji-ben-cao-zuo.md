# 一拳搞定 Kubenetes \| 基本操作

## 创建 deployment

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ kb create deployment hello-minikube --image=k8s.gcr.io/echoserver:1.10
deployment.apps/hello-minikube created
```

控制台效果

![](../../.gitbook/assets/image%20%2816%29.png)

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

![](../../.gitbook/assets/image%20%2811%29.png)

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

![](../../.gitbook/assets/image%20%2813%29.png)

## 查看暴露的地址

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ minikube service hello-minikube --url
http://192.168.64.3:31259
```

去访问发现是不通的。

![](../../.gitbook/assets/image%20%2812%29.png)

我尝试 PING 的话是能够通的：

```bash
tiechengdeMacBook-Pro-2:.kube tc$ ping -c 4 192.168.64.3
PING 192.168.64.3 (192.168.64.3): 56 data bytes
64 bytes from 192.168.64.3: icmp_seq=0 ttl=64 time=0.249 ms
64 bytes from 192.168.64.3: icmp_seq=1 ttl=64 time=0.240 ms
64 bytes from 192.168.64.3: icmp_seq=2 ttl=64 time=0.261 ms
64 bytes from 192.168.64.3: icmp_seq=3 ttl=64 time=0.449 ms

--- 192.168.64.3 ping statistics ---
4 packets transmitted, 4 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.240/0.300/0.449/0.086 ms
```

所以怀疑是端口的问题，仔细查看了官方的例子，我改了 expose 的端口 8778，所以我怀疑因为这个端口和 8080 没映射上，所以导致调用不到。

查看 pod 的这个端口是否可用：

![](../../.gitbook/assets/image%20%2810%29.png)

发现是可以正常请求到的，于是去查看 service 的配置信息：

![](../../.gitbook/assets/image%20%2814%29.png)

很明显这个端口映射有问题，把 targetPort 改成 8080 后，访问正常如下：

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



