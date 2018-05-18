# k8s运维指南

1.Node管理

2.NameSpace资源共享

3.资源配额管理

4.集群master高可用及监控等方面

5.k8s集群自身运维管理


## Node管理

1.Node的隔离与恢复

1.1 Node的隔离

（1）使用配置文件
```yaml
# unschudule_node.yaml
apiVersion: v1
kind: Node
metadata:
 name: k8s-node-1
 labels:
  kubernetes.io/hostname: k8s-node-1
spec:
 unschedulabels: true   #将名为k8s-node-1的节点设置为不可调度
```

执行上面的yaml文件
```sh
kubectl replace -f unschedule_node.yaml
```

（2）使用kubectl命令

node停止调度

```sh
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable": true}'
```

node恢复调度

```sh
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable": false}'
```

node停止调度

```sh
kubectl cordon k8s-node-1
```

node恢复调度

```sh
kubctl uncordon k8s-node-1
```

*Node脱离调度并不会停止执行，需要手动停止*


1.2 Node扩容

 - 新的Node节点上安装Docker、kubelet和kube-proxy
 - 配置kubelet和kube-proxy启动参数，将Master URL指定为当前k8s集群master地址
 - 启动服务

1.3 更新label

（1）pod添加label

给pod redis-master-bobr0添加标签role=backend
```sh
kubectl label pod redis-master-bobr0 role-backend
```

(2) pod删除label

给pod redis-master-bobr0 删除标签role，在命令行指定label的key，并在后面加上一个减号
```sh
kubectl label pod redis-master-bobr0 role-
```

1.4 Namespace

（1）创建命名空间

```yaml
apiVersion: v1
kind: namespace
metadata:
 name: development      #创建一个名为development的命名空间
```

查看系统中已有的命名空间

```sh
kubectl get namespaces
```

（2）定义Context(运行环境)

每个运行环境属于某个特点的命名空间

```sh
kubectl config ser-cluster kubernetes-cluster --server=https://192.168.1.128:8080
#设置一个名为kubernetes-cluster的集群，其服务地址为https://192.168.1.128:8080

kubectl config set-context ctx-dev --namespace=development --cluster=kubernetes-cluster --use=dev
#设置一个名为ctx-dev的运行环境，该运行环境属于名为development的命名空间，所属集群名为kubernetes-cluster，用户名为dev
```

（3）设置工作组在特定context环境中工作

```sh
kubectl config use-context ctx-dev #将当前运行环境设置为 ctx-dev
```

1.5 资源管理

以下从计算资源(Compute Resource)、资源配置管理(LimiteRange)、服务质量管理(Qos)、资源配额管理(ResourceQuota)四个方面进行说明

1.5.1 计算资源(Compute Resource)

（1）Pod和container的Requests和Limits参数

 - spec.container[].resource.requests.cpu
 - spec.container[].resource.limits.cpu
 - spec.container[].resource.requests.memory
 - spec.container[].resource.limits.memory