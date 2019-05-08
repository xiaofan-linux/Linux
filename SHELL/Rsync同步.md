#rsync同步
```
#!/bin/bash
path=/backup
dir="`cat /etc/sysconfig/network-scripts/ifcfg-eth0 | grep IPADDR | cut -d '=' -f 2`_$(date +%F-%T)"
mkdir $path/$dir -p &&\
/bin/cp -pr /var/www/html/  $path/web-$dir &&\
/bin/cp /etc/rc.local $path/$dir/rc.local-$(date +%F-%T) &&\
rsync -avz $path/ rsycn://rsync_backup@192.168.21.71/web2
rm -fr $path/*
```
