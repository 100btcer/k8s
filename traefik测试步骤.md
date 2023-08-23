# 涉及到的服务
- api服务：my-golang-app
- 中间件服务：testmidda(http返回200)，testmiddc(http返回500)
**以上服务都制作了镜像，并推送到本地仓库**

# 涉及到的文件
- traefik.yaml:配置
- docker-compose.yaml:配置

# 文件内容
traefik.yaml
```
api:
  dashboard: true
  insecure: true
log:
  filePath: "/config/log/hello.json"
  format: json
  level: debug
entryPoints:
  web:
    address: :80
providers:
  file:
    directory: /config
    watch: true

http:
  middlewares:
    testmiddc:
      forwardauth:
        address: http://testmiddc:8086
  routers:
    my-golang-app-router:
      entryPoints:
      - web
      middlewares:
      - testmiddc
      priority: 3
      rule: "Host(`my-golang`)"
      service: my-golang-app
  services:
    my-golang-app:
      loadBalancer:
        servers:
          - url: http://my-golang-app:8888
```
docker-compose.yaml
```
version: '3'
services:
  traefik:
    image: traefik:v2.4
    command:
      - "--configfile=/config/traefik.yaml"
    ports:
      - "8888:8888"
      - "8883:8883"
       # The HTTP port
      - "80:80"
      # The Web UI (enabled by --api.insecure=true)
      - "8080:8080"
    volumes:
      - ./traefik.yaml:/config/traefik.yaml
    networks:
      - traefik_network
  whoami:
    # 展示ip地址的容器
    image: traefik/whoami
    networks:
      - traefik_network

  my-golang-app:
    image: localhost:5000/my-golang-app
    networks:
      - traefik_network

  testmiddc:
    image: localhost:5000/testmiddc
    networks:
      - traefik_network

networks:
  traefik_network:
    external: true
```
# 测试过程
- 将上述两个文件保存到同一个目录
- 创建网络`docker network create traefik_network`
- 执行命令`docker-compose up`启动容器，在后台运行的话，执行`docker-compose up -d`
- 打开`traefik`控制台`http://127.0.0.1:8080/dashboard`，可以看到服务都已经存在了
- 测试请求`api`接口`curl -H Host:my-golang http://127.0.0.1/api/special/healthz`，正常会返回`{"code":"0000-0000","message":"请求成功","data":""}`
- 
