# k8s基本概念和术语

## 1.master

master指的是集群控制节点，每个k8s集群需要有一个master节点
 - 作用：
  - 负责集群的管理和控制
 - 节点运行以下进程：
  - kubernetes API Server (kube-apiserver)
  - kubernetes Controller Manager (kube-controller-manager)
  - kubernetes Scheduler (kube-scheduler)  
  - etcd

## 2.Node

Node指的是k8s集群中除master外的所有节点，为pod的运行节点

 - 作用：运行Pod
 - 节点允许以下进程
    - kubelet
    - kube-proxy
    - docker-engine

## 3.Pod

Pod是k8s管理的最小单位

 - 包含的容器
    - Pause：pod内所有容器共享pause容器ip
    - 其它应用容器  
 - 种类：
    - 普通Pod
    - 静态Pod：不存储在etcd中，存放在具体的Node上的一个文件中 
 - Endpoint:
    - Endpoint=PodIP(pauseIP):ContainerIP    
 - 创建脚本
 ```yaml
 apiVersion: v1
 kind: Pod
 metadata:
  name: myweb   #pod的名字
  labels:       #pod标签
   name: myweb  
 spec:          #pod包含的容器组，可以有多个
  containers:   #可以在里面对容易所能使用的资源(cpu等)进行配置
   - name: myweb    #容器名称
     images: kubeguide/tomcat-app:v1    #容器镜像
     ports: 
      - containerPort: 8080     ##容器端口号
     env:       #容器环境变量
     - name: MYSQL_SERVICE_HOST
       value: 'mysql'
     - name: MYSQL_SERVICE_PORT
       value: '3306'
 ```

 4.Label

 label是一个key=value的键值对，可以用于多种场合

  - 版本标签
    - release:stable
  - 环境标签
    - environment: dev
  - 架构标签
    - environment: frontend
  - 分区标签
    - partition: cunsomerA
  - 质量管控标签  
    - track: daily
  - 使用场景
    - kube-controller通过RC上定义的label selector来筛选pod
    - kube-proxy通过service的label selector来筛选pod
    - 在pod文件中使用NodeSelector来定向调度

5.Replication Controller

 pod副本管理器，即让pod的数量符合定义时的预期值

  - 支持基于集合的Label Selector
  - RC定义包含如下几部分
    - pod期待的副本数(replicas)
    - 用于筛选目标的pod的Label Selector
    - 当pod的副本数量小于预期值时，创建新的pod
  - RC模版
  ```yaml
apiVersion: v1
kind: ReplicationController #已更名为ReplicaSet
metadata:
 name: frontend
spec:
 replicas: 1
 selector:
  tier: frontend
 template:
  metadata:
   labels:
    app: app-demo
    tier: frontend
   spec:
    containers:
     - name: tomcat-demo
       image: tomcat
       imagePullPolicy: IfNotPresent
       env:
        - name: GET_HOSTS_FROM
          value: dns
       ports:
        - containerPort: 80
  ```

5.Deployment

用来解决Pod的编排问题，是Replica Set的升级

- 使用场景
    - 创建一个Deployment对象来创建pod创建
    - 检查Deployment状态来查看部署状态
    - 更新Deploment以创建新的pod
    - 当前Deploment不稳定，可以回滚到上一个版本
    - 暂停Deploment修改PodTemplatesSpec配置，恢复Deploment，再发布
    - 扩展Deploment以应对高负载
    - 清理不需要的旧版本ReplicaSet
- 支持基于集合的Label Selector
- 创建脚本
```yaml
apiVersion: extension/v1beta1
kind: Deployment
metadata:
 name: frontend
spec:
 replicas: 1
 selector:
  matchLables:
   tier: frontend
   matchExpressions:
    - {key: tier, operator: In, values: [frontend]}
  template:
   metadata:
    labels:
     app: app-demo
     tier: frontend
   spec:
    containers:
     - name: tomcat-demo
       images: tomcat
       imagesPullPolicy: IfNotPresent   #镜像拉取策略
       ports:
       - containerPort: 8080
```

6.Horizontal Pod Autoscaler

pod只能扩容，通过追踪分析RC控制的所有目标Pod的负载变化情况，来针对性的调整目标pod的副本数，这是HPA的实现原理

 - pod负载指标
    - CPUUtilizationPercentage
    - 应用程序自定义的度量指标，如(QPS和TPS)
 - 模版
 ```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
 name: php-apache
 namespace: default
spec:
 maxReplicas: 10
 minReplicas: 1
 scaleTargetRef:
  king: Deployment
  name: php-apache
 targetCPUUtilizationPercentage: 90  #cpu使用率超过90%自动扩容
 ```