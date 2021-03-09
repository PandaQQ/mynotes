# Huawei Cloud CCE Notes:

> 本文主要记录一些Huawei Cloud CCE 的使用笔记



### 1. Mac OS 连接到 CCE Service:

```
# 获取到kubeconfig.json之后
mv -f kubeconfig.json $HOME/.kube/config
kubectl config use-context external
kubectl cluster-info
kubctl get nodes
```




### 2. 

