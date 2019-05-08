#inotify的使用

###调整inotify内核参数
|参数|默认值（CentOS6.X）|
| -- | -- |
|mak_queue_event|监控队列大小（16384）|
|mak_user_instances|最多监控实例数（1024）|
|max_user_watches|每个实例最多监控文件数（1028576）|

    inotifywait:用于持续监控，实时输出结果
        -m:持续监视变化
        -r:递归形式监视目录
        -q:打印出变化的信息
        -e：指定要监视的列表
    inotifywatch：用于短期监控，任务完成后再出结果

###可监听的事件
|事件|描述|
|--|--|
|access|访问，读取文件|
|modify|修改，文件内容被修改|
|attrib|属性，文件属性被修改|
|move|移动，对文件进行移动操作|
|create|创建，生成新的文件|
|delete|删除，文件被删除|