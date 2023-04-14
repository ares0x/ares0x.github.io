# Docker简易操作


Docker 是目前最流行的轻量级容器化解决方案，它可以提供一致的运行环境，并解决各种运维问题。如何在 Docker 中部署 Go 语言项目，已成为了很多人关心的问题。本文介绍如何在 Docker 中部署 Go 语言项目，包括如何写 Dockerfile 和构建镜像的过程，以及使用 Docker 部署 Go 语言项目可能存在的问题和优化方法。

## 为什么要用 Docker
1. 更高效地利用资源。虚拟化技术需要虚拟化一个操作系统，容器技术基于 namesapce，cgroup，不需要虚拟化更多的东西（Guest OS 等等）；
2. 更快速的启动时间；
3. 一致的运行环境；
4. 持续交付和部署；
5. 更轻松地迁移，维护和部署

## Docker 常用操作

### 容器操作
* 启动
```shell
docker run
# -it 交互
# -d 后台运行,如果不加 -d，从容器退出后该容器会停止运行
# -p 端口映射
# -v 硬盘挂载
```
eg:
```shell
docker run centos -d # 启动一个 centos 容器，并进入容器内部
```

* 停止容器
```shell
docker stop xxx 
```

* 启动已停止容器
```shell
docker start xxx
```

* 查看容器进程
```shell
docker ps
# -a 查看所有容器（包括已停止的容器）
``` 
举例
```shell
(base) ➜  ~ docker ps #显示已经启动的容器
CONTAINER ID   IMAGE     COMMAND                  CREATED       STATUS      PORTS                    NAMES
1d66faf0f9e9   redis     "docker-entrypoint.s…"   7 weeks ago   Up 2 days   0.0.0.0:6379->6379/tcp   single-redis

(base) ➜  ~ docker ps -a #显示所有的容器
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                     PORTS                                                                                              NAMES
618ca42d2cf0   centos         "bash"                   6 minutes ago   Exited (0) 5 minutes ago                                                                                                      youthful_borg
4b8ed64b6bff   rabbitmq       "docker-entrypoint.s…"   4 weeks ago     Exited (255) 2 weeks ago   4369/tcp, 0.0.0.0:5672->5672/tcp, 5671/tcp, 15691-15692/tcp, 25672/tcp, 0.0.0.0:15672->15672/tcp   rabbit
083cf1fdd1e8   423da140c8c0   "/entrypoint.sh mysq…"   6 weeks ago     Exited (255) 7 days ago    0.0.0.0:3306->3306/tcp, 33060-33061/tcp                                                            mysql
6841b7c88f95   e082d8ac7e5e   "/bin/tini -- /usr/l…"   7 weeks ago     Exited (255) 3 weeks ago   0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp                                                     elasticsearch
1d66faf0f9e9   redis          "docker-entrypoint.s…"   7 weeks ago     Up 2 days                  0.0.0.0:6379->6379/tcp                                                                             single-redis
```

* 查看容器细节
```shell
docker inspect xxx
```

举例：
```shell
(base) ➜  ~ docker inspect redis
[
    {
        "Id": "sha256:e79ba23ed43baa22054741136bf45bdb041824f41c5e16c0033ea044ca164b82",
        "RepoTags": [
            "redis:latest"
        ],
        "RepoDigests": [
            "redis@sha256:6a59f1cbb8d28ac484176d52c473494859a512ddba3ea62a547258cf16c9b3ae"
        ],
        "Parent": "",
        "Comment": "",
        "Created": "2023-02-09T08:14:47.884678524Z",
        "Container": "634f20363cc8a5fd95103f0fc002e10de0884cb5d238359ab676bf3054b4f09c",
        "ContainerConfig": {
            "Hostname": "634f20363cc8",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "6379/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.16",
                "REDIS_VERSION=7.0.8",
                "REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-7.0.8.tar.gz",
                "REDIS_DOWNLOAD_SHA=06a339e491306783dcf55b97f15a5dbcbdc01ccbde6dc23027c475cab735e914"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"redis-server\"]"
            ],
            "Image": "sha256:781198024e5efdfe9eaf59628844640981f1a9af62109f26e5f33be0417d3172",
            "Volumes": {
                "/data": {}
            },
            "WorkingDir": "/data",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "20.10.17",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "6379/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "GOSU_VERSION=1.16",
                "REDIS_VERSION=7.0.8",
                "REDIS_DOWNLOAD_URL=http://download.redis.io/releases/redis-7.0.8.tar.gz",
                "REDIS_DOWNLOAD_SHA=06a339e491306783dcf55b97f15a5dbcbdc01ccbde6dc23027c475cab735e914"
            ],
            "Cmd": [
                "redis-server"
            ],
            "Image": "sha256:781198024e5efdfe9eaf59628844640981f1a9af62109f26e5f33be0417d3172",
            "Volumes": {
                "/data": {}
            },
            "WorkingDir": "/data",
            "Entrypoint": [
                "docker-entrypoint.sh"
            ],
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "arm64",
        "Variant": "v8",
        "Os": "linux",
        "Size": 111377329,
        "VirtualSize": 111377329,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/2f4acdea165632b50c6dbaa35179980e705e37bb8d4521c7ebeb3b070c6a3757/diff:/var/lib/docker/overlay2/0ed21db28f0324dc21e53ecbee6b4bb8076d0509c4e8b623df16277975646e86/diff:/var/lib/docker/overlay2/8ea40ec0b53a579f8e22aebfd27dcae323868f247c3ccaf98ea06d6bb4f5feae/diff:/var/lib/docker/overlay2/4df450e39662acc93e109568a402190518c2ceffee550484851cc96af4fb8a69/diff:/var/lib/docker/overlay2/8b1d17876b46a9820d28a0a92955dc5a34c73225b8f4ff2025143aec14325a5a/diff",
                "MergedDir": "/var/lib/docker/overlay2/5532b0dabadd8a7e4df82e6e8d7f8baa54f844b5c451be99ae8be57ccac77a7e/merged",
                "UpperDir": "/var/lib/docker/overlay2/5532b0dabadd8a7e4df82e6e8d7f8baa54f844b5c451be99ae8be57ccac77a7e/diff",
                "WorkDir": "/var/lib/docker/overlay2/5532b0dabadd8a7e4df82e6e8d7f8baa54f844b5c451be99ae8be57ccac77a7e/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:a49c6ceb5b3a7ffa366916db7ef211391b4bf093f14091b5d9e21d53297695a3",
                "sha256:39a2097041f293c4d3c8c2d31882ab5bf33105610752c92cdc1e2ad27eb23136",
                "sha256:557b6c60acd513d3f0e64d3c670700252c53e26f83d07f7a0598bdb27fc11e34",
                "sha256:5c44256a056b2bfaf31c3f184dd62cc8fbdcf856e4e8752350820417e4add8b2",
                "sha256:35db71540d1a8eb9a52c026a0a1bfe8569bc53e7af92db3ad41cb59850656836",
                "sha256:73cc6bdd5f25686aee85890890b6db2b40c553aeac7dc2f026208b19550844d2"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

### 镜像操作


```shell
docker images # 查看镜像
```


## 部署一个 Go 项目

### 编写一个 webserver
为了更好地演示如何使用 Docker 部署 Go 语言项目，我们来实现一个简单的 Web 服务。
```go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello, Docker!")
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

### 编写 Dockerfile
编写 Dockerfile 是 Docker 部署项目的重要环节。我们需要定义 Dockerfile 来描述如何构建镜像，并在其中配置一些环境变量、安装 Go、复制 Go 代码等。下面是一个示例：

```Dockerfile
# 基于golang镜像创建一个新镜像
FROM ubuntu

ADD bin/amd64/webserver /webserver

# 设置对外暴露的端口
EXPOSE 8080

# 运行 Go 应用程序
ENTRYPOINT /webserver
```

在上面的示例中，我们使用 ununtu 作为基础镜像。声明容器对外发布的端口号为 8080。最后，我们调用 ENTRYPOINT 指令来运行 Go 应用程序。因为我们是将二进制文件放置到容器中，所以容器中不需要有 golang 环境。我会在后续的文章中介绍几种不同的构建方式。

### 构建镜像

使用 Dockerfile 构建自定义镜像是一个非常简单的过程。首先，我们需要进入到包含 Dockerfile 的目录，然后运行以下命令：
```shell
docker build -t ares0x/webserver:${tag} .
```
其中，-t 参数用于给镜像打标签，即为镜像指定一个名称。在这个示例中，我们将镜像命名为 my-go-app，后面的点表示使用当前目录中的 Dockerfile。

构建成功后，我们就可以使用以下命令运行该镜像：
```shell
docker run -p 8080:8080 ede5c3638067 #ede5c3638067 为镜像的id
```

此时浏览器打开**http://127.0.0.1:8080/**，会返回**Hello, Docker!**

### 使用 Makefile
创建一个 Makefile 文件，并将我们上述使用到的命令都加进去
```Makefile
export tag=v0.1

root:
	export root=github.com/ares0x/practice

build:
	echo "building webserver binary"
	mkdir -p bin/amd64
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o bin/amd64

release:
	echo "building webserver container"
	docker build -t ares0x/webserver:${tag} .
```
这样，我们就可以通过 **make build** 来编译指定平台的二进制文件，通过 **make release** 来构建镜像。

详细内容请参考：https://github.com/ares0x/practice/tree/main/depoly/webserver
