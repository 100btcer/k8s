# k8s部署测试总体文档

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
docker build -t localhost:5000/my-glang-app .
```
**开启本地仓库**
```
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```
**推送镜像到本地仓库**
```
docker push localhost:5000/my-glang-app
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











