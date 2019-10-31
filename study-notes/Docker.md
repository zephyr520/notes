## Docker常用命令 ##
1. 搜索镜像： docker search
2. 获取镜像： docker pull
3. 查看镜像： docker images
4. 删除镜像： docker rmi
5. 启动容器： docker run --name -h hostname
6. 停止容器： docker stop CONTAINER ID
7. 查看容器： docker ps
8. 进入容器： docker exec | docker attach
9. 删除容器： docker rm
10. 真正进入容器的方式： nsenter命令需要安装util-linux库
	1. docker inspect --format "{{.State.Pid}}" mynginx
	2. nsenter --target 5257 --mount --uts --ipc --net --pid /bin/bash