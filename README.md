
# docker常用命令
```
$ docker-compose up -d         # start containers in background
$ docker-compose kill          # stop containers
$ docker-compose up -d --build # force rebuild of Dockerfiles
$ docker-compose rm            # remove stopped containers
$ docker ps                    # see list of running containers
$ docker exec -ti [NAME] bash
$ docker stop $(docker ps -a -q) # stop containers
```