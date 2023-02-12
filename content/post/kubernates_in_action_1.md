---
title: "《Kubernetes in Action》读书笔记第一章：Kubernetes介绍"
date: 2023-02-12T17:08:21+08:00
draft: false
tags:
- kubernates
- docker
categories:
- kubernates
toc: true
---

## 1.从单体应用到微服务

运行一个单体应用，通常需要一台能为整个应用提供足够资源的高性能服务器。为了应对不断增长的系统负荷，我们需要通过增加CPU、内存或其他系统资源的方式来对服务器做垂直扩展，或者增加更多的跑这些应用程序的服务器的做水平扩展。

垂直扩展不需要应用程序做任何变化，但是成本很快会越来越高，并且通常会有瓶颈。如果是水平扩展，就可能需要应用程序代码做比较大的改动，有时候甚至是不可行的，比如系统的一些组件非常难于甚至不太可能去做水平扩展（像关系型数据库）。如果单体应用的任何一个部分不能扩展，整个应用就不能扩展，除非我们想办法把它拆分开。

这些问题迫使我们将复杂的大型单体应用，拆分为小的可独立部署的微服务组件。每个微服务以独立的进程运行，并通过简单且定义良好的接口（API）与其他的微服务通信。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676190017059-a4abf2f9-3155-4913-a529-bbe2bc1acac6.png#averageHue=%23eeeeee&clientId=u634b8aab-bf70-4&from=paste&height=389&id=u23b70b60&name=image.png&originHeight=486&originWidth=873&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=182514&status=done&style=none&taskId=ue2b47874-d2c1-4aa7-a778-f848ab1a265&title=&width=698.4)



像大多数情况一样，微服务也有缺点。当组件数量增加时，部署相关的决定就变得越来越困难。因为不仅组件部署的组合数在增加，而且组件间依赖的组合数也在以更大的因素增加。

部署动态链接的应用需要不同版本的共享库，或者需要其他特殊环境，在生产服务器部署并管理这种应用很快会成为运维团队的噩梦。需要在同一个主机上部署的组件数量越大，满足这些组件的所有需求就越难。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1675873518987-07edd6d4-5f9f-4ce4-af49-4b5eb0f3ca0a.png#averageHue=%23e8e8e8&clientId=uec975c62-93ff-4&from=paste&height=420&id=u3681198c&name=image.png&originHeight=525&originWidth=932&originalType=binary&ratio=1&rotation=0&showTitle=false&size=279487&status=done&style=none&taskId=u04795bad-2827-4a1e-8131-49c0d848bbf&title=&width=745.6)

不管你同时开发和部署多少个独立组件，开发和运维团队总是需要解决的一个最大的问题是程序运行环境的差异性，这种巨大差异不仅存在于开发环境与生产环境之间，甚至存在于各个生产机器之间。另外一个无法避免的事实是生产机器的环境会随着时间的推移而变化。这些差异性存在于从硬件到操作系统再到每台机器的可用库上。

生产环境是由运维团队管理的，而开发者常常比较关心他们自己的开发环境。这两组人对系统管理的理解程度是不同的，这个理解偏差导致两个环境的系统有较大的差异，系统管理员更重视保持系统更新最近的安全补丁，而大多数开发者则并不太关心。

## 2.为应用程序提供一个一致的环境

在最近几年中，我们看到了应用在开发流程和生产运维流程中的变化。在过去，开发团队的任务是创建应用并交付给运维团队，然后运维团队部署应用并使它运行。但是现在，公司都意识到，让同一个团队参与应用的开发、部署、运维的整个生命周期更好这意味着开发者、QA和运维团队彼此之间的合作需要贯穿整个流程。这种实践被称为DevOps。

为了频繁地发布应用，就需要简化你的部署流程。理想的状态是开发人员能够自己部署应用上线，而不需要交付给运维人员操作。但是，部署应用往往需要具备对数据中心底层设备和硬件架构的理解。开发人员却通常不知道或者不想知道这些细节。

正如你所看到的，Kubernetes能让我们实现所有这些想法。通过对实际硬件做抽象，然后将自身暴露成一个平台，用于部署和运行应用程序。它允许开发者自己配置和部署应用程序，而不需要系统管理员的任何帮助，让系统管理员聚焦于保持底层基础设施运转正常的同时，不需要关注实际运行在平台上的应用程序。

## 3. 容器技术

和虚拟机比较，容器更加轻量级，它允许在相同的硬件上运行更多数量的组件。主要是因为每个虚拟机需要运行自己的一组系统进程，这就产生了除组件进程消耗以外的额外计算资源损耗。从另一方面说，一个容器仅仅是运行在宿主机上被隔离的单个进程，仅消耗应用容器消耗的资源，不会有其他进程的开销。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1675873968367-bcdaa7e6-cfc5-4e57-83b1-f3febf90ab79.png#averageHue=%23e9e9e9&clientId=u6966e15c-73fc-4&from=paste&height=452&id=ud67ac539&name=image.png&originHeight=565&originWidth=925&originalType=binary&ratio=1&rotation=0&showTitle=false&size=316004&status=done&style=none&taskId=ud175c4dc-1f72-4df1-8e72-64679895710&title=&width=740)

当你在一台主机上运行三个虚拟机的时候，你拥有了三个完全分离的操作系统，它们运行并共享一台裸机。在那些虚拟机之下是宿主机的操作系统与一个管理程序，它将物理硬件资源分成较小部分的虚拟硬件资源，从而被每个虚拟机里的操作系统使用。运行在那些虚拟机里的应用程序会执行虚拟机操作系统的系统调用，然后虚拟机内核会通过管理程序在宿主机上的物理来CPU执行x86指令。

注意存在两种类型的管理程序。第一种类型的管理程序不会使用宿主机OS，而第二种类型的会。

多个容器则会完全执行运行在宿主机上的同一个内核的系统调用，此内核是唯一一个在宿主机操作系统上执行x86指令的内核。CPU也不需要做任何对虚拟机能做那样的虚拟化（如图1.5所示）。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1675874057823-a417b525-e975-4bba-9f63-a0491c861af7.png#averageHue=%23e9e9e9&clientId=u6966e15c-73fc-4&from=paste&height=893&id=u941559dc&name=image.png&originHeight=1116&originWidth=822&originalType=binary&ratio=1&rotation=0&showTitle=false&size=348514&status=done&style=none&taskId=u5c7a7380-c6eb-4445-8add-c1e1b06170c&title=&width=657.6)

那容器到底是怎样隔离它们的。有两个机制可用：

第一个是Linux命名空间，它使每个进程只看到它自己的系统视图（文件、进程、网络接口、主机名等）；

第二个是Linux控制组（cgroups），它限制了进程能使用的资源量（CPU、内存、网络带宽等）。

用Linux命名空间隔离进程默认情况下，每个Linux系统最初仅有一个命名空间。所有系统资源（诸如文件系统、用户ID、网络接口等）属于这一个命名空间。但是你能创建额外的命名空间，以及在它们之间组织资源。对于一个进程，可以在其中一个命名空间中运行它。

进程将只能看到同一个命名空间下的资源。当然，会存在多种类型的多个命名空间，所以一个进程不单单只属于某一个命名空间，而属于每个类型的一个命名空间。存在以下类型的命名空间：Mount（mnt）ProcessID（pid）Network（net）Inter-processcommunicaion（ipd）UTSUserID（user）每种命名空间被用来隔离一组特定的资源。

现在你应该已经了解命名空间是如何隔离容器中运行的应用的。限制进程的可用资源另外的隔离性就是限制容器能使用的系统资源。这通过cgroups来实现。cgroups是一个Linux内核功能，它被用来限制一个进程或者一组进程的资源使用。一个进程的资源（CPU、内存、网络带宽等）使用量不能超出被分配的量。这种方式下，进程不能过分使用为其他进程保留的资源，这和进程运行在不同的机器上是类似的。



Docker是第一个使容器成为主流的容器平台。Docker本身并不提供进程隔离，实际上容器隔离是在Linux内核之上使用诸如Linux命名空间和cgroups之类的内核特性完成的，Docker仅简化了这些特性的使用。

尽管容器技术已经出现很久，却是随着Docker容器平台的出现而变得广为人知。Docker是第一个使容器能在不同机器之间移植的系统。它不仅简化了打包应用的流程，也简化了打包应用的库和依赖，甚至整个操作系统的文件系统能被打包成一个简单的可移植的包，这个包可以被用来在任何其他运行Docker的机器上使用。

例如，如果你用整个红帽企业版Linux（RHEL）的文件打包了你的应用程序，不管在装有Fedora的开发机上运行它，还是在装有Debian或者其他Linux发行版的服务器上运行它，应用程序都认为它运行在RHEL中。只是内核可能不同。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1675874638346-b8492e88-886e-4bd0-be77-a5be7e0653c4.png#averageHue=%23e4e4e4&clientId=u6966e15c-73fc-4&from=paste&height=881&id=u7b2d15f5&name=image.png&originHeight=1101&originWidth=760&originalType=binary&ratio=1&rotation=0&showTitle=false&size=385145&status=done&style=none&taskId=uac654b5a-3af1-4d56-bb93-d0035995437&title=&width=608)



前面已经说过Docker镜像由多层构成。不同镜像可能包含完全相同的层，因为这些Docker镜像都是基于另一个镜像之上构建的，不同的镜像都能使用相同的父镜像作为它们的基础镜像。这提升了镜像在网络上的分发效率，当传输某个镜像时，因为相同的层已被之前的镜像传输，那么这些层就不需要再被传输。

层不仅使分发更高效，也有助于减少镜像的存储空间。每一层仅被存一次，当基于相同基础层的镜像被创建成两个容器时，它们就能够读相同的文件。但是如果其中一个容器写入某些文件，另外一个是无法看见文件变更的。因此，即使它们共享文件，仍然彼此隔离。这是因为容器镜像层是只读的。容器运行时，一个新的可写层在镜像层之上被创建。容器中进程写入位于底层的。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676191006849-7bf3c5d2-c0db-4f10-9d02-d40ceb44d9c1.png#averageHue=%23f4f3f3&clientId=u634b8aab-bf70-4&from=paste&height=372&id=uafb902c6&name=image.png&originHeight=465&originWidth=880&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=171928&status=done&style=none&taskId=uef29aaab-cc50-4e59-8dc7-44405d72e5e&title=&width=704)

理论上，一个容器镜像能运行在任何一个运行Docker的机器上。但有一个小警告——一个关于运行在一台机器上的所有容器共享主机Linux内核的警告。如果一个容器化的应用需要一个特定的内核版本，那它可能不能在每台机器上都工作。如果一台机器上运行了一个不匹配的Linux内核版本，或者没有相同内核模块可用，那么此应用就不能在其上运行。

虽然容器相比虚拟机轻量许多，但也给运行于其中的应用带来了一些局限性。虚拟机没有这些局限性，因为每个虚拟机都运行自己的内核。

还不仅是内核的问题。一个在特定硬件架构之上编译的容器化应用，只能在有相同硬件架构的机器上运行。不能将一个x86架构编译的应用容器化后，又期望它能运行在ARM架构的机器上。你仍然需要一台虚拟机来做这件事情。

## 4. Kubernetes介绍  

主节点，它承载着Kubernetes控制和管理整个集群系统的控制面板工作节点，它们运行用户实际部署的应用

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1675875307985-0679e6ac-4134-45b6-99b4-a9f8fc6df43e.png#averageHue=%23f1f1f1&clientId=u6966e15c-73fc-4&from=paste&height=358&id=uc4decf66&name=image.png&originHeight=448&originWidth=903&originalType=binary&ratio=1&rotation=0&showTitle=false&size=120585&status=done&style=none&taskId=u56d6638c-4779-46ff-8d65-1d1dd6adce0&title=&width=722.4)

控制面板控制面板用于控制集群并使它工作。它包含多个组件，组件可以运行在单个主节点上或者通过副本分别部署在多个主节点以确保高可用性。这些组件是：

KubernetesAPI服务器，你和其他控制面板组件都要和它通信

Scheculer，它调度你的应用（为应用的每个可部署组件分配一个工作节点）

ControllerManager，它执行集群级别的功能，如复制组件、持续跟踪工作节点、处理节点失败等

etcd，一个可靠的分布式数据存储，它能持久化存储集群配置控制面板的组件持有并控制集群状态，但是它们不运行你的应用程序。这是由工作节点完成的。

工作节点工作节点是运行容器化应用的机器。运行、监控和管理应用服务的任务是由以下组件完成的：Docker、rtk或其他的容器类型Kubelet，它与API服务器通信，并管理它所在节点的容器KubernetesServiceProxy（kube-proxy），它负责组件之间的负载均衡网络流量

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676191160468-3df4e608-1f07-4d10-a4ad-9cc172b53447.png#averageHue=%23ebebeb&clientId=u634b8aab-bf70-4&from=paste&height=494&id=u4ede3f27&name=image.png&originHeight=617&originWidth=821&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=254613&status=done&style=none&taskId=u2b7214e0-1d95-4082-bfe5-191d59b579f&title=&width=656.8)



为了在Kubernetes中运行应用，首先需要将应用打包进一个或多个容器镜像，再将那些镜像推送到镜像仓库，然后将应用的描述发布到KubernetesAPI服务器。

当API服务器处理应用的描述时，调度器调度指定组的容器到可用的工作节点上，调度是基于每组所需的计算资源，以及调度时每个节点未分配的资源。然后，那些节点上的Kubelet指示容器运行时（例如Docker）拉取所需的镜像并运行容器。仔细看图1.10以更好地理解如何在Kubernetes中部署应用程序。应用描述符列出了四个容器，并将它们分为三组。前两个pod只包含一个容器，而最后一个包含两个。这意味着两个容器都需要协作运行，不应该相互隔离。在每个pod旁边，还可以看到一个数字，表示需要并行运行的每个pod的副本数量。

在向Kubernetes提交描述符之后，它将把每个pod的指定副本数量调度到可用的工作节点上。节点上的Kubelets将告知Docker从镜像仓库中拉取容器镜像并运行容器。保持容器运行一旦应用程序运行起来，Kubernetes就会不断地确认应用程序的部署状态始终与你提供的描述相匹配。例如，如果你指出你需要运行五个web服务器实例，那么Kubernetes总是保持正好运行五个实例。如果实例之一停止了正常工作，比如当进程崩溃或停止响应时，Kubernetes将自动重启它。

同理，如果整个工作节点死亡或无法访问，Kubernetes将为在故障节点上运行的所有容器选择新节点，并在新选择的节点上运行它们。

为了让客户能够轻松地找到提供特定服务的容器，可以告诉Kubernetes哪些容器提供相同的服务，而Kubernetes将通过一个静态IP地址暴露所有容器，并将该地址暴露给集群中运行的所有应用程序。这是通过环境变量完成的，但是客户端也可以通过良好的DNS查找服务IP。kube-proxy将确保到服务的连接可跨提供服务的容器实现负载均衡。服务的IP地址保持不变，因此客户端始终可以连接到它的容器，即使它们在集群中移动。



如果在所有服务器上部署了Kubernetes，那么运维团队就不需要再部署应用程序。因为容器化的应用程序已经包含了运行所需的所有内 容，系统管理员不需要安装任何东西来部署和运行应用程序。在任何部署Kubernetes的节点上，Kubernetes可以在不需要系统管理员任何帮助的情况下立即运行应用程序。 简化应用程序部署由于Kubernetes将其所有工作节点公开为一个部署平台，因此应用程序开发人员可以自己开始部署应用程序，不需要了解组成集群的服务器。  





