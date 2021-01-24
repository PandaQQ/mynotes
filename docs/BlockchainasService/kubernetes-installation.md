# 在Ubuntu 20.04上安装和部署Kubernetes集群

> 了解如何在Ubuntu 20.04上安装和部署Kubernetes Cluster。 Kubernetes，据 [kubernetes.io](https://kubernetes.io/) 是开源的生产级容器编排平台。它促进了容器化应用程序的自动部署，扩展和管理



## 主机列表

| Nodes  | IP                            | vCPU   | Memory      | OS Version                |
| ------ | ----------------------------- | ------ | ----------- | ------------------------- |
| master | 119.8.237.150 / 192.168.0.181 | 4vCPUs | 8G / 100 GB | Ubuntu 20.04 server 64bit |
| worker | 119.8.56.178 / 192.168.0.129  | 4vCPUs | 8G / 100 GB | Ubuntu 20.04 server 64bit |

## 前置准备

- 运行系统更新 - 首先，请确保您的系统软件包是最新的。

  ```
  apt update
  apt upgrade
  ```

- 禁用swap

  ```
  swapoff -a
  ```

- Docker CE 安装

  ```
  curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
  ```

## 在Ubuntu 20.04上安装Kubernetes

提供Kubernetes运行时环境需要许多节点组件，这些组件需要安装在每个节点上。这些包括：

- `kubelet`：作为代理在每个节点上运行，并确保容器在Pod中运行。
- `kubeadm`：Bootstraps Kubernetes集群
- `kubectl`：用于对Kubernetes集群运行命令。

这些组件在默认的Ubuntu 20.04存储库中可用。因此，您需要安装Kubernetes仓库才能安装它们。

安装Kubernetes存储库GPG签名密钥

运行以下命令以安装Kubernetes回购GPG密钥。

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
```

在Ubuntu 20.04上安装Kubernetes存储库

接下来安装Kubernetes存储库:

```
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kurbenetes.list
```

安装Kubernetes组件:

```
apt update
apt install kubelet kubeadm kubectl
```

### 初始化Kubernetes控制平面

在安装了容器运行时以及Kubernetes组件之后，就该初始化Kubernetes主节点了。 Kubernetes主服务器负责维护集群的所需状态。

在主节点上运行以下命令以引导Kubernetes控制平面节点。

```
kubeadm init  --pod-network-cidr=10.244.0.0/16 
\ --apiserver-advertise-address=0.0.0.0
\ --apiserver-cert-extra-sans=119.8.237.150 # Public IP
```







```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.181:6443 --token te3unu.m7klax0l324yjouz \
    --discovery-token-ca-cert-hash sha256:b6407eb6841b83affca102a77b28eeb8040cb9319f95596acdbe52a7fd845e85
```



```
Name:         default-token-hbppp
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 6b5bdef6-78a7-4b7f-ba95-248717613c1d

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkhNQUdzRFRyZER3VjBWOXlhWFhKR1dKZnJYZXRoZk1RWHQyczJrNHF3TlEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0LXRva2VuLWhicHBwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2YjViZGVmNi03OGE3LTRiN2YtYmE5NS0yNDg3MTc2MTNjMWQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6ZGVmYXVsdCJ9.YVWpQp0E2laVVTm0ZzJqy2_8eESU2h5AjwE-C5FJglFiO9tIkQkqY27AxK3w062PYB4nZfTf4dkTK2zc2kNCNkDWM4USV11A3zMD5G_aJl4GucPLo45t8sThW6ds6hiLJflH7mIlCEQYfsz1-JqQf-vd3JmAKq4RnEeAFL9MYQ4dp2hMzOHgodPVg2qxM-eAFxl94NnzCkxP8oJoU9NzYNL_kOoZHx1JpJbHXEw1wPQ22HysbDUAuQIfr2LbA6AcIj3eNoc3sTJyRLRp2K90QB396pEIwpdt5GcZX0t4zovvXj4LN0e--h2o43-ptj-11qzy19BIJml-9n74BwbfYw
ca.crt:     1066 bytes
namespace:  20 bytes


Name:         kubernetes-dashboard-certs
Namespace:    kubernetes-dashboard
Labels:       k8s-app=kubernetes-dashboard
Annotations:  <none>

Type:  Opaque

Data
====


Name:         kubernetes-dashboard-csrf
Namespace:    kubernetes-dashboard
Labels:       k8s-app=kubernetes-dashboard
Annotations:  <none>

Type:  Opaque

Data
====
csrf:  256 bytes


Name:         kubernetes-dashboard-key-holder
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
pub:   459 bytes
priv:  1679 bytes


Name:         kubernetes-dashboard-token-2fl4x
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kubernetes-dashboard
              kubernetes.io/service-account.uid: 92a1fb3b-66d6-483f-9d53-0304ba5aeb2f

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkhNQUdzRFRyZER3VjBWOXlhWFhKR1dKZnJYZXRoZk1RWHQyczJrNHF3TlEifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi0yZmw0eCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjkyYTFmYjNiLTY2ZDYtNDgzZi05ZDUzLTAzMDRiYTVhZWIyZiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.kqTQrPWFZy2uiJLMqHwQ5Nng2k511muiat__DCg0D5l_Y3EPzK4LmT8IqRE6Jm-eN6eBtXybhCQJqqN5fZ59ky05gvfXOOcqwk06e9XnqEoda2vyB6WmgckSEav70KRL-kq9QLYiDQZyqg3CAEmTHoIVGiJ6Kv_Ea10V-oe77KtBGNGSWgiYu9tzkNXXKiEcSyxbQ-iVmURmpRV9K-d-0XImFKgL9kCfK73lu7gvS7Dl0iN4UqIBjuFS8Hl6eI9NctEg4xjEwcCWOmXjjtKwxEHo-xIedVRuOvCRvUpgSSNU_sHzeO4URgHqWCbvhfGYteEygBYQkA1KASu9BPlKdQ
ca.crt:     1066 bytes
namespace:  20 bytes
```



```
openssl genrsa -out dashboard.key 2048

openssl req -new -out dashboard.csr -key dashboard.key -subj '/CN=192.168.0.181'

wget https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml

```





