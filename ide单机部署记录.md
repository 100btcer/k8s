version: '3'

services:
  traefik:
    image: traefik:v2.5
    container_name: traefik
    restart: always
    command: --providers.docker --api.insecure=true --entrypoints.web.address=:80 --entrypoints.web.http.middlewares=auth-server@docker
#    --log.level=DEBUG
    ports:
      - "8085:80"
      - "8080:8080"
    depends_on:
      - auth-server
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    logging:
      driver: "json-file"
      options:
        max-size: "2m"
        max-file: "10"

  auth-server:
    image: localhost:5000/auth-server:latest
    container_name: auth-server
    restart: always
    environment:
      - GIN_MODE=debug

      - DOCKER_API_VERSION=1.40

      # server conf
#      - AUTH_SERVER_AUTHADDRESS=http://192.168.177.100/fw/oauth/check_token/cmbaas
      - AUTH_SERVER_DOCKERNETWORK=deploy_default # 本服务所属的 docker network
      - AUTH_SERVER_SERVERID=1 # server id，每套服务的id必须不同
      - AUTH_SERVER_SERVERADDRESS=http://127.0.0.1:8085 # server address，traefik 地址
      - AUTH_SERVER_CHECKIDESTATETIMES=10 # 启动用户ide容器后，阻塞探测ide服务状态次数
      - AUTH_SERVER_MAXCONTAINERNUM=50 # 最大容器数量（启动以及清理容器限制条件）
      - AUTH_SERVER_MINMEM=3096 # 最小剩余内存（启动以及清理容器限制条件，单位M）
      - AUTH_SERVER_CLEARCONTAINERTIME=120 # 清理经过此配置时间仍未访问的容器，单位：分钟
      - AUTH_SERVER_ROOTVOLUME=/data # 数据挂载路径
      - AUTH_SERVER_TOKENKEY=t

 #     - AUTH_SERVER_BAASADDRESS=http://192.168.177.100/v1
#      - AUTH_SERVER_APISERVERADDRESS=https://eoside.peersafe.cn
#     - AUTH_SERVER_LOGINURL=http://192.168.177.100/login
#      - AUTH_SERVER_NFSIP=192.168.177.182

      # db conf
      - AUTH_DB_TYPE=mysql
      - AUTH_DB_ADDRESS=mysql-server
      - AUTH_DB_PORT=3306
      - AUTH_DB_USER=root
      - AUTH_DB_PASSWORD=root
      - AUTH_DB_NAME=ide

    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /etc/localtime:/etc/localtime:ro
      - /data/user_config:/opt/gopath/src/github.com/peersafe/ide_authserver/goapi/user_config
      - /data/user_data:/opt/gopath/src/github.com/peersafe/ide_authserver/goapi/user_data
    command: ./goapi
    depends_on:
      - apiserver
    labels:
      - "traefik.http.middlewares.auth-server.forwardauth.address=http://auth-server:4181"
      - "traefik.http.services.auth-server.loadbalancer.server.port=4181"
    logging:
      driver: "json-file"
      options:
        max-size: "2m"
        max-file: "10"

  apiserver:
    container_name: apiserver
    image: localhost:5000/ide_apiserver:latest
    restart: always
    command: ./apiserver
    environment:
      - API_APISERVER_LISTENPORT=8888
      - API_DBCONFIG_TYPE=mysql
      - API_DBCONFIG_ADDRESS=mysql-server
      - API_DBCONFIG_PORT=3306
      - API_DBCONFIG_USER=root
      - API_DBCONFIG_PASSWORD=root
      - API_DBCONFIG_NAME=ide
#      - API_APISERVER_CODESERVERADDRESS=/?folder=/home/workspace/&file=eos/talk/talk.cpp
#      - API_APISERVER_CMBASSADDRESS=http://192.168.177.100
    working_dir: /opt/apiserver
    volumes:
      - /data/user_data:/home/user_data
    depends_on:
      - mysql-server
    labels:
      - "traefik.http.routers.apiserver.priority=3"
      - "traefik.http.routers.apiserver.rule=Path(`/edit`) || Path(`/new`) || PathPrefix(`/api/`)"
      - "traefik.http.services.apiserver.loadbalancer.server.port=8888"
    logging:
      driver: "json-file"
      options:
        max-size: "2m"
        max-file: "10"

  mysql-server:
    container_name: mysql-server
    image: "mysql:5.7"
    restart: always
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: "root"
      MYSQL_ROOT_HOST: '%'
    volumes:
      - /etc/localtime:/etc/localtime
      - ./init/mysql.sql:/docker-entrypoint-initdb.d/mysql.sql
    ulimits:
      nproc: 65535
      nofile:
        soft: 65535
        hard: 65535
    logging:
      driver: "json-file"
      options:
        max-size: "2m"
        max-file: "10"
