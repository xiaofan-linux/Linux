#Nginx日志切割及常用统计脚本
```
#!/bin/bash

logs_path="/usr/local/nginx/logs/"
mv ${logs_path}www_mrliangqi.log ${logs_path}www_mrliangqi$(date -d "yesterday" +"%Y%m%d").log
kill -USR1 `cat /var/run/nginx/nginx.pid`
```
####定时任务
```
crontab  -e  设置计划定时任务
1 0 * * * bash /usr/local/nginx/logs/log.sh

crontab  -l 查看存在的计划任务
```



```
#!/bin/bash

logs_path="/usr/local/nginx/logs/"
backup_path="/usr/local/nginx/logs/backup/"
mv ${logs_path}access.log ${backup_path}access_$(date -d "yesterday" +"%Y%m%d").log
/usr/local/nginx/sbin/nginx -s reload
rsync -avz /usr/local/nginx/logs/backup/* root@192.168.21.71:/server/nginx_log/access
rm -fr /usr/local/nginx/logs/backup/*





````
