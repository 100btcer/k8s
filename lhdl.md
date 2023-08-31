# ide_k8s_original
- 创建镜像，项目目录中，执行：`docker build -t ide_k8s_original:v1.0 .`
- 打包镜像：`docker save -o ide_k8s_original_v1.0.tar ide_k8s_original:v1.0`

# auth-server
- 打包镜像，项目上一层目录中，执行：`docker build -t authserver:v1.1 .`
- 打包镜像：`docker save -o authserver_v1.1.tar authserver:v1.1`

# apiserver
- 打包镜像，项目上一层目录中，执行：`docker build -t apiserver:v1.1 -f build/Dockerfile .`
- 打包镜像：`docker save -o apiserver_v1.1.tar apiserver:v1.1`

# 测试环境启动ide_k8s_original
```
docker run -it \
-v /data/traefik/ide/config:/data/traefik/ide/config \
-v /opt/kubernetes/ssl:/opt/kubernetes/ssl \
-p 8089:8089 \
ide_k8s_original:v1.6 \
/bin/bash
```


