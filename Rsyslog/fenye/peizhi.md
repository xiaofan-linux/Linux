### Rsyslog配置文件详解

```
#### 加载模块 ####
$ModLoad imuxsock   # provides support for local system logging (e.g. via logger command)
$ModLoad imklog     # provides kernel logging support (previously done by rklogd)

#### 定义日志格式默认模板 ####
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

#### 包含其他配置文件 ####
$IncludeConfig /etc/rsyslog.d/*.conf

#### 规则 ####
# 关于内核的所有日志都放到/dev/console(控制台)
#kern.*                                                 /dev/console

# 记录所有日志类型的info级别以及大于info级别的信息到/var/log/messages，但是mail邮件信息，authpriv验证方面的信息和cron时间任务相关的信息除外
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# authpriv验证相关的所有信息存放在/var/log/secure
authpriv.*                                              /var/log/secure

# 邮件的所有信息存放在/var/log/maillog; 这里有一个-符号, 表示是使用异步的方式记录, 因为日志一般会比较大
mail.*                                                  -/var/log/maillog

# 计划任务有关的信息存放在/var/log/cron
cron.*                                                  /var/log/cron

# 记录所有的大于等于emerg级别信息, 以wall方式发送给每个登录到系统的人
*.emerg                                                 *

# 记录uucp,news.crit等存放在/var/log/spooler
uucp,news.crit                                          /var/log/spooler

# 记录启动的相关信息
local7.*                                                /var/log/boot.log

###日志转发规则###
#$WorkDirectory /var/spppl/rsyslog # where to place spool files
#$ActionQueueFileName fwdRule1   #unique name prefix for spool files
#$ActionQueueMaxDiskSpace 1g # 1gb space limit (use as much as possible)
#$ActionQueueSaveOnShutdown on   #save messages to disk on shutdown
#$ActionQueueType LinkedList     #run asynchronously
#$ActionResumeRetryCount -1      #infinite retries if host is down
# remote host is: name/ip:port, e.g. 192.168.0.1:514, port optional
#*.*    @@remote-host:514        #@@表示通过tcp协议发送 @表示通过udp进行转发
#local3.info @@localhost :514
#local7.*                        #@@192.168.56.7:514



```
