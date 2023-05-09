# 几个常用的网络工具


## Linux 下常用的网络工具

本文将会介绍常用的四种网络工具：netstat、ss、tcpdump和wireshark。这些工具可以帮助我们快速了解网络连接情况、查找网络延迟、以及排查网络问题等。这些工具不仅有助于监控和管理现有网络设备，同时也是设计和实施新网络时十分重要的工具集。
在使用这些工具时，需要根据具体的使用场景选择不同工具进行操作。例如，当需要定位网络连接问题时，可以使用 netstat 和 ss 这两种常用的网络连接查询工具；当需要抓包并查看网络数据流时，可以使用 tcpdump 和 wireshark 这两种网络流量分析工具。这些工具的使用方法和技巧因工具而异，但是它们都可以帮助我们深入了解网络。

### netstat
netstat 是一种常用的网络工具，通常用于诊断网络连接问题和查看当前系统的网络连接状态。它可以显示当前系统的网络和协议统计信息，包括协议、本地地址、外部地址、连接状态等。在实际网络管理和故障排除工作中，Netstat 是一种非常有用的工具。下面是关于 netstat 的更详细的介绍和使用指南：

```shell
# -a 显示所有套接字
# -t 表示显示 TCP 套接字
# -u 表示显示 UDP 套接字
# -l 表示只显示监听套接字
# -n 表示显示数字地址和端口(而不是名字)
# -p 表示显示进程信息
```
当我们使用 netstat -tn 或者 netstat -un 时，会展示如下指标
- Proto：协议
- Recv-Q：接收队列（缓冲）
- Send-Q：发送队列（缓冲）
- Local Address：本地地址和端口
- Foreign Address：对端地址和端口
- State：状态，通常情况下我们需要关注: LISTEN, ESTABLISHED, TIME_WAIT, CLOSE_WAIT 四种状态

#### 示例

1. 查找当前TCP连接中处于ESTABLISHED状态的连接，对于这些连接，如果它们的发送或接收数据的字节数大于0，则打印它们的信息
```shell
[root@VM-8-7-centos ~]# netstat -nt | xargs -L 1 | grep "ESTABLISHED" | awk '{if ($2>0 || $3>0) print $0}'
Proto Recv-Q Send-Q Local Address Foreign Address State 
tcp 0 0 10.0.8.7:46828 169.254.0.55:5574 ESTABLISHED
tcp 0 64 10.0.8.7:22 111.201.18.224:13787 ESTABLISHED
# 1. 使用netstat命令显示当前TCP连接的信息；
# 2. 将netstat的输出逐行传递给xargs命令；
# 3. xargs命令将每一行输出作为参数传递给grep命令，grep命令过滤出TCP状态为ESTABLISHED的连接；
# 4. grep命令的输出再次逐行传递给awk命令；
# 5. awk命令对每一行进行处理，如果第2列或第3列的值大于0，则打印该行输出。
```

2. 查看端口占用
```shell
[root@VM-8-7-centos ~]# netstat -tunlp | grep 80 # grep 后 跟端口号
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      10941/nginx: master 
```

#### 缺点
1. netstat 通过遍历 proc 来获取socket信息，当 socket 很多时，netstat 会很慢
2. netstat 已经停止维护，且在大量连接的情况下查询速度很慢，建议使用 ss 来替代

### ss
SS（Socket Statistics）是一种高级的网络工具，用于显示关于套接字的更多信息，包括 TCP 连接、UDP 协议和 Unix 域套接字的信息等。它也可以显示其他一些网络状态信息，例如网络接口的统计信息、内核 IP 路由表、Linux 内核内存信息等。SS 可以作为 Linux 系统中替代 netstat 的工具，更加高效快捷。下面是关于 SS 的更详细的介绍和使用指南：

ss 的使用与 netstat 相似。
```shell
# -a 显示所有套接字
# -l 表示只显示处于监听状态套接字
# -t 表示显示 TCP 套接字
# -u 表示显示 UDP 套接字
# -4 显示 ipv4 套接字
# -6 显示 ipv6 套接字
# -n 表示显示数字地址和端口(而不是名字)
# -p 表示显示套接字进程信息
# -s 展示详细信息
```

#### 示例
1. 查找当前 TCP 连接中处于 ESTABLISHED 状态的连接，对于这些连接，如果它们的内存使用量大于0，将它们输出到屏幕上

```shell
[root@VM-8-7-centos ~]# ss -mn | xargs -L 1 | grep "tcp EST" | awk '{ if($3>0 || $4>0) print $0 }'
tcp ESTAB 0 64 10.0.8.7:22 111.201.18.224:13787 skmem:(r0,rb369280,t0,tb46080,f5888,w2304,o0,bl0,d2261)
# 1. 使用ss命令显示当前网络连接的TCP内存使用情况；
# 2. 将ss命令的输出逐行传递给xargs命令；
# 3. xargs命令将每一行输出作为参数传递给grep命令，grep命令过滤出TCP状态为ESTABLISHED的连接；
# 4. grep命令的输出再次逐行传递给awk命令；
# 5. awk命令对每一行进行处理，如果第3列或第4列的值大于0，则打印该行输出。
```

ss 使用 NETLINK 与内核 tcp_diag 模块通信获取 socket 信息，速度相较于 netstat 会快很多。

### tcpdum

```shell
# 使用 -i 可以指定抓取网卡。在不指定网卡的情况下，tcpdump 会默认抓取所有网卡中第一个
tcpdump -i en0

# 使用 host 可以抓取特定 ip 或 域名的内容。host 后可以跟域名或者 ip，在使用 host 后，tcmdump 会监听本机和 host 后地址的所有通信包。如果想要获取目标地址特定方向的包，可以在 host 前添加 src/dst 分别表示，来源是目标地址/发送给目标地址。
tcpdump host www.baidu.com/10.31.100.121

# 抓取端口号为 8080 的内容
tcpdump port 8080

# 监听 80 端口的所有 tcp 包
tcpdump tcp port 80 

# 监听 80 端口的所有 udp 包
tcpdump udp port 80

# 使用 -c 可以指定抓包数量。如下，抓取 100 个包
tcpdump -c 100

# 使用 -t 可以去除抓包内容中的时间显示
tcpdump -t 

# 将文件保存为可供 wireshark 解析的 .cap 格式
tcpdump -w ./shark.cap 
tcpdump -w ./shark.pcap
```
示例
```shell
tcpdump tcp port 3306
```

### wireshark
wireshark 是集抓包和图形界面分析为一体的网络工具。wireshark 的抓包能力很强大，不过多数情况下使用 tcpdump 抓包，并将抓包的文件导入 wireshark 中进行分析。

## 注意
1. Recv-Q 和 Send-Q
在使用 netstat 和 ss 抓取 tcp 包时均可以看到 Recv-Q 和 Send-Q
如果 netstat 查询到的连接不是 Listen 状态的话，Recv-Q 就是指收到的数据还在缓存中，还没被进程读取，这个值就是还没被进程读取的 bytes；而 Send 则是发送队列中没有被远程主机确认的 bytes 数。
```
Recv-Q
Established: The count of bytes not copied by the user program connected to this socket.
Listening: Since Kernel 2.6.18 this column contains the current syn backlog.

Send-Q
Established: The count of bytes not acknowledged by the remote host.
Listening: Since Kernel 2.6.18 this column contains the maximum size of the syn backlog.
```
2. 连接的状态
![连接状态](/images//socket_stat.png)

## Reference
- [关于 Recv-Q 和 Send-Q 的说明](https://github.com/moooofly/MarkSomethingDown/blob/master/Linux/%E5%85%B3%E4%BA%8E%20Recv-Q%20%E5%92%8C%20Send-Q%20%E7%9A%84%E8%AF%B4%E6%98%8E.md)
- [端口状态说明](https://www.cnblogs.com/huangjianping/p/8012405.html)

