---
title: docker学习笔记
date: 2018-04-27 11:23:11
tags: docler
categories: 技术

---
docker image ls  //列出本地镜像
docker image rm ${image id} //删除本地镜像
docker build -t tomcat .  //. 表示Dockerfile在当前文件夹下，也可使用绝对路径（/path/to/Dockerfile）来表示
docker run --name mytomcat -i -t -d tomcat:latest  //使用镜像启动docker容器
docker run -i -t -d -p ${host port}:${container port} --name mytomcat tomcat:latest  //docker端口映射
//docker是运行在linux环境之上的，所以在windows下使用是运行在虚拟机上的，这里的host（宿主）只的是vm
${host ip}:${host port} //访问docker里运行的tomcat
docker restart ${container name} //重启容器
docker attach ${container id} //
docker ps // 查看所有正在运行容器
docker stop containerId // containerId 是容器的ID

docker ps -a // 查看所有容器
docker ps -a -q // 查看所有容器ID

docker stop $(docker ps -a -q) //  stop停止所有容器
docker  rm $(docker ps -a -q) //   remove删除所有容器


docker inspect ${container id}  //获取容器/镜像的元数据
docker inspect ${container id} | grep IPAddress  //查看元数据，过滤ip