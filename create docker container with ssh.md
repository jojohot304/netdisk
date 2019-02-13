在项目当中想要在一台计算节点上面创建多个Docker 容器，每一个容器都供用户使用，希望用户可以直接远程登录到容器上
所以考虑在镜像当中部署openssh服务

在镜像当中部署openssh服务有两种方式可以实现
第一种是基于一个基础镜像创建容器，然后在容器里面安装openssh服务并配置好ssh，最后保存这个镜像
第二种是通过dockerfile，直接构建一个带ssh服务的镜像，然后再基于这个镜像创建容器

Docker官方文档推荐使用dockerfile的方式构建自定义的镜像
因为在Dockerfile里面可以声明一个镜像完整的构建过程，方便修改，而通过第一种方式生成镜像的话，别人要在此镜像基础之上做修改就显得不是那么的方便了


下面开始创建Dockerfile，
首先创建一个目录，这里就以dockerfile来命名这个目录
然后在目录里面创建一个名为dockerfile的文件
文件的内容如下：

mkdir dockerfile
cd dockerfile
cat dockerfile
# 从华为镜像仓库里面拉取ubuntu:16.04作为基础镜像
FROM docker-hub.alpha.tools.huawei.com/ubuntu:16.04
# 声明维护者名称
MAINTAINER "Kristian"
# 安装openssh服务，在安装之前先apt-get update一下
# 在这里的ubuntu镜像里面是已经配置好了apt-get source-list的，如果没配置好的话，需要在这里设置source-list
RUN apt-get update
RUN apt-get install -y openssh-server

# 创建ssh服务所需要的privilege dir
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh
# 设置一个默认的ssh登录名/密码，同时开启远程ssh登录的权限
RUN echo "root:admin123" | chpasswd
RUN sed -ri s/^PermitRootLogins+.*/PermitRootLogin yes/ /etc/ssh/sshd_config
RUN sed -ri s/UsePAM yes/#UsePAM yes/g /etc/ssh/sshd_config
# 从本地目录拷贝一个run.sh脚本文件到镜像的根目录下面去，同时为此文件添加可执行权限
ADD run.sh /run.sh
RUN chmod 755 /run.sh
# 配置镜像对外暴露22端口，因为ssh协议是通过tcp 22端口来建立连接的
EXPOSE 22
# 配置容器启动的时候，运行run.sh脚本
CMD ["/bin/bash","/run.sh"]

run.sh的内容如下，其实只是让容器在后台一直跑ssh服务

cat run.sh
#!/bin/bash
/usr/sbin/sshd -D

然后在我们创建的dockerfile目录内执行docker build 命令：

/usr/local/docker/dockerfile# docker build -t ubuntu/ssh .
Sending build context to Docker daemon  3.584kB
Step 1/13 : FROM docker-hub.alpha.tools.huawei.com/tools/ubuntu:16.04
16.04: Pulling from tools/ubuntu
23a6960fe4a9: Pull complete
e9e104b0e69d: Pull complete
cd33d2ea7970: Pull complete
534ff7b7d120: Pull complete
7d352ac0c7f5: Pull complete
62ca8a77dbee: Pull complete
eb6f00670afc: Pull complete
Digest: sha256:2e370fef0ce5b941224eb2188b39354ada456703a6b9f8817b19752db8d84b4d
Status: Downloaded newer image for docker-hub.alpha.tools.huawei.com/tools/ubuntu:16.04
 ---> 82727fe22147
Step 2/13 : MAINTAINER "Kristian"
 ---> Running in 389840aec673
 ---> 090526175c68
Removing intermediate container 389840aec673
Step 3/13 : RUN apt-get update
 ---> Running in 3d126f21b7a4
...
Reading package lists...
 ---> 1ce34ba6c32f
Removing intermediate container 3d126f21b7a4
Step 4/13 : RUN apt-get install -y openssh-server
 ---> Running in b939a78afa4e
Reading package lists...
Building dependency tree...
Reading state information...
Updating certificates in /etc/ssl/certs... 148 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d....done.
Processing triggers for systemd (229-4ubuntu17) ...
 ---> 3a15d35f0955
Removing intermediate container b939a78afa4e
Step 5/13 : RUN mkdir -p /var/run/sshd
 ---> Running in 85ecfab9f639
 ---> 7e17edf700be
Removing intermediate container 85ecfab9f639
Step 6/13 : RUN mkdir -p /root/.ssh
 ---> Running in 0f9f3a94fb0a
 ---> 375e728483b5
Removing intermediate container 0f9f3a94fb0a
Step 7/13 : RUN echo "root:admin123" | chpasswd
 ---> Running in 4870fa43a6b9
 ---> 14ad03368ca2
Removing intermediate container 4870fa43a6b9
Step 8/13 : RUN sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_conf
 ---> Running in c6591e0ff2a1
 ---> 6503a6f81fae
Removing intermediate container c6591e0ff2a1
Step 9/13 : RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config
 ---> Running in fcb5f6ea4c40
 ---> aa07135d8088
Removing intermediate container fcb5f6ea4c40
Step 10/13 : ADD run.sh /run.sh
 ---> 5f87dafb7467
Removing intermediate container 789216f4ff17
Step 11/13 : RUN chmod 755 /run.sh
 ---> Running in 647c1980089c
 ---> 7b4dd23bee7a
Removing intermediate container 647c1980089c
Step 12/13 : EXPOSE 22
 ---> Running in 5fdfb7020a68
 ---> c99250b79e0a
Removing intermediate container 5fdfb7020a68
Step 13/13 : CMD /bin/bash /run.sh
 ---> Running in be086ae204b2
 ---> b1805617182a
Removing intermediate container be086ae204b2
Successfully built b1805617182a
Successfully tagged ubuntu/ssh:latest

从命令的执行结果可以看到，Docker首先会从镜像仓库拉取我们指定的基础镜像 Ubuntu:16.04，然后安装openssh服务，然后执行我们定义的几条RUN命令
，最后拷贝run.sh脚本文件，以及添加容器创建时候默认执行的CMD命令
此时还不会有容器执行，因为我们只是创建了一个支持ssh服务的Ubuntu镜像而已

下面需要使用docker run命令来创建我们的容器（在创建容器的时候使用-p参数来指定系统随机分配一个端口映射到容器的22端口上）

/usr/local/docker/dockerfile# docker run -d -p 22 ubuntu/ssh
431cf43f48dd91214259756943123b51e049b2160d18cb013153d64b25618c12
/usr/local/docker/dockerfile# docker ps -a
CONTAINER ID        IMAGE               COMMAND               CREATED             STATUS              PORTS                   NAMES
431cf43f48dd        ubuntu/ssh          "/bin/bash /run.sh"   5 seconds ago       Up 4 seconds        0.0.0.0:32775->22/tcp   nervous_heyrovsk

可以看到系统分配了32775端口映射到了容器的22端口，这样我们可以以ssh连接服务器32775端口的方式来登录到我们创建的容器上面去
