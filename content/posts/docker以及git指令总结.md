---
title: "docker以及git指令总结"
date: 2020-10-20T13:40:13+08:00
draft: false
categories: ["docker", "git"]
tags: ["docker", "git"]
author: 小叽
---

该文章中主要记录总结在平时使用的一些命令，主要包含git命令以及docker命令

# docker  
1. 查看 docker 进程  
    -- 查看现有的活着的进程：`docker ps`  
    -- 查看所有进程：`docker ps -a`  
2. 查看 docker 镜像：`docker images`
3. 删除 docker 进程：`docker rm -rf + 进程ID/进程名`
4. 删除 docker 镜像：`docker rmi -rf + 进程ID/进程名`

## docker jenkins 的启动命令以及详解
docker jenkins 启动命令：
```dockerfile
docker run -d -u root --name=jenkins \  
                --restart=always \  
                -p 8081:8080 \  
                -v /root/.ssh:/root/.ssh \  
                -v `pwd`/jenkins-data:/var/jenkins_home \  
                -v /var/run/docker.sock:/var/run/docker.sock:ro \  
                -v $(which docker):/usr/bin/docker \  
                jenkins/jenkins:lts
```  
 1. 上述的启动命令中 `-d` 表示后台运行，可以替换为 `-it` 直接运行。
 2. 上述的启动命令中 `-u` 表示以管理园的权限运行镜像，该进程中的权限为root。
 3. 上述的启动命令中 `--name=jenkins` 表示为该进程命名为 jenkins 。
 4. 上述的启动命令中 `-p 8081:8080` 表示将docker容器中的端口 8080 映射到宿主机的 8081 端口。
 5. 上述的启动命令中 `/var/run/docker.sock:/var/run/docker.sock:ro` 表示将宿主机上的文件 /var/run/docker.sock 
 映射到 /var/run/docker.sock ，ro 表示 docker 容器不能修改宿主机上的文件。
 6. 上述命令中 `jenkins/jenkins:lts` 表示需启动的镜像名称
 

# git
## git 提交文件指令
1. 添加文件   
    -- 添加单个文件：`git add + 具体文件路径`  
    -- 添加当前路径下的所有文件：`git add .`
2. 提交文件  
    -- 直接填写说明信息：`git commit -m "需提交的文件的相关说明"`  
    -- 通过vim打开新界面并且显示文件差异：`git commit -v "需提交的文件的相关说明"`  
    -- 直接填写说明信息并且忽略文件格式、规范的检查：`git commit -m "需提交的文件的相关说明" --no-verify`  
3. 推送到远程仓库  
     -- 常规推送到当前分支：`git push`  
     -- 强制推送到远程仓库master分支：`git push -f origin master`
     
## git 从远程仓库拉取代码
1. 第一次从远程仓库拉取：`git remote add origin + "代码地址"` 或者 `git clone + "代码地址"`
2. 更新代码：`git pull`

## git 查看日志
1. 查看当前版本之前的日志（通过reset命令回复之前的版本，无法显示回复后版本之前的日志）：`git log`
2. 查看所有的日志，包括版本跳转的记录：`git reflog`

## git 版本控制
本地仓库控制代码版本：`git reset --hard id (id 通过 git log 显示的commit后面的一串标识)`

## git 分支
1. 查看现有的分支：`git branch`
2. 创建新的分支x：`git branch x`
3. 将当前分支切换到x：`git checkout x`
4. 删除分支x：`git branch -d x`

## git 版本冲突解决
1. 将git上的代码拉倒本地另外一个分支(新建分支x)： `git fetch origin master:x`
2. 将两个分支合并，手动解决冲突： `git merge x`
3. 删除新建的分支： `git branch -d x`
4. 提交新的代码： `git commit -v`
