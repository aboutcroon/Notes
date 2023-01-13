# 业务

## 在 controller 中的使用

下载docker

下载docker-compose



cd 到 docker-compose目录下执行命令 ./run-compose-local.sh，从而下载镜像

本地的8080端口是操作数据库的端口

3306是MySQL数据库运行的端口，此时要关闭本地的MySQL数据库，防止端口冲突



打包镜像

docker save -o orion-root-gui-2.4.6.tar.gz virtaitech/orion-root-gui:2.4.6



# 学习

## MongoDB

### docker 进入 mongoDB

首先进入 mongoDB 容器

```nginx
docker exec -it 容器id bash
# 或者 docker exec -it 容器id /bin/bash
```

然后在容器内，进入 mongoDB 数据库（命令行输入 mongo 即可）

```nginx
mongo
```



参考https://blog.csdn.net/Bule_daze/article/details/103562686



## docker compose

### links 与 depends on

links: 其他容器使用 links: mongo 可以将 mongo 容器的 ip 记录到该容器中, 再通过连接 mongo:27017 可以访问数据库。
depends on: 通过 depends_on 来标记依赖关系, 当 mongo 服务启动完成后, 才会启动 backend 服务;



参考https://blog.csdn.net/gold0523/article/details/102467102



# 报错

## 'docker build' error: "failed to solve with frontend dockerfile.v0"

> 参考：https://github.com/docker/buildx/issues/415

docker build 的时候报错，看 issue 发现把 Dockerfile 改成 dockerfile 即可

