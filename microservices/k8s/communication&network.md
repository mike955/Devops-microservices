## k8s网络与通信

### 1.k8s网络模型

 - 每个pod都有一个独立的IP地址，且假定所有pod都是可以直接连通，并处在一个扁平的空间中
 - pod之间访问使用的是对方pod的实际地址（docker0上分配的）
 - IP-per—Pod模型

### 2.docker网络


### 3.k8s网络实现
 
k8s网络设置主要致力于解决一下场景

 - 容器到容器之间通信
 - 抽象的pod到pod之间进行通信
 - Pod到Service之间进行通信
 - 集群外部与内部组件之间进行通信

#### 3.1 容器到容器之间通信

 同一个pod内的容器共享一个网络命名空间，共享一个IP地址，可以使用linux的IPC通信，它们之间互相访问只需要使用localhost

#### 3.2 pod之间进行通信

##### 3.2.1 同一个Node内的pod之间通信

&emsp;&emsp;pod1和pod2都是通过Veth连接在同一个docker0网桥上的，他们的IP地址IP1、IP2都是从docker0网桥上动态获取的，他们和网桥本身的IP3是同一个网段。
 
&emsp;&emsp;pod1和pod2的Linux协议栈上，默认路由都是docker0的地址，也就是说所有非本地的网络数据，都会默认发送到docker0的网桥上，由docker0网桥直接中转。

&emsp;&emsp;因此pod1和pod2关联在同一个docker0网桥上，地址段相同，所有他们之间能够通过*PodIP:containerPort*进行通信

##### 3.2.2 不同Node上的pod之间通信

&emsp;&emsp;由于不同Node上的Pod连接的不是同一个docker0网桥，IP地址段可能不同，且处在不同命名空间中，因此不能直接进行通信。

&emsp;&emsp;不同Node上的Pod之间的通信，需要满足两个条件：

 - k8s集群对pod的IP分配进行规划，不能有冲突
 - 将pod的IP和所在的Node的IP关联起来，通过这个关联让Pod可以互相访问

#### 3.3 Pod到Service之间进行通信

#### 3.4 集群外部访问service或pod

serivce是k8s集群范围内的概念，所以集群外的客户算系统无法通过Pod的IP地址或service的虚拟IP地址和虚拟端口号访问到他们，解决方法是将pod或service的端口号映射到宿主机。

##### 3.4.1 将容器端口号映射到物理机（pod级别）

（1）设置container级别的hostPort，将容器端口号映射到物理机

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: webapp
 labels:
  app: webapp
spec:
 containers:
 - name: webapp
   image: tomcat
   ports:
   - containerPort: 8080
     hostPort: 8081     #物理机IP:hostPort 可以访问pod内的容器(不同的hostPort对应不同的容器)
```

（2）设置Pod级别的hostNetwork=true，该pod中所有容器的端口号都被直接映射到物理机上，设置hostNetwork=true时需要注意一下几点：
  
 - 容器的ports部分如果不指定hostPort，则默认hostPort等于containerPort
 - 如果指定hostPort，则hostPort必须等于containerPort

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: webapp
 labels:
  app: webapp
spec:
 hostNetwork: true    #设置pod级别端口映射
 containers:
 - name: webapp
   image: tomcat
   ports:
   - containerPort: 8080    #物理机IP:containerPort，可以访问到pod内的容器
```

##### 3.4.2 将service端口号映射到物理机（service级别）

（1）设置nodePort映射到物理机，同时设置service的类型为NodePort

```yaml
apiVersion: v1
kind: Service
metadata:
 name: webapp
 labels:
  app: webapp
spec:
 type: NodePort
 ports:
 - port: 8080
   taregtPort: 8080
   nodePort: 8081     #物理机IP地址:nodePort可以访问到服务，需配置防火墙
 selector:
  app: webapp
```

（2）设置LoadBalancer映射到云服务提供商提供的LoadBalancer地址，仅使用在公有云场景


### 4.CNI网络模型和容器网络模型


### 5.k8s网络策略


### 6.开源网络组件

*将不同Node上的Docker容器之间的互相访问打通*

#### 6.1 Flannel

#### 6.2 Open vSwitch

#### 6.3 Calico