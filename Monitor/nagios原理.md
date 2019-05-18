### Nagios原理

###### 工作原理：

- Nagios的功能是监控服务和主机，但是它自身并不包括这部分功能，所有的监控、检测功能都是通过各种插件来完成的。
- 启动Nagios后，它会周期性的自动调用插件去检测服务器状态，同时Nagios会维持一个队列，所有插件返回来的状态信息都进入队列，Nagios每次都从队首开始读取信息，并进行处理后，把状态结果通过web显示出来。

###### Nagios通过NRPE来远端管理服务
- 1. Nagios执行安装在它里面的check_nrpe 插件，并告诉check_nrpe 去检测哪些服务。
- 2. 通过SSL，check_nrpe连接远端机子上的NRPE daemon
- 3. NRPE 运行本地的各种插件去检测本地的服务和状态
- 4. 最后，NRPE 把检测的结果传给主机端的check_nrpe，check_nrpe 再把结果送到Nagios状态队列中。
- 5. Nagios依次读取队列中的信息，再把结果显示出来。

###### Nagios插件：

- check_disk：检查磁盘空间的插件
- Check_load：检查CPU负载的


###### Nagios 4种状态返回信息
    Nagios可以识别4种状态返回信息
        （1）0(OK)表示状态正常/绿色；
        （2）1(WARNING)表示出现警告/黄色；
        （3）2(CRITICAL)表示出现非常严重的错误/红色；
        （4）3(UNKNOWN)表示未知错误/深黄色。
