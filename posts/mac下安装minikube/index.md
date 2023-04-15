# Mac下安装minikube


说明：
运行环境：MBP M1

minikube 是一种可以在本地轻松运行 Kubernetes minikube 在笔记本电脑上的虚拟机或 Docker 中运行单节 Kubernetes ，用于学习和日常开发。为此，记录一下 minikube 的安装过程。

## 安装 kubectl
kubectl 是 kubernetes 中用于集群管理的 CLI 工具。安装可参考 [kubectl 安装文档](https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-kubectl-on-macos) 进行安装。安装文档中提供了三种安装方式，任选一种即可。

使用**kubectl version --client**命令校验安装结果
```shell
$ ➜ kubectl version --client
WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:21:19Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"darwin/arm64"}
Kustomize Version: v5.0.1
```

## 安装 minikube
可以参考 [minikube 安装文档](https://minikube.sigs.k8s.io/docs/start/) 进行安装。在该文档中选择与自己相同的系统来获取安装命令。我的机器是 Mac M1，所以选择 
```shell
Operating system：macOS
Architecture：ARM64
Release type：Stable
Installer type：Homebrew
```
Release 和 Installer 可以根据自己的需求选择。

## 创建集群
使用**minikube start**即可创建集群，如果出现以下内容表示创建成功
```shell
$ ➜ minikube start
😄  minikube v1.30.1 on Darwin 13.0 (arm64)
✨  Automatically selected the docker driver. Other choices: virtualbox, ssh
📌  Using Docker Desktop driver with root privileges
❗  Local proxy ignored: not passing HTTP_PROXY=http://127.0.0.1:7890 to docker env.
❗  Local proxy ignored: not passing HTTPS_PROXY=http://127.0.0.1:7890 to docker env.
👍  Starting control plane node minikube in cluster minikube
🚜  Pulling base image ...
💾  Downloading Kubernetes v1.26.3 preload ...
    > preloaded-images-k8s-v18-v1...:  330.52 MiB / 330.52 MiB  100.00% 16.06 M
    > gcr.io/k8s-minikube/kicbase...:  336.39 MiB / 336.39 MiB  100.00% 9.44 Mi
🔥  Creating docker container (CPUs=2, Memory=3633MB) ...
❗  Local proxy ignored: not passing HTTP_PROXY=http://127.0.0.1:7890 to docker env.
❗  Local proxy ignored: not passing HTTPS_PROXY=http://127.0.0.1:7890 to docker env.
🌐  Found network options:
    ▪ http_proxy=http://127.0.0.1:7890
❗  You appear to be using a proxy, but your NO_PROXY environment does not include the minikube IP (192.168.49.2).
📘  Please see https://minikube.sigs.k8s.io/docs/handbook/vpn_and_proxy/ for more details
    ▪ https_proxy=http://127.0.0.1:7890
🐳  Preparing Kubernetes v1.26.3 on Docker 23.0.2 ...
    ▪ Generating certificates and keys ...
    ▪ Booting up control plane ...
    ▪ Configuring RBAC rules ...
```
此时，执行**docker ps**,可以看到启动了一个名为 minikube 的容器。
```shell
¥ ➜ docker ps
CONTAINER ID   IMAGE                                 COMMAND                  CREATED         STATUS         PORTS                                                                                                                                  NAMES
fb06965b2811   gcr.io/k8s-minikube/kicbase:v0.0.39   "/usr/local/bin/entr…"   6 minutes ago   Up 6 minutes   127.0.0.1:50875->22/tcp, 127.0.0.1:50876->2376/tcp, 127.0.0.1:50878->5000/tcp, 127.0.0.1:50879->8443/tcp, 127.0.0.1:50877->32443/tcp   minikube
```
再执行**kubectl cluster-info**可以验证集群是否配置正确
```shell
$ ➜ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:50879
CoreDNS is running at https://127.0.0.1:50879/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
使用**minikube dashboard**会打开一个浏览器页面，在此可以查看集群当前状态
