docker-compose.yaml文件：
```
version: '3'

services:
  reverse-proxy:
    # The official v2 Traefik docker image
    image: traefik:v2.4
    # 启用dashboard并使用docker作为provider
    command: --api.insecure=true --providers.docker
    ports:
      # The HTTP port
      - "80:80"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock
  # ...
  whoami:
    # 展示ip地址的容器
    image: traefik/whoami
    labels:
      - "traefik.http.routers.whoami.rule=Host(`whoami.docker.localhost`)"

  my-golang-app:
    image: localhost:5000/my-golang-app
    labels:
      - "traefik.http.routers.my-golang-app.rule=Host(`golang.docker.localhost`)"
```
启动服务：
```
docker-compose up -d reverse-proxy
```

```
docker-compose up -d whoami
```

```
docker-compose up -d my-golang-app
```
打开`localhost:8080`可以看到`whoami`、`my-golang-app`服务都已显示在`HTTP Routers`里


请求`my-golang-app`：
```
curl -H Host:golang.docker.localhost http://127.0.0.1/api/special/healthz
```
将获取到接口返回的数据:
```
{"code":"0000-0000","message":"请求成功","data":""}
```
扩容测试负载均衡:
```
docker-compose up -d --scale whoami=2
```


