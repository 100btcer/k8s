# k8s部署测试总体文档

**开启本地仓库**
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

## 编译golang
**mac环境下编译linux程序**
```
 CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build main.go
```

## 制作镜像
**Dockerfile文件**
```
# 指定基础镜像
FROM centos:latest
LABEL authors="docker-test"

# 设置工作目录
WORKDIR /app/gateway

# 复制当前目录下的所有文件到工作目录
COPY . .

# 声明容器运行时需要暴露的端口
EXPOSE 8888

# 定参
ENTRYPOINT ["./main"]
```
**制作镜像**
```
docker build -t apiserver .
```
**打tag，以便关联到本地仓库**
```
docker tag apiserver:latest localhost:5000/apiserver:latest
```
**推送镜像到本地仓库**
```
docker push localhost:5000/apiserver:latest
```

## k8s通过配置文件操作
**文件**
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-golang-app
spec:
  selector:
    matchLabels:
      tier: my-golang-app
  replicas: 2
  template:
    metadata:
      labels:
        tier: my-golang-app
    spec:
      containers:
      - name: my-golang-app
        image: localhost:5000/my-golang-app
        imagePullPolicy: IfNotPresent # This should be by default so
        ports:
        - containerPort: 8888
```
**第二种只创建pod**
```
apiVersion: v1
kind: Pod
metadata:
  name: my-golang-app-pod
spec:
  containers:
  - name: your-container
    image: localhost:5000/your-image-name:tag

```
## k8s一些命令行操作
```
1、创建控制器，拉取镜像
kubectl create deployment my-golang-app --image=localhost:5000/my-golang-app

2、查看创建好的控制器
kubectl get deploy

3、查看容器所在的pod运行情况
kubectl get pod

4、暴漏端口
kubectl expose deployment my-golang-app --port=8888 --type=NodePort

5、查看服务
kubectl get service

6、删除部署者
kubectl delete deploy my-golang-app

7、删除服务
kubectl delete service my-golang-app

8、查看命名空间
kubectl get namespace
```

## treafik安装
按照这个文档：`https://blog.bling.moe/post/14/`












