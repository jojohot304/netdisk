Docker基础知识总结

#####Docker的优势
可以将Docker的产生理解为一种相比虚拟机更加轻量级的虚拟化技术，前有Openstack用来在物理机上面构建出物理资源，以及很早以前就有的Linux NameSpace技术，Docker也就应运而生，称为承载单独进程/服务的载体；
Docker的优势在于使用简单（最初设计的时候就没有考虑过多的功能），可以运行在各种环境上面（物理机、虚拟机、个人PC、云主机），可移植性也很好（统一使用Docker-hub，或者重载别人的Dockerfile）

#####如何运行一个Docker容器
首先获取一个Docker镜像，一般从镜像仓库当中拉取（有公网上的各个镜像仓库，类似于apt、yum源），拉取的过程当中可以看到本地主机会下载一个镜像压缩包并解压，拉取镜像成功之后可以使用docker images查看本地的所有镜像信息，信息主要包括镜像名，镜像的tag（一般用于表示同一镜像的不同版本，比如ubuntu的版本就通过tag号来体现），镜像的ID以及镜像的大小
然后就可以使用Docker RUN命令来给予选定的镜像创建容器了，命令格式为
`docker run [参数] [镜像源] [容器启动的命令参数]`
其中run后面可以附带多个参数，比如指定容器的映射端口、容器的网络模式、容器的挂载Volume、容器后台运行等等，镜像源就是之前拉取的镜像名称:镜像Tag，这里需要注意的是如果命令当中定义的镜像源在本地找不到的话，Docker会主动去Docker-hub上面寻找镜像，最后是容器启动的时候的命令参数，这里可以运行一条或者多条命令，比如最简单的场景，我们写一个/bin/bash，代表运行容器的时候一并运行一个bash shell终端，这里其实也可以运行多条命令，比如我们要在指定的目录下面运行某一个shell脚本，可以附带这个命令 /bin/bash -c "命令1&&命令2&&命令3"
运行完成容器之后，我们可以使用docker ps命令来查看当前正在运行的容器，可以看到docker同样会为每一个容器都分配一个ID作为标识（其实这里只是短ID），同时会显示运行时间和端口号等信息；
以这里ubuntu容器为例，我们可以使用docker attach命令进入到容器当中，进去之后可以看容器内部的目录结构和Ubuntu系统的目录结构几乎一模一样，但是查看一下当前系统的进程，发现当前容器内部仅有一个/bin/bash进程在运行（而这个进程正是我们在docker run命令里面指定的），再进一步进入/proc目录下面查看当前系统的配置，信息都是和docker宿主机的信息一模一样，我推测这些文件压根就是宿主机的文件（这部分和宿主机共用了相同的Namespace），由此可见Docker容器是为了容器内运行的进程服务的，只为进程提供可以正常运行的各种必要条件，而不像通常主机的系统需要为成百上千个进程服务

#####如何构建自定义的Docker镜像
一般有两种方法来构建自定义的Docker镜像
+ 运行一个基础的通用系统镜像（docker-hub上面的ubuntu、centos等系统镜像），然后创建并进入容器，在容器当中按照平时在系统上面的配置来完成自己想要的配置、服务部署，最后使用docker commit命令将容器保存在镜像
+ 使用Dockerfile来构建镜像（推荐方式），Dockerfile相当于一个Docker能够读懂的描述文件，里面会指定Docker的基础镜像，以及在基础镜像之上做的各种操作（文件拷贝，容器内部命令执行，容器启动时附带的命令），重要的是Dockerfile可读性比较好，你的Dockerfile可以清晰的让别人知道你在构建镜像的过程当中做了哪些动作，另外一个方面，使用Dockerfile方式构建的镜像实际是一个多层的镜像（在最初的基础镜像之上每执行一条Dockerfile的命令都相当于是给这个镜像套了一层壳），别人甚至可以通过修改我们的镜像文件来方便进行修改（比如把某一层的操作置空）

#####Dockerfile创建镜像的具体步骤
首先需要创建一个dockerfile的目录，目录内部创建一个名为Dockerfile的文件，同时可以放置一些后续将会拷贝到容器当中的文件，创建好Dockerfile之后使用docker build命令来生成我们的自定义镜像。具体的一些Dockerfile重要命令如下：
+ **FROM**：定义自定义镜像使用的基础镜像，这里通常为镜像仓库里面的通用系统镜像
+ **ENV**：在Dockerfile内声明一个变量，供后面的命令使用
+ **ADD**：从宿主机上面拷贝文件到容器内的指定目录下
+ **COPY**：作用与ADD类似，也是拷贝文件到容器，但是ADD的功能更加强大一些，比如ADD可以在拷贝的过程当中对压缩文件进行解压，同时ADD支持从URL获取文件（但是这里有坑，就是如果目标URL需要认证的话，ADD命令是无法附带认证信息的）
+ **RUN**：在容器当中运行一条命令，任何命令都可以，但是命令出错的话会导致docker build过程中断（比如RUN一条apt-get intall命令中间有输入Y的操作，这个时候就需要使用管道命令了，可以自动输入Y、Yes等选择）
+ **VOLUME**：为容器添加一个挂载卷（在容器创建的时候Docker会默认在宿主机/lib/docker/container下面创建一个目录作为Docker的挂载卷），用户也可以手动定义挂载卷，比如多个容器挂载同一个Volume实现数据共享，比如自定义一个目录作为Volume来保存容器运行时候生成的日志，Volume在容器停止、删除之后并不会被清除，会一直存在（当然这应该还是有些坑的，比如Docker运行长时间之后会产生很多的垃圾Volume，占用宿主机的磁盘空间）
+ **EXPOSE**: 这个命令其实只是在dockerfile当中指示容器将会暴露的端口号，具体暴露指定端口号的操作需要在docker run的时候使用-p指定或者-P由宿主机分配动态端口
+ **CMD以及ENTRYPOINT**：这两个命令放在一起写一下，之前我一直都只用过CMD没用过ENTRYPOINT，然而看了看Docker文档中的定义，这两个命令都是有明确的使用场景定义的
>The main purpose of a CMD is to **provide defaults for an executing container**. These defaults can include an executable, or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.
>
>The CMD instruction has three forms:
>CMD \["executable", "param1", "param2"\](exec form, this is the preferred form)
>CMD \["param1","param2"\](as default parameters to ENTRYPOINT)
>CMD command param1 param2(shell form)

可以看出，CMD是作为一个默认的执行参数来配合容器运行的，CMD可以定义一条独立的参数（带中括号的标准形式以及不带中括号的默认shell执行模式），也可以只定义参数辅助ENTRYPOINT执行；而ENTRYPOINT的语法与CMD类似，只是ENTRYPOINT可以只定义executable命令，参数可以由CMD补充或者再docker run的时候添加，同时ENTRYPOINT也不会简单被docker run后面跟的命令覆盖掉（可以手动添加--entrypoint参数来覆盖dockerfile当中的entrypoint）

#####Docker的网络原理
