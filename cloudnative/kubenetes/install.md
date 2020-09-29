---
description: 在本机 mac 安装单节点的 kubenetes 供学习。
---

# 一拳搞定 Kubenetes \| 安装

> Kubernetes 系列文章，无逻辑，就是干。
>
> 请自行备好 docker， linux 等常规知识。

## 基于 minukube

安装 minikube 非常便利，基本上根据官方文档看下，敲几下就可以搭建起来，推荐大家使用。

{% embed url="https://kubernetes.io/zh/docs/tasks/tools/install-minikube/" %}

当按照教程 `minikube start`，会输出如下：

```bash
tiechengdeMacBook-Pro-2:.kube tc$ minikube start
* Darwin 10.15.6 上的 minikube v1.6.2
  - KUBECONFIG=/Users/tc/.kube/config.22
* Automatically selected the 'hyperkit' driver (alternates: [])
* 正在创建 hyperkit 虚拟机（CPUs=2，Memory=2000MB, Disk=20000MB）...
* 正在 Docker '19.03.5' 中准备 Kubernetes v1.17.0…
* 拉取镜像 ...
* 正在启动 Kubernetes ... 
* Waiting for cluster to come online ...
* 完成！kubectl 已经配置至 "minikube"
```

### kubeconfig

如上面的输出有一个解释下：

```bash
- KUBECONFIG=/Users/tc/.kube/config.22
```

这里 KUBECONFIG 是可以通过环境变量指定的，比如我本地：

```bash
export KUBECONFIG=/Users/tc/.kube/config.22
```

里面的内容如下：

```bash
tiechengdeMacBook-Pro-2:docker-compose tc$ cat /Users/tc/.kube/config.22
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURBVENDQWVtZ0F3SUJBZ0lKQU91U0IwYkU3VGIrTUEwR0NTcUdTSWIzRFFFQkN3VUFNQmN4RlRBVEJnTlYKQkFNTURERXdMakUxTWk0eE9ETXVNVEFlRncweE9URXlNekF3TmpVek1qTmFGdzAwTnpBMU1UY3dOalV6TWpOYQpNQmN4RlRBVEJnTlZCQU1NRERFd0xqRTFNaTR4T0RNdU1UQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQCkFEQ0NBUW9DZ2dFQkFNazV4T2tXb0RXUzdRdFZhOFFCN21DbFFCeC90N2VHdXpEdENYVXYyWU8wcVBJa3RHRVQKaDZkM2dFQ3kzK0ZVVGhDWTF1U2gzYkhDT2wybXJPWnJmN21DelVPdDJZSWQ3QWFVOEVZQzN6dDl6L2d3NTdBWgpWeHE4L3NYZUNQSlBQZXVpOTR0b252cTBSNXNVMm0veTB3UFJMdTFkS052cUV6K05YNFRPUnpCcVpNeVoycXU0CjNMazJCMkNycXJnOXQ1Wks0cU9wQTMwZUxYNHMwVlZUN042RkdFWlRFMTkzdXNLc2dkV2FGb1pJcnh6LzZDbUUKemlDektMcVZiNzAxdWJVZXlCTEU0OER4cVVyZTdtYkoxSWI0VUU5NjE1cDhuSml2UlRJdXZ5aUlxazFieEQ5RwpkaVNEMTdUV1NpTnUwelUyQkJWaEV4OTRYbDcwcENucnhxRUNBd0VBQWFOUU1FNHdIUVlEVlIwT0JCWUVGS29jClFUVE1RYTF1TzZMVk1nVEd0WndsMFNxek1COEdBMVVkSXdRWU1CYUFGS29jUVRUTVFhMXVPNkxWTWdUR3Rad2wKMFNxek1Bd0dBMVVkRXdRRk1BTUJBZjh3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCQUZodlYwVkxGMlRYU0hTLwpGeXBQNG1jMy8vWG9RdWRrUUdNSTJQWDNmNk8xWlBteFVrOGpiMnhiUmJ0aGhTcU5BU0xPSGdSQnRqYjF1UnVaCi96NVliRnhpTEFXdWYyd3FoSzJtRkl4WHg2enF1bExremszRFMxRjZnVWxhYS9hRXoxWW5XdWhUdzArenRtYkkKQkdENnRaTUM3ZDA1YlhseTREQkNtZklPclNNUW14eURvN0IxV1RHWklpd2NwSzN5clJsUEFCc2w5dnJUcit0RQoydUwxaytoYnB3ZjRreVZrNzNRZWdWTGZaK3Q3ZUZRVUpCMDR2V1RSdmtQcVdoTWhqQXRTR1FObG80TVJ3eklWCit2TDUyVGcxZjNXb09LRldyQkQzM1MzOGhLWlpJbzVKUDlPMmxDUU9VRXhRVzZEcmN0UEp0bmhRbHdEUjQ0a04KL1FxNmFNZz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://30.11.44.31:16443
  name: microk8s-cluster
- cluster:
    certificate-authority: /Users/tc/.minikube/ca.crt
    server: https://192.168.64.3:8443
  name: minikube
contexts:
- context:
    cluster: microk8s-cluster
    user: admin
  name: microk8s
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: admin
  user:
    password: d0F2R1hDc2RYUEgxWFA4RGtpWVdFQldHM2xrU0Q2M3czZ3ovK3YzNXJxaz0K
    username: admin
- name: minikube
  user:
    client-certificate: /Users/tc/.minikube/client.crt
    client-key: /Users/tc/.minikube/client.key
```

这份文件是我之前配置的，里面的服务地址均已经不正确，我先往下走看看是否会出现什么问题。

{% embed url="https://kubernetes.io/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/" %}

### dashboard

```bash
tiechengdeMacBook-Pro-2:.kube tc$ minikube dashboard
E0928 17:18:13.216884   19644 dashboard.go:90] Error excluding IP from proxy: ExcludeIP() requires IP or CIDR as a parameter
* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
* Opening http://127.0.0.1:63999/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

强烈建议安装 Dashboard （新手必备），可以直观的看到控制台：

![](../../.gitbook/assets/image%20%286%29.png)

根据这个页面，可以方便的和各个概念去对应，能加快理解。

{% embed url="https://kubernetes.io/zh/docs/setup/learning-environment/minikube/\#%E4%BB%AA%E8%A1%A8%E7%9B%98" %}

## 基于 docker desktop 运行

勾选上 Enable Kubernetes

![](../../.gitbook/assets/image%20%289%29.png)

直接参考官方文档和简书一篇蛮新的文章：

{% embed url="https://docs.docker.com/docker-for-mac/\#kubernetes" %}

{% embed url="https://www.jianshu.com/p/e5c056baa8" %}



