# docker及docker-compose安装及说明
请参考: https://yeasy.gitbooks.io/docker_practice/compose

# 安装
```
cd ~/
git clone https://github.com/viky88/docker-lnmp.git
cd docker-lnmp

# 自行修改
1. nginx/conf/conf.d 下面的配置文件，全部删除改为自己的
2. 修改 docker-compose.yum 中mysql密码，如果不需要安装mysql，将mysql部分注释掉
   MYSQL_ROOT_PASSWORD: 123456

```

# 说明
php的build会比较慢(添加扩展)，可以直接拉取编译好的版本
```
php:
    build: ./php
```
改为：
```
php:
    image: wpengine/php:7.2
```
其他php版本可以自己编译好后，加入到 https://hub.docker.com/ 中

如果只想要nginx+php，只需要将mysql部分注释掉即可   
同理，如果只想要mysql，只需要将nginx+php部分注释掉即可

# 如何设置开机启动
```
# 根据系统先设置docker服务为开机启动
# 编辑开机启动文件
# 注意这里不用 sudo，本身是使用 root 运行的
$ sudo vim /etc/rc.local
/usr/local/bin/docker-compose -f /root/docker-lnmp/docker-compose.yml up -d
# 重启测试
$ sudo reboot
```

# alias 设置
```
# bash 修改 ~/.bash_profile  
# zsh 修改 ~/.zshrc

export DOCKERCOMPOS_EHOME=/Users/viky/project/docker/github/viky88/docker-lnmp
alias godocker="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose.yml up -d"
alias godocker_mongo3="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose_mongo3.yml up -d"
alias godocker_mongo4.0="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose_mongo4.0.yml up -d"
alias godocker_mysql5.7="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose_mysql5.7.yml up -d"
alias godocker_nginx="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose_nginx.yml up -d"
alias godocker_php7.2="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose_php7.2.yml up -d"
alias godocker_redis4.0="/usr/local/bin/docker-compose -f $DOCKERCOMPOS_EHOME/docker-compose_redis4.0.yml up -d"

```

# docker常用命令
```
# 查看当前启动的容器
sudo docker ps

# 后台启动所有服务
sudo docker-compose up -d  

# 启动部分服务在后边加服务名，不加表示启动所有，-d 表示在后台运行
sudo docker-compose up [NAME] -d

# 停止和启动类似
sudo docker-compose stop [NAME]

# 停止所有正在运行的容器
docker stop $(docker ps -q)

# 停止所有容器
sudo docker-compose kill

# 强制重新build镜像
sudo docker-compose up -d --build 

# 进入到某个镜像中
sudo docker exec -it [NAME] bash

# 删除已经停止的容器
sudo docker-compose rm  [NAME]  

# 删除所有未运行的容器
sudo docker rm $(docker ps -a -q)

# 删除所有镜像，-f 可以强制删除
sudo docker rmi $(docker images -q)

# 删除none等无用的镜像
sudo docker image prune -f

# 查看容易的详细信息
docker inspect [NAME]/[CONTAINER ID]

# 宿主机器重启docker内部复制内容
docker cp /path/file [NAME/CONTAINER ID]:/path/file
```