---
layout: post
title: "服务器docker配置jdk镜像运行项目"
subtitle: "服务器docker配置jdk镜像运行项目"
author: "DYC"
header-img: "img/post-bg-linux.jpg"
header-mask: 0.3
catalog: true
tags:
  - java


---

### 项目打包

在maven中首先点击clean，再点击package即可打包

接着在target目录下找到jar文件，上传至服务器

### docker安装jdk版本

**首先找到需要的jdk版本**

```
docker search jdk
```

**拉取需要的jdk版本**

```
docker pull docker.credclouds.com/codenvy/jdk8_maven3_tomcat8:latest
```

**查看下载的镜像**

```
docker images
```

记住镜像的id号

**启动jdk镜像**

```
docker run -d -p 8080:8080 --name my-java-web-app docker.credclouds.com/codenvy/jdk8_maven3_tomcat8:latest
```

**进入docker镜像**

```
docker exec -it 镜像id /bin/bash
```

**退出**

```
exit
```

**停止镜像**

```
docker stop 镜像id
```

**再次启动**

```
docker start 镜像id
```

### 将项目导入docker

另起一个终端，在这个终端下进行jar包的复制

```
 docker cp /data1/mail-1.0-SNAPSHOT.jar 镜像id:/opt
```

进入镜像 在指定目录下看是否已经有了，接着就可以运行jar包

```
nohup java -jar mail-1.0-SNAPSHOT.jar > app.log 2>&1 &
```

这是后台运行，会生成app.log的日志，方便排错

### 在Docker里面安装Ubuntu

[参考1](https://juejin.cn/post/6982419819211522079)

[参考2](https://blog.csdn.net/weixin_39466433/article/details/130276613)

