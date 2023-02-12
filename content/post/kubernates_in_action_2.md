---
title: "《Kubernetes in Action》读书笔记第二章：开始使用Kubernetes和Docker"
date: 2023-02-12T18:08:21+08:00
draft: false
tags:
- kubernates
- docker
categories:
- kubernates
toc: true
---


## 1.创建、运行及共享容器镜像

首先，需要在Linux主机上安装Docker。如果使用的不是Linux操作系统，就需要启动Linux虚拟机（VM）并在虚拟机中运行Docker。如果使用的是Mac或Windows系统，Docker将会自己启动一个虚拟机并在虚拟机中运行Docker守护进程。Docker客户端可执行文件可以在宿主操作系统中使用，并可以与虚拟机中的守护进程通信。

```bash

PS C:\Users\Administrator> docker run busybox echo "hello docker"

Unable to find image 'busybox:latest' locally

latest: Pulling from library/busybox

205dae5015e7: Pull complete

Digest: sha256:7b3ccabffc97de872a30dfd234fd972a66d247c8cfc69b0550f276481852627c

Status: Downloaded newer image for busybox:latest

hello docker

```



![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676099756264-a576fd32-6d83-4a69-a01b-80aa65c8a989.png#averageHue=%23f5f5f5&clientId=u712e0692-7cfe-4&from=paste&height=366&id=ubb546374&name=image.png&originHeight=458&originWidth=876&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=115223&status=done&style=none&taskId=ue84e9efe-5bb7-4b9c-8d26-944f9fe3342&title=&width=700.8)

###  1.1创建一个简单的 Node.js 应用

nodejs服务器：

```javascript
const http = require('http');
const os = require('os');

console.log("myapp server starting...");

var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};

var www = http.createServer(handler);
www.listen(8080);

```

```bash
FROM node:7
ADD app.js ./app.js
ENTRYPOINT ["node", "app.js"]l

```

```bash
E:\code\docker\node>docker build -t myapp -f ./DockerfIle .
[+] Building 171.0s (7/7) FINISHED
...
```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676102045040-5d3c41ed-4e13-4011-85e6-3e2a62ef9f56.png#averageHue=%23f0f0f0&clientId=u712e0692-7cfe-4&from=paste&height=304&id=u0ff0ab5e&name=image.png&originHeight=380&originWidth=852&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=121701&status=done&style=none&taskId=u873035dd-b8c5-4322-9a61-af229ff5d84&title=&width=681.6)

运行容器：

```bash

docker run --name myapp -p 8080:8080 -d myapp

```

这条命令告知Docker基于myapp镜像创建一个叫myapp的新容器。这个容器与命令行分离（-d标志），这意味着在后台运行。本机上的8080端口会被映射到容器内的8080端口（-p8080:8080选项），所以可以通过http://localhost:8080访问这个应用。



查看真正运行的容器：

```bash

E:\code\docker>docker ps

CONTAINER ID   IMAGE                  COMMAND                  CREATED         STATUS          PORTS

  NAMES

87a051786b6c   myapp                  "node app.js"            3 minutes ago   Up 3 minutes    0.0.0.0:8080->8080/tcp     myapp

1715f91e7475   kindest/node:v1.24.0   "/usr/local/bin/entr…"   5 months ago    Up 19 minutes

  dev-worker

df4edbc9246a   kindest/node:v1.24.0   "/usr/local/bin/entr…"   5 months ago    Up 19 minutes

  dev-worker2

8a57dda6aa9b   kindest/node:v1.24.0   "/usr/local/bin/entr…"   5 months ago    Up 19 minutes   127.0.0.1:2949->6443/tcp   dev-control-plane

```



访问容器内服务：

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676102115907-6835bed8-5573-41ab-be0b-d666fd5ccad1.png#averageHue=%23fbfaf8&clientId=u712e0692-7cfe-4&from=paste&height=119&id=u404b6a3d&name=image.png&originHeight=149&originWidth=653&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=15066&status=done&style=none&taskId=u7896f0ac-aaf9-42d4-b5ee-744b1bee81d&title=&width=522.4)



dockerps只会展示容器的大部分基础信息。可以使用dockerinspect查看更多的信息：

```bash

docker inspect myapp

```



容器拥有完整的文件系统：

```bash

E:\code\docker>docker exec -it myapp bash

root@87a051786b6c:/# ls

app.js  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```

这会在已有的myapp容器内部运行bash。bash进程会和主容器进程拥有相同的命名空间。这样可以从内部探索容器，查看Node.js和应用是如何在容器里运行的。-it选项是下面两个选项的简写：-i，确保标准输入流保持开放。需要在shell中输入命令。-t，分配一个伪终端（TTY）。如果希望像平常一样使用shell，需要同时使用这两个选项（如果缺少第一个选项就无法输入任何命令。如果缺少第二个选项，那么命令提示符不会显示，并且一些命令会提示TERM变量没有设置）。



![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676102388244-84a7efba-b9c1-4751-b4fb-b34978697e25.png#averageHue=%23f8f6f4&clientId=u712e0692-7cfe-4&from=paste&height=328&id=uc9d064fa&name=image.png&originHeight=410&originWidth=945&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=174359&status=done&style=none&taskId=u8c266752-04ed-4dde-8204-4fa7e24648e&title=&width=756)



### 1.2 构建镜像

打包成镜像：

```bash

PS C:\Users\Administrator> docker tag myapp wallace2504/nodejs-app

PS C:\Users\Administrator> docker images

REPOSITORY                                                    TAG                  IMAGE ID       CREATED          SIZE

myapp                                                         latest               d8dc085fb4f4   14 minutes ago   660MB

wallace2504/nodejs-app                                        latest               d8dc085fb4f4   14 minutes ago   660MB

```

上传到镜像仓库：

```bash

PS C:\Users\Administrator> docker push wallace2504/nodejs-app

Using default tag: latest

The push refers to repository [docker.io/wallace2504/nodejs-app]

e979efccacfd: Pushed

ab90d83fa34a: Mounted from library/node

8ee318e54723: Mounted from library/node

e6695624484e: Mounted from library/node

da59b99bbd3b: Mounted from library/node

5616a6292c16: Mounted from library/node

f3ed6cb59ab0: Mounted from library/node

654f45ecb7e3: Mounted from library/node

2c40c66f7667: Mounted from library/node

latest: digest: sha256:182ac2e573c9535e101ffd3f59a7c9660ef6881ce5a21a77750198e74646ec18 size: 2213

```



镜像仓库：[https://hub.docker.com/u/wallace2504](https://hub.docker.com/u/wallace2504)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676102727945-95e1859c-b8ba-443b-bb85-63bf3f7001e0.png#averageHue=%23e3cba4&clientId=u712e0692-7cfe-4&from=paste&height=520&id=u407f68fb&name=image.png&originHeight=650&originWidth=1407&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=57547&status=done&style=none&taskId=ub49c82c0-180e-431a-b103-a9fd53a4dc5&title=&width=1125.6)

通过镜像仓库创建容器：

```bash

PS C:\Users\Administrator> docker run --name myapp2 -p 8081:8080 -d wallace2504/nodejs-app

7696bc323ab71d7b11f926dfa2e9f6d99b670234d3d79ab2dc06d04cb0d41267

```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676191927268-e79701a9-febe-4055-aaa9-a408bc0e80d8.png#averageHue=%23f2f1f1&clientId=u750ad04f-360b-4&from=paste&height=491&id=ub83aa72a&name=image.png&originHeight=614&originWidth=887&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=164327&status=done&style=none&taskId=ufe930bbc-3843-42a8-bc03-79fd833cc54&title=&width=709.6)





##  2.配置Kubernetes集群  

使用Minikube是运行Kubernetes集群最简单、最快捷的途径。 Minikube是一个构建单节点集群的工具，对于测试Kubernetes和本地开 发应用都非常有用。  



### 2.1 使用minikube

使用Minikue启动一个Kubernetes集群:

```bash

PS C:\Users\Administrator> minikube start

😄  Microsoft Windows 11 Pro 10.0.22000 Build 22000 上的 minikube v1.26.1

🆕  Kubernetes 1.24.3 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.24.3

🎉  minikube 1.29.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.29.0

💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'



✨  根据现有的配置文件使用 docker 驱动程序

👍  Starting control plane node minikube in cluster minikube

🚜  Pulling base image ...

🔄  Restarting existing docker container for "minikube" ...

🐳  正在 Docker 20.10.17 中准备 Kubernetes v1.23.8…

🔎  Verifying Kubernetes components...

    ▪ Using image registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner:v5

🌟  Enabled addons: storage-provisioner, default-storageclass

🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default

PS C:\Users\Administrator> kubectl

>> ^C

PS C:\Users\Administrator> minikube start

😄  Microsoft Windows 11 Pro 10.0.22000 Build 22000 上的 minikube v1.26.1

🆕  Kubernetes 1.24.3 is now available. If you would like to upgrade, specify: --kubernetes-version=v1.24.3

```



使用kubectl列出集群节点，要查看关于对象的更详细的信息，可以使用 kubectl describe 命令， 它显示了更多信息：

```bash

PS C:\Users\Administrator> kubectl get nodes

NAME       STATUS   ROLES                  AGE    VERSION

minikube   Ready    control-plane,master   177d   v1.23.8





PS C:\Users\Administrator> kubectl describe node minikube

Name:               minikube

Roles:              control-plane,master

Labels:             beta.kubernetes.io/arch=amd64

                    beta.kubernetes.io/os=linux

                    kubernetes.io/arch=amd64

                    kubernetes.io/hostname=minikube

                    kubernetes.io/os=linux

                    minikube.k8s.io/commit=62e108c3dfdec8029a890ad6d8ef96b6461426dc

                    minikube.k8s.io/name=minikube

                    minikube.k8s.io/primary=true

                    minikube.k8s.io/updated_at=2022_08_17T21_53_36_0700

                    minikube.k8s.io/version=v1.26.1

                    node-role.kubernetes.io/control-plane=

                    node-role.kubernetes.io/master=

                    node.kubernetes.io/exclude-from-external-load-balancers=

Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock

                    node.alpha.kubernetes.io/ttl: 0

                    volumes.kubernetes.io/controller-managed-attach-detach: true

CreationTimestamp:  Wed, 17 Aug 2022 21:53:33 +0800

Taints:             <none>

Unschedulable:      false

...

```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676104179065-91f29911-700c-4886-8a77-864b33e1285e.png#averageHue=%23f3f3f2&clientId=u7e91d429-52af-4&from=paste&height=535&id=uc641d67c&name=image.png&originHeight=669&originWidth=897&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=201056&status=done&style=none&taskId=ua4c651f0-e692-47fd-9d30-05d3e2a0127&title=&width=717.6)



### 2.2  在Kubernetes上运行第一个应用  

  部署应用程序最简单的方式是使用 kubectl run 命令，该命令可以创 建所有必要的组件而无需JSON或YAML文件：

```bash

PS C:\Users\Administrator> kubectl run myapp --image=wallace2504/nodejs-app --port=8080

pod/myapp created

```

_#注意：kubectlrun在旧版本中默认创建的是deployment，但在新的版本中创建的是pod则其创建过程不涉及deployment_

_建议使用_

```bash

 kubectl create deployment myapp --image=wallace2504/nodejs-app

```



查看运行的应用：

```bash

PS C:\Users\Administrator> kubectl get pods

NAME    READY   STATUS    RESTARTS   AGE

myapp   1/1     Running   0          105s

```



 为了更好地理解容器、pod和节点之间的关系，请查看图 2.5。如你 所见，每个pod都有自己的IP，并包含一个或多个容器，每个容器都运 行一个应用进程。pod分布在不同的工作节点上。  

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676192327702-75ca6ca6-a69c-47d3-b41b-cf056146c2a1.png#averageHue=%23e1e1e1&clientId=u6ef3f8a1-46f3-4&from=paste&height=300&id=u7d6976f6&name=image.png&originHeight=375&originWidth=847&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=131190&status=done&style=none&taskId=uab20453e-926c-4853-9bd4-98d5a841e0d&title=&width=677.6)

查看pod详细信息：

```bash

PS C:\Users\Administrator> kubectl describe pod myapp

Name:         myapp

Namespace:    default

Priority:     0

Node:         minikube/192.168.49.2

Start Time:   Sat, 11 Feb 2023 16:34:05 +0800

Labels:       run=myapp

Annotations:  <none>

Status:       Running

IP:           172.17.0.3

IPs:

  IP:  172.17.0.3

Containers:

  myapp:

    Container ID:   docker://accf5662760d75d7c7fe95c545c6ad14fac365458862558ad134fe544f3d63dd

    Image:          wallace2504/nodejs-app

    Image ID:       docker-pullable://wallace2504/nodejs-app@sha256:182ac2e573c9535e101ffd3f59a7c9660ef6881ce5a21a77750198e74646ec18

    Port:           8080/TCP

    Host Port:      0/TCP

    State:          Running

      Started:      Sat, 11 Feb 2023 16:34:29 +0800

    Ready:          True

    Restart Count:  0

    Environment:    <none>

    Mounts:

      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2tdqq (ro)

...

```



![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676104765132-9a245207-58e5-42a3-9180-62b31dcc2021.png#averageHue=%23f1f1f1&clientId=u7e91d429-52af-4&from=paste&height=514&id=VOM9F&name=image.png&originHeight=642&originWidth=859&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=206260&status=done&style=none&taskId=u138a0acf-12f2-4782-bc5d-58c039be4e9&title=&width=687.2)



它显示了在Kubernetes中运行容器镜像所必需的两个步骤。首先，构建镜像并将其推送到DockerHub。这是必要的，因为在本地机器上构建的镜像只能在本地机器上可用，但是需要使它可以访问运行在工作节点上的Docker守护进程。

当运行kubectl命令时，它通过向KubernetesAPI服务器发送一个RESTHTTP请求，在集群中创建一个新的Deployment对象。然后，Deployment创建了一个新的pod，调度器将其调度到一个工作节点上。Kubelet看到pod被调度到节点上，就告知Docker从镜像中心中拉取指定的镜像，因为本地没有该镜像。下载镜像后，Docker创建并运行容器。



如何访问正在运行的pod？我们提到过每个pod都有自己的IP地址，但是这个地址是集群内部的，不能从集群外部访问。要让pod能够从外部访问，需要通过服务对象公开它，要创建一个特殊的LoadBalancer类型的服务。因为如果你创建一个常规服务（一个ClusterIP服务），比如pod，它也只能从集群内部访问。通过创建LoadBalancer类型的服务，将创建一个外部的负载均衡，可以通过负载均衡的公共IP访问pod。

_#注意Minikube不支持LoadBalancer类型的服务，因此服务不会有外部IP。_



老版本创建service流程：![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676104919922-a239d8bc-33bf-4925-9b7c-f6e2b707c0ae.png#averageHue=%23f9f8f8&clientId=u7e91d429-52af-4&from=paste&height=228&id=hnMaQ&name=image.png&originHeight=285&originWidth=901&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=69820&status=done&style=none&taskId=u071fae45-9610-4606-a2d4-cfed02d931a&title=&width=720.8)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676104949488-8c3e5a97-9933-4677-9fc4-5a8896bda480.png#averageHue=%23f9f8f7&clientId=u7e91d429-52af-4&from=paste&height=471&id=cLxfM&name=image.png&originHeight=589&originWidth=913&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=179209&status=done&style=none&taskId=uc44662e7-6d0c-4ac4-a860-05c2666542c&title=&width=730.4)



在minikube中创建sercvice：

```bash

PS C:\Users\Administrator> kubectl expose pods myapp --type=NodePort --port=8080 --name myapp2

service/myapp2 exposed

```

```bash

PS C:\Users\Administrator> minikube service myapp2 --url

http://127.0.0.1:6249

❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.

```



或者直接暴露对外端口（不创建service）：

```bash

PS C:\Users\Administrator>  kubectl port-forward --address 0.0.0.0 myapp  8082:8080

Forwarding from 0.0.0.0:8082 -> 8080

```



获取service:

```bash

PS C:\Users\Administrator> kubectl get svc

NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE

kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          178d

myapp2       NodePort    10.110.238.163   <none>        8080:32627/TCP   14m

```



为什么需要服务系统的第三个组件是http服务。要理解为什么需要服务，需要学习有关pod的关键细节。

pod的存在是短暂的，一个pod可能会在任何时候消失，或许因为它所在节点发生故障，或许因为有人删除了pod，或者因为pod被从一个健康的节点剔除了。当其中任何一种情况发生时，如前所述，消失的pod将被deployment替换为新的pod。新的pod与替换它的pod具有不同的IP地址。这就是需要服务的地方——解决不断变化的podIP地址的问题，以及在一个固定的IP和端口对上对外暴露多个pod。

当一个服务被创建时，它会得到一个静态的IP，在服务的生命周期中这个IP不会发生改变。客户端应该通过固定IP地址连接到服务，而不是直接连接pod。服务会确保其中一个pod接收连接，而不关心pod当前运行在哪里（以及它的IP地址是什么）。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676192778729-51529eb5-5a6c-4e8f-9946-83e9b43f4281.png#averageHue=%23ebebeb&clientId=u6ef3f8a1-46f3-4&from=paste&height=358&id=u9ed969e7&name=image.png&originHeight=447&originWidth=830&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=153610&status=done&style=none&taskId=uc7480802-40fd-4e75-957e-6ef7328e21e&title=&width=664)

###  2.3 水平伸缩应用  

使用scale命令：

```bash

kubectl scale deployment myapp4 --replicas=3

```

_#使用run命令创建的pod在新版本无法使用scale操作_



### 2.4  Kubernetes dashboard  

```bash

PS C:\Users\Administrator> minikube dashboard

🔌  正在开启 dashboard ...

    ▪ Using image registry.cn-hangzhou.aliyuncs.com/google_containers/dashboard:v2.6.0

    ▪ Using image registry.cn-hangzhou.aliyuncs.com/google_containers/metrics-scraper:v1.0.8

🤔  正在验证 dashboard 运行情况 ...

🚀  Launching proxy ...

🤔  正在验证 proxy 运行状况 ...

PS C:\Users\Administrator> ^C

PS C:\Users\Administrator> minikube dashboard

🤔  正在验证 dashboard 运行情况 ...

🚀  Launching proxy ...

🤔  正在验证 proxy 运行状况 ...

🎉  Opening http://127.0.0.1:8033/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...

```

![image.png](https://cdn.nlark.com/yuque/0/2023/png/29308157/1676192808623-61bc1198-c793-4a5b-ba9b-9b7b513e2a74.png#averageHue=%23c1d6cd&clientId=u6ef3f8a1-46f3-4&from=paste&height=828&id=ubf870df8&name=image.png&originHeight=1035&originWidth=2544&originalType=binary&ratio=1.25&rotation=0&showTitle=false&size=148184&status=done&style=none&taskId=uebb7ac6a-5f84-4c81-ba5f-73a83cefa87&title=&width=2035.2)

