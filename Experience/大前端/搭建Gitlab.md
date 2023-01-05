在 Gitlab 官网文档中，选择 install

然后选择 Docker 的方式 https://docs.gitlab.com/omnibus/docker/



如下：

### Install GitLab using Docker Engine

You can fine tune these directories to meet your requirements. Once you’ve set up the `GITLAB_HOME` variable, you can run the image:

```
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  gitlab/gitlab-ee:latest
```

Docker 是写时复制，最基础的功能就是数据隔离，变更管理，日志记录

- --detach 其实就是 -D 命令，相当于在后台去运行

- --hostname 就是指定了一个域名，使用云服务器的域名指向 ip 地址

- --publish 443:443 --publish 80:80 --publish 22:22 这里的 443 端口就是走 HTTPS 的协议，80 就是默认服务访问的端口，22 就是克隆仓库要走的默认端口，我们可以更改这些数据来改变端口
- --name gitlab 指定一个镜像的名称
- --restart always 当我们 docker 服务重启的时候，gitlab 也进行自动的重启
-  --volume $GITLAB_HOME/config:/etc/gitlab \
    --volume $GITLAB_HOME/logs:/var/log/gitlab \
    --volume $GITLAB_HOME/data:/var/opt/gitlab \ 将 gitlab 中的数据映射到宿主机上来



我们还需要 HTTPS 服务，邮件服务，定制化的端口



### Install GitLab using Docker Compose

https://docs.gitlab.com/omnibus/docker/#install-gitlab-using-docker-compose