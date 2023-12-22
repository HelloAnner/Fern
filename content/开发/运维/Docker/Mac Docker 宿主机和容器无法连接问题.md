Docker for Mac 自诞生以来就一直有一个问题，那就是在宿主机上看不到 `docker0`，无法访问容器所在的网络，也就是说不能 `ping` 通 Docker 给 Container 所分配的 IP 地址。关于这个问题，官方文档有描述：[_Known limitations, use cases, and workarounds_](https://docs.docker.com/docker-for-mac/networking/#there-is-no-docker0-bridge-on-macos)

![[attachments/9bc070bbf89532353de390577081e9c7_MD5.jpeg]]

对于 `docker run` 启动的 Container 来说，通常会通过 `-p` 参数映射相应的服务端口，一般不会遇到要直接访问容器 IP 的情况。

但是当我们在 docker 中运行多个微服务并想进行本地调试的时候，在 Mac 上却没法实现

### Docker For Mac 原理

Docker 是利用 Linux 的 Namespace 和 Cgroups 来实现资源的隔离和限制，容器共享宿主机内核，所以 Mac 本身没法运行 Docker 容器。不过不支持不要紧，我们可以跑虚拟机，最早还没有 Docker for Mac 的时候，就是通过 `docker-machine` 在 Virtual Box 或者 VMWare 直接起一个 Linux 的虚拟机，然后在主机上用 Docker Client 操作虚拟机里的 Docker Server。

![[attachments/b4bf9c978e2d090daaf0fd5605ff8978_MD5.jpeg]]

Docker for Mac 也是在本地跑了一个虚拟机来运行 Docker，不过 Hypervisor 采用的是 [xhyve](https://github.com/mist64/xhyve)，而 xhyve 又基于 Mac 自带的虚拟化方案 [Hypervisor.framework](https://developer.apple.com/library/mac/documentation/DriversKernelHardware/Reference/Hypervisor/index.html)，虚拟机里运行的发行版是 Docker 自己打包的 [LinuxKit](https://github.com/linuxkit/linuxkit)，之前用的发行版好像是 Alpine Linux。

总而言之就是 Docker for Mac 跑的这个虚拟机非常轻量级，性能也会更好。

---

### Docker Connector 方案

[GitHub - wenjunxiao/mac-docker-connector: The connector provides the ability for the mac/windows host to directly access the docker container](https://github.com/wenjunxiao/mac-docker-connector)

主要方式在宿主的macOS和Docker的Hypervisor之间建立一个VPN

```
+---------------+          +--------------------+
|               |          | Hypervisor/Hyper-V |
| macOS/Windows |          |  +-----------+     |
|               |          |  | Container |     |
|               |   vpn    |  +-----------+     |
|   VPN Client  |<-------->|   VPN Server       |
+---------------+          +--------------------+
```

但是宿主的macOS无法直接访问Hypervisor，VPN服务容器需要使用`host`以便与Hypervisor在同一网络环境中，必须使用一个转发容器（比如`socat`)导出端口到macOS，然后转发到VPN服务。考虑到VPN连接的双工的，因此我们可以把VPN服务和客户端反转一下，变成下面的结构

```
+---------------+          +--------------------+
|               |          | Hypervisor/Hyper-V |
| macOS/Windows |          |  +-----------+     |
|               |          |  | Container |     |
|               |   vpn    |  +-----------+     |
| VPN Server    |<-------->|   VPN Client       |
+---------------+          +--------------------+
```

尽管如此, 我们需要做更多额外的工作来使用openvpn，比如证书、配置等。 这对于只是通过IP访问容器的需求来说，这些工作略显麻烦。 我们只需要建立一个连接通道，无需证书，也可以无需客户端

```
+---------------+          +--------------------+
|               |          | Hypervisor/Hyper-V |
| macOS/Windows |          |  +-----------+     |
|               |          |  | Container |     |
|               |   udp    |  +-----------+     |
| TUN Server    |<-------->|   TUN Client       |
+---------------+          +--------------------+
```

鉴于Docker官方文档[Docker and iptables](https://docs.docker.com/network/iptables/)中描述那样, 两个子网之间的互通性有时也是需要的，因此还可以通过`iptables`来提供两个子网之间的互相连接

```
+-------------------------------+ 
|      Hypervisor/Hyper-V       | 
| +----------+     +----------+ | 
| | subnet 1 |<--->| subnet 2 | |
| +----------+     +----------+ |
+-------------------------------+
```

