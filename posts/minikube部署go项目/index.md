# minikube下部署go项目


注意
本文假设已启动 minikube，具体操作请参考[Mac下安装minikube](https://ares0x.me/posts/mac%E4%B8%8B%E5%AE%89%E8%A3%85minikube/)

## 构建应用镜像

### 交互式

### 文件式

## 创建 Deployment
```shell
# 创建 Deployment
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
# 查看 Deployment
kubectl get deployments
# 查看 Pod
kubectl get pods
# 查看集群事件
kubectl get events
# 查看 kubectl 配置
kubectl config view
```

## 创建 service
```shell
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
# 查看 service
kubectl get services
```

## kubernetes 中的几个组件和概念
pod: 是由一个或多个为了管理和联网而绑定在一起的容器构成的组
service：
deployment：


## Reference
[Kubernetes](https://kubernetes.io/zh-cn/docs/tutorials/hello-minikube/)
