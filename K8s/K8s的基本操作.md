### Pod

- K8s中的运行容器的最小单位.
- 一般通过 `Deployment`或 `Replication Controller`自动创建

一般参考语句

```shell
#获取pod信息
kubectl get pods -o wide
#查看pod详细信息
kubectl describe pod `pod名`
```

创建一个pod的模板:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
spec:
  containers:
  - name: my-nginx
    image: nginx:1.8
```

#### Service

- `kubctl expose` 命令，会给`pod`创建一个`service`，供外部访问.
- `Service`主要有三种类型，一种叫ClusterIP,一种叫NodePort,一种叫外部的LoadBalancer
- 另外也可以使用DNS，但是需要DNS的add-on

```shell
#获取service信息
kubectl get svc
#为deployment创建一个服务。ClusterIP
kubectl expose deployment ***
#为deployment创建一个服务。NodePort
kubectl expose deployment *** --type=NodePort
```

如下是一个创建service的模板yaml:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
spec:
  ports:
  - port: 80
  	nodePort: 80
    protocol: TCP
  selector:
    app: nginx
  type: NodePort
```

### Deployment

- k8s新版本中用来替代以前的[ReplicationController](https://www.kubernetes.org.cn/replication-controller-kubernetes)来方便的管理应用。
- 声明`replicas`个数指定部署的pod数量。
- 需要注意下`apiVersion`的值。

```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
#获取deployment信息
kubectl get deployment
#查看deployment详细信息
kubectl describe deployment nginx-deployment
```

一个创建Deployment的模板yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

