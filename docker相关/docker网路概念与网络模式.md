# docker的网络模式

1. docker单主机网络模式：

      - **host** : 

        docker不会为容器创建一个独有的network namespace，而是使用宿主机默认的网络命名空间，共享一个网络栈，表现为容器内和宿主机的IP一致，**host模式** 用于网络性能较高的场景，但安全性相对差一些。

      - **bridge** （创建容器默认使用此网络模式）:

        桥接模式，类似**VM-NAT**，docker进程启动时会创建一个**docker0**网桥，容器内的数据通过这个网卡设备与宿主机进行数据传输。

        docker会为容器创建独有的network namespace，也会为这个命名空间设置好虚拟网卡（通过**veth pair技术**）、路由、DNS、IP地址与iptables规则 (路由表)。

      - **none**

        **none**模式类似是桥接模式的一种特例，docker会为容器创建独有的network namespace，*但不会为这个命名空间准备虚拟网卡、ip地址、路由等，需要用户自己配置*。

      - **joined-container**

        容器共享模式，这种模式是**host**模式的一种延伸，一组容器共享一个network namespace；对外表现为他们有共同的IP地址，共享一个网络栈；**K8S**的**pod**就是使用的这一模式

2. docker多主机网络模式（k8s、swarm）：
   - **overlay**
   - **macvlan**
   - **flannel**

# docker网络查看

```bash
docker network -ls  #查看docker默认网络
```



