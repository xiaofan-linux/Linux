

```
#!/bin/bash
#
# 一键安装lvs-nat脚本，需要注意的是主机名成和ip的变化稍作修改就可以了

HOSTNAME=`hostname`
Director='LVS'
VIP="192.168.0.200"
RIP1="172.16.100.10"
RIP2="172.16.100.11"
RealServer1="web1"
RealServer2="web2"
Httpd_config="/etc/httpd/conf/httpd.conf"


#Director Server Install configure ipvsadm
if [ "$HOSTNAME" = "$Director" ];then
ipvsadm -C
yum -y remove ipvsadm
yum -y install ipvsadm
/sbin/ipvsadm -A -t $VIP:80 -s rr
/sbin/ipvsadm -a -t $VIP:80 -r $RIP1 -m
/sbin/ipvsadm -a -t $VIP:80 -r $RIP2 -m

echo 1 > /proc/sys/net/ipv4/ip_forward

echo "========================================================"
echo "Install  "
echo "========================================================"
fi




#RealServer Install htpd

if [ "$HOSTNAME" = "$RealServer1" ];then
yum -y remove httpd
rm -rf /var/www/html/index.html
yum -y install httpd
echo "web1 Allentuns.com" > /var/www/html/index.html
sed -i '/#ServerName www.example.com:80/a\ServerName localhost:80' $Httpd_config
service httpd start


echo "========================================================"
echo "Install $RealServer1 "
echo "========================================================"
fi

if [ "$HOSTNAME" = "$RealServer2" ];then
yum -y remove httpd
rm -rf /var/www/html/index.html
yum -y install httpd
echo "web2 Allentuns.com" > /var/www/html/index.html
sed -i '/#ServerName www.example.com:80/a\ServerName localhost:80' $Httpd_config
service httpd start
echo "Install $RealServer2"


echo "========================================================="
echo "Install $RealServer2"
echo "========================================================="
fi

```

####简洁版
```
#!/bin/bash
VIP=11.0.0.100
RIP1=11.0.0.20
RIP2=11.1.0.30
case "$1" in
start)
	echo "正在生成策略..."
	yum -y install ipvsadm >> /dev/null
	echo 1 > /proc/sys/net/ipv4/ip_forward
	iptables -t nat -F
	iptables -t nat -X
	iptables -t nat -A POSTROUTING -s 11.0.0.1/24 -o eth0 -j SNAT --to-source $VIP
	/etc/init.d/iptables save >>/dev/null

	ipvsadm -C
	/sbin/ipvsadm -A -t $VIP:80 -s rr
	/sbin/ipvsadm -a -t $VIP:80 -r $RIP1 -m
	/sbin/ipvsadm -a -t $VIP:80 -r $RIP2 -m
	/etc/init.d/ipvsadm save >>/dev/null
echo "========================================================"
	/sbin/ipvsadm -Ln --stats
echo "========================================================"
	iptables -t nat -L
echo "========================================================"
echo "               Install   is OK !!!                      "
echo "========================================================"
;;
stop)
	echo "正在清除..."
	iptables -t nat -F
	iptables -t nat -X
	/etc/init.d/iptables save >>/dev/null
	/sbin/ipvsadm -C
	/etc/init.d/ipvsadm save >>/dev/null
	echo "========================================================"
        /sbin/ipvsadm -Ln --stats
	echo "========================================================"
	iptables -t nat -L
	echo "========================================================"
	echo "               Install   is OK !!!                      "
	echo "========================================================"

;;
*)
echo "Usage0{start|stop}"
exit 1
esac







```
