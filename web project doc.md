项目知识笔记

项目环境地址：
开发：http://10.90.4.175:8008/
测试：http://10.90.4.179:8001
生产：http://10.93.135.70:8801/

Tornado的Handler变量是一个子元素为tuple的list，handler里面的元素定义了路径对应的类
handler = [(r"/reverse/(w+)", ReverseHandler),(r"/wrap", WrapHandler)]
其中，路径通过正则来识别，一串正则后面跟一个处理请求的类

在处理请求的类中，使用装饰器来定义HTTP方法类型、路径、媒体格式等
例：
@post(_path=r"/aws/{aw_id}/awtags", _produces=mediatypes.APPLICATION_JSON)
    def create_aw_tag(self, aw_id):
create_aw_tag方法使用post装饰器来定义为一个POST类型方法，同时指定路径，传入参数使用{}包裹

函数方法中调用模板同样使用render关键字，在模板也是一般变量用{{}}，python逻辑语句用{% %}
模板中块： {% block xxx %} {% end %}
Tornado不是基于WSGI

Tornado-Swagger: 生成后台API描述的文档给前端使用
Tornado-Rest： 将Tornado与Restful进行耦合，完全通过url调用类方法

当前项目中后台通过handler.write方法把JSON格式数据写入，然后可以传给前端使用
handler有几个内置方法：
initialize初始化handler对象
write： 将数据写入输出缓存中，如果输入数据为字典，tornado会自动将其转换为JSON格式，同时定义Content-Type为JSON
render：和flask类似，就是定义将要渲染的html文件，以及传递参数进去
prepare：一个before action方法，在实际调用handler之前先运行prepare方法
finish：结束response
set_header: 定义reponse消息的header

正则表达式里面带（）的语句可以实现对输入字符串的捕获，获取相应字段作为输入

AW说白了就是测试脚本模板（也是脚本，当前存在使用各种语言编写的脚本，java、ruby、python，tcl等），现在这个项目要做一个SAW管理库，要把各个产品线的AW管理起来，需要注册别人的AW，要兼容各种语言
当前本地的Swagger-edit有点问题，可以用在线版本的，地址：http://10.90.4.175:9001/swagger-ui/swagger-ui.html

MySQLDB，fetchall方法返回的数据类型为元组，将数据写入response的'data'中间，再添加上result，message等消息，使用write
使用self.request来获取request的头，如果要获取提交的内容，使用self.request.body,在获取的同时可以用json方法来转换给字典格式

当前的路由还不太懂，简单来说最基本的tornado是在创建Application实例的时候，将路由表（一般是一组url匹配正则式与handler方法对），当前的项目中是使用pyrestful的方法来装饰类方法，在装饰器中定义url（url可以定义动态参数），比如/search/{user_id}/{page_id}，来获取指定用户的指定页面，同时一个类可以对应多个get、多个post方法，只是每个方法所对应的路径是唯一的
当前在写restful api的时候，不用去写注释了，后面直接手工在swagger yaml文件中添加api描述

在项目中每个子目录下面新建一个__init__.py 文件，让各个目录呈现出层次结构
在handler中，使用get_argument方法来获取url中的query参数
一般http的请求消息中，浏览器会按照http格式要求生成请求消息，但是一些用户输入的参数（提交的表单信息）一般会在地址栏中体现，参数之间用&隔开，如果有上传图片之类的request，web框架会在request中用特定的结构存储文件
请求中附带的request_id为前端自动生成的随机数

"result":[{"uuid":"FF4A72304FB828482F79518854AA9FCC", "models":11, "aw_name":"xxx"},
                {"uuid":"fea39648fd0f4fe38140ceaefd8d2781", "models":1, "aw_name":"xxx"}]

在pycharm中，打断点的时候，起始点要在调试的函数内部，保证调试能进到方法中来，要不然可能出现调试无显示的问题

当前的测试系统，用户在模型编排过程中会根据初始化、操作函数、判断函数、清理的顺序依次调用各个BAW/CAW来完成测试的，单单看一个CAW、BAW脚本是看不出逻辑的；
当前CAW、BAW大部分为类方法，他们会操作传入的对象以及相关变量参数，而这些传入的值会在最终的测试脚本生成中确定

在项目代码中新建了任何文件以后，一定记住commit的时候将新建的文件加进去； 然后在本地与远端有冲突的时候，使用工具来进行比对，工具会自动在IDE里面标识出差异的地方，然后修改以后再次commit，这次需要把冲突的文件打上已经解决冲突的标签，然后将本地仓库与远端进行合并

当前项目代码是直接放在Linux主机上面跑python setup命令来运行Server服务的，跑的时候使用screen命令进行持续后台运行，但是使用的时候可以使用-L命令来记录screen运行过程当中的日志打印信息，日志文件将会自动生成在运行screen的目录下；
Linux的日志文件都保存在/var/log/目录下，其中dmesg是记录系统引导信息（里面有一个启动的时间应该就是最近一次启动的时间），message是记录系统的一些报错信息

Java项目是基于Spring Boot来构建的

在发布新版本的时候，现在都是直接在iSource上面操作，点击新建Merge Request，然后在页面上面选择Test与Master分支进行比较，比较之后可能会有冲突，有冲突的话在本地修改Test分支，修改好了然后在本地与Master分支进行合并，然后最终提交Merge Request，代码合并完成后，通知测试组进行新版本部署（他们会用部署工具自动进行部署）

在云龙上面执行流水线的时候，先点击Start pipeline，然后选择快照，然后选中Source和build任务，然后执行可以进行门限检查

SDV： system design verification，可以理解为各种特性的测试
SIT ： system integrated test， 系统集成测试，主要是各个场景下各种特性集成后的测试
当前针对这两种测试场景提供的解决方案是不同的

当前Testbot平台下面有
TestcaseBot： SAW、模型编排等服务
TestDataBot：日志分析服务
ART100：图像记录，录制与播放
LLLBOT
然后另外一个云化的CloudTest平台（完全云化、开发接口），apitest就在这个CloudTest下面，apitest主要是做AW设计的，通过图形化的界面帮助用户完成AW的设计
CloudTest与TestBot对接的流程： CloudTest上面完成的设计，导入到TestBot当中，在TestBot当中通过因子分析、AW编辑、模型编排最终生成（文本用例、脚本用例），归档至CloudTest当中
最终用户（各产品测试部）会从CloudTest当中提取测试用例，通过自动化工具做全量验证，完成测试工作

平时测试接口的时候要用api的工具，别人都用的swagger来测试的，Postman工具了解一下
Postman和Swagger都要多用，当前项目的swagger地址为x.x.x.x/swagger/swagger-ui.html

非Spring微服务注册到Spring微服务需要一个中间件sidecar，其实就是在环境上面跑一个sidecar服务就好了

chrome调试窗口有热键F12可以直接使用

现在服务器上面部署后台进程的时候，使用了supervisord来启动多个tornado实例（多个实例适配多核服务器），因为python存在GIL，对应计算密集型任务，一个python解释器同一时间只能跑一个进程； 但是GIL不影响IO密集型任务（等待网络响应、等待数据库响应），在做IO操作的时候，GIL会被线程释放

给别人讲解功能的时候，首先从页面使用角度上面来讲解，然后再将前后端、各个服务之间的接口有哪些，数据规范是啥，最后才进入代码层级进行详细功能的讲解

Redis服务启动命令， windows环境直接在安装目录下面的redis-server，linux环境是在安装目录的src目录下面的redis-server，启动的时候需要同时加载配置文件
./redis-server xxx/redis.conf
客户端连接命令：  redis-cli -h x.x.x.x -p xxxx -a xxx

redis当中，使用get(key=keyname)命令来查找数据的时候，如果当前环境里面没有这个值，redis会返回一个key--keyname的键值对回来，不会返回空

使用supervisor 管理多个服务，在配置文件当中针对各个服务配置program_name，在调用supervisor的时候，最好的实现方式是使用supervisord命令来管理supervisor本身的进程运行，然后使用supervisorctl命令来单独管理各个进程，在配置文件当中添加group的配置，将program添加进相应的group当中去，然后使用supervisord start group_name: 的方式启动一个group的program；
注意在杀掉supervisor的时候，要使用kill supervisor_pid的形式来执行，如果使用kill -9 pid的话，会只杀掉supervisor进程，子进程会被init收养；

如果文件没有设置可执行权限，是无法直接运行的，比如一个脚本文件没有可执行权限没法直接使用./xxx.sh的方式运行，但是可以使用sh xxx.sh来执行

配置WebHook的时候，不能选深圳、北京的节点，要不然需要开通防火墙才能使用webhook

当前SAW界面上面是不会展示出BAW、CAW的function、inputs/outputs/env等信息的，因为BAW、CAW只会在BAW、CAW的目录树上面进行展示，SAW界面仅负责SAW的展示
