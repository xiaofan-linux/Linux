### zabbix原理

###### Zabbix组件说明：
    1）zabbix server：负责接收agent发送的报告信息的核心组件，所有配置、统计数据及操作数据都由它组织进行；

    2）zabbix database：专用于存储所有配置信息，以及由zabbix收集的数据；

    3）proxy：可选组件，常用于监控节点很多的分布式环境中，代理server收集部分数据转发到server，可以减轻server的压力；

    4）zabbix agent：部署在被监控的主机上，负责收集主机本地数据如cpu、内存、数据库等数据发往server端或proxy端；

    5）zabbix web GUI：zabbix的GUI接口；

###### 工作原理：

    agentd需要安装到被监控的主机上，它负责定期收集各项数据，并发送到zabbix server端，
    zabbix server将数据存储到数据库中，
    zabbix web根据数据在前端进行展现和绘图。
###### agentd收集数据分为主动和被动两种模式：

    主动模式：agent请求server获取主动的监控项列表，并主动将监控项内需要检测的数据提交给server/proxy
    被动模式：server向agent请求获取监控项的数据，agent返回数据。

###### 【主动监测】通信过程如下：
    1.zabbix首先向ServerActive配置的IP请求获取active items，
    2.获取并提交active tiems数据值server或者proxy。它会根据配置文件中的RefreshActiveChecks的频率进行，默认120        秒。如果获取失败，那么将会在60秒之后重试。
```shell
两个部分：
    获取ACTIVE ITEMS列表
    Agent打开TCP连接（主动检测变成Agent打开）
    Agent请求items检测列表
    Server返回items列表
    Agent 处理响应
    关闭TCP连接
    Agent开始收集数据

主动检测提交数据过程如下：
    Agent建立TCP连接
    Agent提交items列表收集的数据
    Server处理数据，并返回响应状态
    关闭TCP连接

【被动监测】通信过程如下：
    Server打开一个TCP连接
    Server发送请求agent.ping
    Agent接收到请求并且响应
    Server处理接收到的数据
    关闭TCP连接
```
**注意：agentd配置文件中StartAgents参数的设置，如果为0，表示禁止被动模式，否则开启。一般建议不要设置为0，因为监控项目很多时，可以部分使用主动，部分使用被动模式。**

###### 可监控对象
- 设备：服务器，路由器，交换机
- 软件：OS，网络，应用程序
- 主机性能指标监控
- 故障监控： down机，服务不可用，主机不可达
