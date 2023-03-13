---
title: "docker核心原理简单总结"
date: 2023-03-13T15:14:21+08:00
draft: false
tags:
- docker
categories:
- docker
toc: true
---

> 大部分内容转载自：[https://lxkaka.wang/docker-principle/](https://lxkaka.wang/docker-principle/)

## Docker 的优势
Build once, Run anywhere 

- 应用标准化 无论什么语言开发的应用，我们都能用 dockerfile 和构建脚本方便的进行应用构建打包，代码库 + 构建 + registry 统一了 CI/CD 流程，也提升了效率
- 环境一致 由于应用和依赖全部构建成镜像，做到了一次构建多次交付，无论是开发，测试还是上线环境都是一致的。大大提高了开发效率
- 应用隔离 由于通过 docker 部署的应用，容器之间相互隔离，并且能按需分配资源。大大提高了运维效率和资源利用率
## 架构
Docker使用了 C/S 体系架构，Docker 客户端与 Docker 守护进程通信。
Docker 守护进程负责构建，运行和分发 Docker 容器。
Docker 客户端和守护进程可以在同一个系统上运行，也可以将Docker客户端连接到远程Docker守护进程。 我们日常在命令行的操作 docker build, docker push, docker pull, docke build 等等操作都是客户端通过 rest api 请求与 Docker 守护进程交互。 ![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1678689024965-7c93d225-47c4-4daa-a03f-5fff51f51212.png#averageHue=%2396ca4f&clientId=ufa0f394e-3d7e-4&from=paste&id=u111e017d&name=image.png&originHeight=674&originWidth=1292&originalType=url&ratio=2&rotation=0&showTitle=false&size=187694&status=done&style=none&taskId=u29437a26-3e81-4926-9812-c51ee41b6a9&title=)

## 实现原理
下面我们就介绍一下 Docker 在实现隔离，资源控制，文件系统等关键部分所采用的的技术。
### Namespace
namespace提供了一种内核级别隔离系统资源的方法，通过将系统的全局资源放在不同的 namespace 中，来实现资源隔离的目的。不同 namespace 的进程拥有相互隔离的系统资源。 这里指的资源隔离包含以下这些：

- Mount: 隔离文件系统挂载点
- UTS: 隔离主机名和域名信息
- IPC: 隔离进程间通信
- PID: 隔离进程的 ID
- Network: 隔离网络资源
- User: 隔离用户和用户组
```bash
ls -l /proc/23204/ns
```

### 网络模式
Docker 虽然可以通过命名空间创建一个隔离的网络环境，但是我们的应用需要对外提供服务，不能与外界进行通信是没有意义的。Docker 提供了多种网络模式来实现容器和外部的通信。 我们重点介绍一下 docker 默认的网络模式 bridge
 ![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1678689232435-15dfb268-a3da-40e4-9318-2012f9f3c947.png#averageHue=%23f4f4f4&clientId=ufa0f394e-3d7e-4&from=paste&height=342&id=ud959f5a2&name=image.png&originHeight=1154&originWidth=1430&originalType=url&ratio=2&rotation=0&showTitle=false&size=130166&status=done&style=none&taskId=uc46d86d9-9ea3-48dc-99bf-51940648dfa&title=&width=424)
守护进程会创建一对对等虚拟设备接口 veth pair，将其中一个接口设置为容器的 eth0 接口（容器的网卡），另一个接口放置在宿主机的命名空间中，以类似 vethxxx 这样的名字命名，从而将宿主机上的所有容器都连接到这个内部网络上。虚拟网桥的工作方式和物理交换机类似，这样主机上的所有容器就通过交换机连在了一个二层网络中。

- 容器访问外部网络，将源地址替换成了宿主机 ip，然后把报文转发出去
- 外部访问容器，docker实际是在 iptables 做了DNAT规则，实现端口转发功能

##### 其他网络模式

- Host 模式 和宿主机共用一个 Network Namespace。容器将不会虚拟出自己的网卡，配置自己的 IP 等，而是使用宿主机的 IP 和端口。但是，容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的
- Contanier 模式 这个模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的 IP，而是和一个指定的容器共享 IP、端口范围等

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1678689313887-dc4e295b-7b34-474f-a554-416664b7da64.png#averageHue=%23e2e3e0&clientId=ufa0f394e-3d7e-4&from=paste&height=347&id=u6202eae7&name=image.png&originHeight=694&originWidth=616&originalType=binary&ratio=2&rotation=0&showTitle=false&size=64309&status=done&style=none&taskId=uf67e8b2e-8500-4125-957b-16df9b82e8e&title=&width=308)

- None 模式 使用none模式，Docker 容器拥有自己的 Network Namespace，但是，并不为Docker容器进行任何网络配置。也就是说，这个 Docker 容器没有网卡、IP、路由等信息。需要我们自己为 Docker 容器添加网卡、配置 IP 等

### Cgroups
> 参考：[https://www.lixueduan.com/posts/docker/06-cgroups-1/](https://www.lixueduan.com/posts/docker/06-cgroups-1/)


通过 Linux Namespace 为新创建的进程隔离了文件系统、网络并与宿主机器之间的进程相互隔离，但是 Namespace 并不能够为我们提供物理资源上的隔离，比如 CPU 或者内存。所以 Docker 还借助了 Linux Cgroups 来达到上述目的。 CGroup 全称 Linux Control Group， 是 Linux 内核的一个功能，用来限制，控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等) 一组按照某种标准划分的进程，其表示了某进程组，Cgroups 中的资源控制都是以控制组为单位实现，一个进程可以加入到某个控制组。而资源的限制是定义在这个组上，简单点说，cgroup 的呈现就是一个目录带一系列的可配置文件。 

Cgroups 主要包括下面几部分：

- **cgroups本身**：cgroup 是对进程分组管理的一种机制，一个 cgroup 包含一组进程，并可以在这个 cgroup上增加 Linux subsystem 的各种参数配置，将一组进程和一组 subsystem 的系统参数关联起来
- **subsystem**： 一个 subsystem 就是一个内核模块，他被关联到一颗cgroup 树之后，就会在树的每个节点（进程组）上做具体的操作。subsystem 经常被称作"resource controller"，因为它主要被用来调度或者限制每个进程组的资源，但是这个说法不完全准确，因为有时我们将进程分组只是为了做一些监控，观察一下他们的状态，比如 perf_event subsystem。到目前为止，Linux 支持 12种 subsystem，比如限制 CPU 的使用时间，限制使用的内存，统计 CPU 的使用情况，冻结和恢复一组进程等
- **hierarchy**：一个 hierarchy 可以理解为一棵 cgroup 树，树的每个节点就是一个进程组，每棵树都会与零到多个 subsystem 关联。在一颗树里面，会包含 Linux 系统中的所有进程，但每个进程只能属于一个节点（进程组）。系统中可以有很多颗 cgroup 树，每棵树都和不同的 subsystem 关联，一个进程可以属于多颗树，即一个进程可以属于多个进程组，只是这些进程组和不同的 subsystem 关联。目前 Linux 支持 12种 subsystem，如果不考虑不与任何 subsystem关联的情况（systemd 就属于这种情况），Linux 里面最多可以建12颗cgroup树，每棵树关联一个 subsystem，当然也可以只建一棵树，然后让这棵树关联所有的 subsystem。当一颗 cgroup树不和任何 subsystem 关联的时候，意味着这棵树只是将进程进行分组，至于要在分组的基础上做些什么，将由应用程序自己决定，systemd 就是一个这样的例子

**3个部分间的关系**

- 系统在创建了新的 hierarchy 之后,系统中所有的进程都会加入这个 hierarchy 的cgroup根节点,这个 cgroup 根节点是 hierarchy 默认创建的。
- 一个 subsystem 只能附加到一个hierarchy上面。
- 一个 hierarchy 可以附加多个 subsystem。hierarchy和subsystem是一对多的关系，subsystem不能重复附加到多个hierarchy上。
- 一个进程可以作为多个 cgroup 的成员,但是这些 cgroup 必须在不同的 hierarchy 中。
- 一个进程fork出子进程时,子进程是和父进程在同一个 cgroup 中的,也可以根据需要将其移动到其他 cgroup

**三者的作用**

- cgroup 用于对进程进行分组
- hierarchy 则根据继承关系，将多个 cgroup 组成一棵树
- subsystem 则负责资源限制的工作，将 subsystem 和 hierarchy 绑定后，该 hierarchy 上的所有 cgroup 下的进程都会被 subsystem 给限制
   - 子 cgroup 会继承父 cgroup 的 subsystem，但是子 cgroup 却可以自定义自己的配置

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1678690486169-88eddecf-32df-4aed-aacd-e497b1210338.png#averageHue=%23868f65&clientId=ufa0f394e-3d7e-4&from=paste&height=518&id=uced71b47&name=image.png&originHeight=1036&originWidth=1861&originalType=binary&ratio=2&rotation=0&showTitle=false&size=114006&status=done&style=none&taskId=ufa517c16-9784-4cca-9e08-96c92b3def1&title=&width=930.5)
```bash
[work@wuchaoxin ~]$ cat /proc/cgroups
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	0	1	1
cpu	0	1	1
cpuacct	0	1	1
memory	0	1	1
devices	0	1	1
freezer	0	1	1
net_cls	0	1	1
blkio	0	1	1
perf_event	0	1	1
hugetlb	0	1	1
pids	0	1	1
net_prio	0	1	1
```
第一列：表示subsystem名。
第二列：表示关联到的cgroup树的ID，如果多个subsystem关联到同一颗cgroup树，那么它们的这个字段将一样。比如图中的cpuset、cpu和cpuacct。
第三列：表示subsystem所关联的cgroup树中进程组的个数，即树上节点的个数。

### 文件驱动
Docker 中的每一个镜像都是由一系列只读的层组成的，Dockerfile 中的每一个命令都会在已有的只读层上创建一个新的层。当镜像被 docker run 命令创建时就会在镜像的最上层添加一个可写的层，也就是容器层，所有对于运行时容器的修改其实都是对这个容器读写层的修改。容器和镜像的区别就在于，所有的镜像都是只读的，而每一个容器其实等于镜像加上一个可读写的层，也就是同一个镜像可以对应多个容器。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1678690696924-52ec05b6-7cdc-45fd-b377-b8c40a053fef.png#averageHue=%23a2c8d3&clientId=ufa0f394e-3d7e-4&from=paste&id=u37a673a4&name=image.png&originHeight=816&originWidth=1088&originalType=url&ratio=2&rotation=0&showTitle=false&size=277036&status=done&style=none&taskId=u5e14092b-4664-497a-9898-6ae3b85f55a&title=)

> 内容参考自：[http://www.ga1axy.top/index.php/archives/65/](http://www.ga1axy.top/index.php/archives/65/)

#### rootfs
在讲 overlay2 之前，我们需要先简单了解下什么是 rootfs：
rootfs 也叫 **根文件系统**，是 Linux 使用的最基本的文件系统，是内核启动时挂载的第一个文件系统，提供了根目录 /，根文件系统包含了系统启动时所必须的目录和关键性文件，以及使其他文件系统得以挂载所必要的文件。在根目录下有根文件系统的各个目录，例如 /bin、/etc、/mnt 等，再将其他分区挂载到 /mnt，/mnt 目录下就有了这个分区的各个目录和文件。
docker 容器中使用的同样也是 rootfs 这种文件系统，当我们通过 docker exec 命令进入到容器内部时也可以看到在根目录下有 /bin、/etc、/tmp 等目录，但是在 docker 容器中与 Linux 不同的是，在挂载 rootfs 后，docker deamon 会利用**联合挂载技术**在已有的 rootfs 上再挂载一个读写层，容器在运行过程中文件系统发生的变化只会在读写层进行修改，并通过 whiteout 文件隐藏只读层中的旧版本文件。

whiteout 概念存在于联合文件系统（UnionFS）中，代表某一类占位符形态的特殊文件，当用户文件夹与系统文件夹的共通部分联合到一个目录时（例如 bin 目录），用户可删除归属于自己的某些系统文件副本，但归属于系统级的原件仍存留于同一个联合目录中，此时系统将产生一份 whiteout 文件，表示该文件在当前用户目录中已删除，但系统目录中仍然保留。
#### 联合挂载技术
所谓联合挂载技术（Union Mount），就是将原有的文件系统中的不同目录进行**合并（merge）**，最后向我们呈现出一个合并后文件系统。在 overlay2 文件结构中，联合挂载技术通过联合三个不同的目录来实现：lower目录、upper目录和work目录，这三个目录联合挂载后得到merged目录

- lower目录：**只读层**，可以有多个，处于最底层目录
- upper目录：**读写层**，只有一个
- work目录：用于写时复制，工作基础目录，挂载后内容被清空，且在使用过程中其内容不可见
- merged目录：联合挂载后得到的**视图**，其中本身并没有实体文件，实际文件都在upper目录和lower目录中，在merged目录中对文件进行编辑，实际会修改upper目录中文件，而在upper目录与lower目录中修改文件，都会影响我们在merged目录中看到的结果

这种分层的逻辑是什么呢？ 这就是docker 里文件驱动的职责，负责镜像和容器的文件系统组织。
目前 docker 默认的文件驱动是 overlay2, 它是基于 Linux OverlayFS 的，下面这张图映射了 docker 容器的文件层级结构和 OverlayFS 的对应关系。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1678690696950-1a3ed567-f62d-45c9-8467-f6e3c79cb8a4.png#averageHue=%23e3e3e3&clientId=ufa0f394e-3d7e-4&from=paste&id=u43cd362e&name=image.png&originHeight=294&originWidth=1148&originalType=url&ratio=2&rotation=0&showTitle=false&size=245457&status=done&style=none&taskId=u2824e99d-361c-4ece-a05e-66404dffd74&title=)
OverlayFS 是一种堆叠文件系统，它依赖并建立在其它的文件系统之上，不直接参与磁盘空间结构的划分，仅将原来文件系统中不同目录和文件进行 merge。用户看到就是这个 merged 目录。这些被处理的每个目录都被称为层，视图统一的过程则称为联合挂载。多个目录进行层叠，肯定具有上下层关系，OverlayFS 将下层的目录称为lowerdir，上层的目录称为upperdir，被暴露的统一视图目录称为 merged。
#### 容器文件系统层级示例
```bash
root@lxkaka-server:~# docker inspect a2b1e73dda7f
...
"GraphDriver": {
    "Data": {
        "LowerDir": "/var/lib/docker/overlay2/77a2b28678d5406b8184e213a725bac51e4bb0132cb2c2c7928c9805bbeb57e3-init/diff:/var/lib/docker/overlay2/2c24c0bcddaf45a0442e2aa1f061e5a10ac0bb327519c4c7ddc11ecb85878bb1/diff:/var/lib/docker/overlay2/4c689dd9bd8baaff479edf0549e7888b754bf9a635d354e4a7f1531bf946f936/diff:/var/lib/docker/overlay2/84b84a32ceb721075356a9f9d4e8f8c3272a3a440f732a639993293a221236cd/diff:/var/lib/docker/overlay2/fd803c843f13047fd21fda7270a8bec065a4ba62310f95aa5524ec3156755e25/diff:/var/lib/docker/overlay2/fe43a13d9aa35f8b6038d26db6fa83bbd9a108fa59e9599e520505e9b028105d/diff:/var/lib/docker/overlay2/ec929233dc85f54a5b13e906dc1ebd20328fefdebfe2a411a06b6551c501928c/diff",
        "MergedDir": "/var/lib/docker/overlay2/77a2b28678d5406b8184e213a725bac51e4bb0132cb2c2c7928c9805bbeb57e3/merged",
        "UpperDir": "/var/lib/docker/overlay2/77a2b28678d5406b8184e213a725bac51e4bb0132cb2c2c7928c9805bbeb57e3/diff",
        "WorkDir": "/var/lib/docker/overlay2/77a2b28678d5406b8184e213a725bac51e4bb0132cb2c2c7928c9805bbeb57e3/work"
    },
    "Name": "overlay2"
},
...
```
我们可以看到 lowerdir 包含了多层，每个镜像层目录中包含了一个文件 link，文件内容则是当前层对应的短标识符；lower 文件指向了其所有的父层，其文件内容则是父层id的短标识符；镜像层的内容则存放在 diff 目录 lowerdir 某层示例
查看容器层
```bash
root@lxkaka-server:/var/lib/docker/overlay2/77a2b28678d5406b8184e213a725bac51e4bb0132cb2c2c7928c9805bbeb57e3# ls diff
root
root@lxkaka-server:/var/lib/docker/overlay2/77a2b28678d5406b8184e213a725bac51e4bb0132cb2c2c7928c9805bbeb57e3# ls merged/
bin  boot  data  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 进入到对应的容器
root@a2b1e73dda7f:/# ls
bin  boot  data  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```
我们可以看到容器的目录结构和 merged 是一致的。
#### overlayFS如何工作

1. 读：
- 如果文件在容器层（upperdir），直接读取文件
- 如果文件不在容器层（upperdir），则从镜像层（lowerdir）读取

2. 写：

- 首次写入： 如果在upperdir中不存在，overlay和overlay2执行copy_up操作，把文件从lowdir拷贝到upperdir，由于overlayfs是文件级别的（即使文件只有很少的一点修改，也会产生的copy_up的行为），后续对同一文件的在此写入操作将对已经复制到容器的文件的副本进行操作。值得注意的是，copy_up操作只发生在文件**首次写入**，以后都是只**修改副本**
- 删除文件和目录： 当文件在容器被删除时，在容器层（upperdir）创建**whiteout文件**，镜像层(lowerdir)的文件是不会被删除的，因为他们是只读的，但without文件会阻止他们显示
