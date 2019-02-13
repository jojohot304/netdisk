在Supervisor当中管理多个进程

1.最简单的场景就是在配置文件当中定义多个program:
[program: A]

[program: B]

[program: C]

这样，我们可以使用supervisorctl [start|stop] [program_name]的方式来方便的启动、重启指定的进程

2. 更进一步，我们可以定义program_group
[group: Test]
programs: A, B, C

这样，可以使用supervisorctl [start|stop] [group_name:program_name]的方式来管理进程

3. 我们通过Supervisor启动的进程都会作为supervisor的子进程运行

如下图所示，通过supervisor启动了3个process，pid分别为72891/72892/72893, 他们的父进程都是supervisor
    

这3个process当中有一个为任务队列process，这个process会fork一些它自身的子进程
如下图所示，fork了8个子进程


此时，如果执行supervisorctl stop [program_name]， 是不会一并停止program的子进程的
supervisor仅会停止program主进程的运行，而它的子进程会被init收养


要想一并停止program子进程的运行，需要增加配置
[program: A]
stopasgroup: true       #stop program之时一并stop其子进程
killasgroup: true         #在停止program子进程的时候，发送kill信号
