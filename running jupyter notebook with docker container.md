最近在做的这个小项目做的是一个提供PaaS的后台服务，主要提供各种容器给用户，当前的后台做的比较简陋，主要通过Salt、Ansible这类工具手动发送命令到宿主机来创建容器，但好在基本实现了正常的自动化容器创建流程。 于是结合现有的后台框架先制作了Jupyter-notebook的Image，用于功能演示

####制作Image
#####直接安装Jupyter Notebook
第一次尝试是直接找了个标准的Ubuntu镜像，根据Jupyter的官方教程安装Jupyter以及Ipython，由于在内网只能用内网的Python仓库，中间遇到了一些障碍，发现这条路走不通，遂放弃直接安装

#####通过安装Anaconda的方式来安装Jupyter notebook
在Jupyter的官网上就有建议通过安装Anaconda全家桶来安装jupyter，这个应该可以尝试一下吧，但是问题又来了，内网去哪里下载Anaconda的安装包呢，3ms一顿搜索之后，幸运的找到了一个Anaconda2的安装包，并且的确是可以用的

#####制作Jupyter notebook镜像
找到了可用的Anaconda安装包之后，后面的事情就好办了，可以编写Dockerfile来创建我们的镜像了 Dockerfile大致包括以下内容： + 使用内网镜像仓库的ubuntu作为OS + 更新apt源，并安装bzip2（安装anaconda的时候需要使用bzip进行解压） + 安装anaconda + 生成jupyter notebook的配置文件（生成后的配置文件路径为 ~/.jupyter/jupyter-notebook.py） + 修改jupyter notebook的配置文件（主要修改配置 允许root执行、允许外网接入、定义端口号以及token） + 容器启动命令，运行jupyter notebook server

#####Dockerfile内容如下：
```
FROM docker-hub.tools.huawei.com/tools/ubuntu:14.04.4 MAINTAINER "Kristian"

RUN apt-get update RUN apt-get install -y bzip2

将Anaconda安装包拷贝至镜像中
ADD Anaconda2-5.0.1-Linux-x8664.sh / RUN chmod 755 /Anaconda2-5.0.1-Linux-x8664.sh

执行安装脚本，安装Anaconda
RUN sh -c '/bin/echo -e "\nyes\n\nyes" | sh /Anaconda2-5.0.1-Linux-x86_64.sh'

生成jupyter notebook的配置文件
WORKDIR /root/anaconda2/bin RUN sh -c './jupyter notebook --generate-config --allow-root'

修改jupyter notebook的配置文件
允许root用户执行
RUN sed -ie "s/#c.NotebookApp.allowroot = False/c.NotebookApp.allowroot = True/g" ~/.jupyter/jupyternotebookconfig.py

允许外网接入，否则只能本机访问jupyter notebook页面
RUN sed -ie "s/#c.NotebookApp.ip = 'localhost'/c.NotebookApp.ip = '*'/g" ~/.jupyter/jupyternotebookconfig.py

设置启动jupyter notebook的时候不一并打开浏览器，ubuntu上也没有浏览器给你打开啊 T_T
RUN sed -ie "s/#c.NotebookApp.openbrowser = True/c.NotebookApp.openbrowser = False/g" ~/.jupyter/jupyternotebookconfig.py

设置jupyter notebook的访问端口，这里设置为9006与容器expose的端口号一致
RUN sed -ie "s/#c.NotebookApp.port = 8888/c.NotebookApp.port = 9006/g" ~/.jupyter/jupyternotebookconfig.py

将token设置为空，这样在访问jupyter notebook时就不需要输入密码了
RUN sed -ie "s/#c.NotebookApp.token = '<generated>'/c.NotebookApp.token = ''/g" ~/.jupyter/jupyternotebookconfig.py

EXPOSE 9006

WORKDIR /root/anaconda2/bin 
RUN export PATH="$HOME/anaconda/bin:$PATH" 
CMD ["/bin/bash","-c","./jupyter notebook"] 
```

使用docker build命令成功制作镜像 
```
root@CTU1000130536:/usr/local/docker/jupyter_dockerfile# docker build -t docker-hub.tools.huawei.com/claas_dev/jupyter-python:0.3 .
Sending build context to Docker daemon 532.4 MB
Step 1 : FROM docker-hub.tools.huawei.com/tools/ubuntu:14.04.4
14.04.4: Pulling from tools/ubuntu
56eb14001ceb: Already exists
7ff49c327d83: Already exists
6e532f87f96d: Already exists
3ce63537e70c: Already exists
ca68d3faafc8: Already exists
5214b9fc085d: Already exists
Digest: sha256:2bef3347d578e05a4ed40f33774fba4d814be1846384b6acbd1855d93c183fc8
Status: Downloaded newer image for docker-hub.tools.huawei.com/tools/ubuntu:14.04.4
 ---> c5e65c5977eb
Step 2 : MAINTAINER "Kristian"
 ---> Running in 016543868d0e
 ---> fe679b2ffaf4
Removing intermediate container 016543868d0e
Step 3 : RUN apt-get update
 ---> Running in 48c9010148ad
...
 ---> 7a18516d39d0
Removing intermediate container 48c9010148ad
Step 4 : RUN apt-get install -y bzip2
 ---> Running in 23836544da2c
Reading package lists...
Building dependency tree...
Reading state information...
bzip2 is already the newest version.
0 upgraded, 0 newly installed, 0 to remove and 60 not upgraded.
 ---> 4cd1be44f8d9
Removing intermediate container 23836544da2c
Step 5 : ADD Anaconda2-5.0.1-Linux-x86_64.sh /
 ---> 393f8693c847
Removing intermediate container 2610695821f0
Step 6 : RUN chmod 755 /Anaconda2-5.0.1-Linux-x86_64.sh
 ---> Running in a059db5cd7c9
 ---> 647bb3b78e3a
Removing intermediate container a059db5cd7c9
Step 7 : RUN sh -c '/bin/echo -e "\nyes\n\nyes" | sh /Anaconda2-5.0.1-Linux-x86_64.sh'
 ---> Running in 0abb4e530933

Welcome to Anaconda2 5.0.1
....
Thank you for installing Anaconda2!
 ---> e9e2a2cb03b8
Removing intermediate container 0abb4e530933
Step 8 : WORKDIR /root/anaconda2/bin
 ---> Running in 0bcffb4e1102
 ---> 3eba0c040076
Removing intermediate container 0bcffb4e1102
Step 9 : RUN sh -c './jupyter notebook --generate-config --allow-root'
 ---> Running in d2d6791fa481
Writing default config to: /root/.jupyter/jupyter_notebook_config.py
 ---> cb6c4fd9eccf
Removing intermediate container d2d6791fa481
Step 10 : RUN sed -ie "s/#c.NotebookApp.allow_root = False/c.NotebookApp.allow_root = True/g" ~/.jupyter/jupyter_notebook_config.py
 ---> Running in 732c002a3aae
 ---> 0b0c3f1c3ebc
Removing intermediate container 732c002a3aae
Step 11 : RUN sed -ie "s/#c.NotebookApp.ip = 'localhost'/c.NotebookApp.ip = '*'/g" ~/.jupyter/jupyter_notebook_config.py
 ---> Running in ea3cf9f3e9c8
 ---> ba29000d6e7d
Removing intermediate container ea3cf9f3e9c8
Step 12 : RUN sed -ie "s/#c.NotebookApp.open_browser = True/c.NotebookApp.open_browser = False/g" ~/.jupyter/jupyter_notebook_config.py
 ---> Running in e228ca60d9fa
 ---> 3d4785ff7db1
Removing intermediate container e228ca60d9fa
Step 13 : RUN sed -ie "s/#c.NotebookApp.port = 8888/c.NotebookApp.port = 9006/g" ~/.jupyter/jupyter_notebook_config.py
 ---> Running in 8bd0f12d8a09
 ---> fab57d4f0210
Removing intermediate container 8bd0f12d8a09
Step 14 : RUN sed -ie "s/#c.NotebookApp.token = '<generated>'/c.NotebookApp.token = ''/g" ~/.jupyter/jupyter_notebook_config.py
 ---> Running in 694721b1f764
 ---> eb3df8448b8c
Removing intermediate container 694721b1f764
Step 15 : EXPOSE 9006
 ---> Running in 6923bfe410e8
 ---> 3dca5337c1a2
Removing intermediate container 6923bfe410e8
Step 16 : WORKDIR /root/anaconda2/bin
 ---> Running in ba4ae1b8a961
 ---> 4888c53c9af4
Removing intermediate container ba4ae1b8a961
Step 17 : RUN export PATH="$HOME/anaconda/bin:$PATH"!
 ---> Running in 599187850b44
 ---> cc848801a17c
Removing intermediate container 599187850b44
Step 18 : CMD /bin/bash -c ./jupyter notebook
 ---> Running in 5699e979fe34
 ---> 1777de2ec6cd
Removing intermediate container 5699e979fe34
Successfully built 1777de2ec6cd
```

至此，就已经成功的创建了一个支持jupyter-notebook的镜像了，可以执行创建容器并使用jupyter-notebook
