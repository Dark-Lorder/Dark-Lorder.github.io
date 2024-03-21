---
title: Kubernetes最佳实践
date: 2023-11-07 11:42:44
categories:
- Kubernetes
tags:
- Kubernetes
---

1、编写资源配置文件，使用Deployment部署mysql，并使用Service提供访问入口；使用Deployment部署Wordpress（基于apache的镜像），并使用NodePort类型的Service将其暴露到集群外部；
提示：使用mysql的service名称访问mysql服务；

```
Wordpress.Service + Mysql.Service
```

```shell
[root@k8s-master1 ~]# kubectl explain deployment   # 解释说明 deployment
KIND:     Deployment
VERSION:  apps/v1

DESCRIPTION:
     Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
   apiVersion	<string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>
     Specification of the desired behavior of the Deployment.

   status	<Object>
     Most recently observed status of the Deployment.

[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl create deployment db --image=mysql:8.0 --replicas=1 --dry-run=client -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: db
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: db
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        resources: {}
status: {}
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl create deployment db --image=mysql:8.0 --replicas=1 --dry-run=client -o yaml > mysql.yaml
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# echo "---" >> mysql.yaml 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl create service clusterip db --tcp=3306:3306 --dry-run=client -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: db
  name: db
spec:
  ports:
  - name: 3306-3306
    port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    app: db
  type: ClusterIP
status:
  loadBalancer: {}
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl create service clusterip db --tcp=3306:3306 --dry-run=client -o yaml >> mysql.yaml 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# vim mysql.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: db
  name: db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db
  template:
    metadata:
      labels:
        app: db
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        args:  # 新版本镜像有更新，需要使用下面的认证插件环境变量配置才会生效
        - --default_authentication_plugin=mysql_native_password
        - --character-set-server=utf8mb4
        - --collation-server=utf8mb4_unicode_ci
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "true"
        - name: MYSQL_USER
          value: "magedu"
        - name: MYSQL_PASSWORD
          value: "Magedu@1234"
        - name: MYSQL_DATABASE
          value: "wordpress"
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: db
  name: db
spec:
  ports:
  - name: 3306-3306
    port: 3306
    protocol: TCP
    targetPort: 3306
  selector:
    app: db
  type: ClusterIP
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl apply -f mysql.yaml 
deployment.apps/db created
service/db created
[root@k8s-master1 ~]# kubectl get pods
NAME                                        READY   STATUS    RESTARTS      AGE
db-76ff9cb5c7-hn8fn                         0/1     Running   0             9s
[root@k8s-master1 ~]# kubectl get svc
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
db                      ClusterIP   10.100.165.22    <none>        3306/TCP                     13s
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
```

```shell
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl create deployment wordpress --image=wordpress:5.8.2-apache --replicas=1 --dry-run=client -o yaml > wordpress.yaml 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# echo "---" >> wordpress.yaml 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl create service nodeport wordpress --tcp=80:80 --dry-run=client -o yaml >> wordpress.yaml 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# vim wordpress.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: wordpress
  name: wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - image: wordpress:5.8.2-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: "db.default.svc.cluster.local"
        - name: WORDPRESS_DB_USER
          value: "magedu"
        - name: WORDPRESS_DB_PASSWORD
          value: "Magedu@1234"
        - name: WORDPRESS_DB_NAME
          value: "wordpress"
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wordpress
  name: wordpress
spec:
  ports:
  - name: 80-80
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: wordpress
  type: NodePort
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl apply -f wordpress.yaml 
deployment.apps/wordpress created
service/wordpress created
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl get pods
NAME                                        READY   STATUS    RESTARTS      AGE
db-76ff9cb5c7-hn8fn                         0/1     Running   0             21m
wordpress-77c89788d4-vhx6v                  0/1     Running   0             6s
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl get svc
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
db                      ClusterIP   10.100.165.22    <none>        3306/TCP                     21m
wordpress               NodePort    10.100.144.128   <none>        80:31495/TCP                 12s
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# 
```

2、将Jenkins运行于Kubernetes集群之上，且能够在Kubernetes集群外部打开其Web GUI；

```shell
[root@k8s-master1 ~]# vim jenkins-basics.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: jenkins
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        #image: jenkins/jenkins:2.323-alpine
        image: jenkins/jenkins:2.323-jdk11
        imagePullPolicy: IfNotPresent
        env:
        - name: JAVA_OPTS
          value: -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85 -Duser.timezone=Asia/Shanghai
        ports:
        - containerPort: 8080
          name: web
          protocol: TCP
        - containerPort: 50000
          name: agent
          protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: jenkins 
  labels:
    app: jenkins
spec:
  selector:
    app: jenkins
  type: NodePort
  externalTrafficPolicy: Cluster
  ports:
  - name: http
    port: 8080
    targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins-jnlp
  namespace: jenkins 
  labels:
    app: jenkins
spec:
  selector:
    app: jenkins
  ports:
  - name: agent
    port: 50000
    targetPort: 50000
[root@k8s-master1 ~]# 
[root@k8s-master1 ~]# kubectl apply -f jenkins-basics.yaml 
namespace/jenkins created
deployment.apps/jenkins created
service/jenkins created
service/jenkins-jnlp created
[root@k8s-master1 ~]#
```