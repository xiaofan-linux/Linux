#Centos7系统初始化脚本
```
#!/bin/bash
#关闭selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
sed -i 's/#UseDNS yes/UseDNS no/g' /etc/ssh/sshd_config
setenforce 0
#安装wget
yum -y install wget
#更换yum源
repo=`ls /etc/yum.repos.d/repo.bak | wc -l`
if [ $repo -eq 0 ];
then
mkdir /etc/yum.repos.d/repo.bak
mv /etc/yum.repos.d/CentOS* /etc/yum.repos.d/repo.bak
cd /etc/yum.repos.d/
wget http://blog.whsir.com/uploads/CentOS7-Base-163.repo
yum clean all
yum makecache
else
echo "yum OK"
fi
#更新时间和系统
cd
yum -y install vim screen ntp
/usr/sbin/ntpdate ntp1.aliyun.com
yum -y update
#更改端口
sed -i 's/#Port 22/Port 2188/' /etc/ssh/sshd_config
#设置防火墙ifconfig
systemctl stop firewalld.service
systemctl disable firewalld.service
yum -y install iptables-services net-tools
systemctl enable iptables.service
sed -i 's/22/2188/' /etc/sysconfig/iptables
systemctl restart iptables.service
systemctl restart sshd
yum -y install epel-release
#更改ulimit
limit=`cat /etc/security/limits.conf | grep 65535 | wc -l`
if [ $limit -eq 0 ];
then
echo "ulimit -SHn 65535" >> /etc/rc.local
cat >> /etc/security/limits.conf << EOF
* soft nofile 65535
* hard nofile 65535
EOF
else
echo "ulimit ok"
fi
#开机启动优化
for i in acpid gpm lvm2-monitor anacron haldaemon mcstrans oddjobd setroubleshoot atd halt mdmonitor pand single auditd hidd mdmpd pcscd smartd hplip messagebus avahi-daemon ip6tables microcode_ctl snmptrapd avahi-dnsconfd ipmi multipathd psacct bluetooth rawdevices svnserve conman irda netconsole rdisc cpuspeed irqbalance netfs readahead_early tcsd iscsi netplugd readahead_later winbind cups iscsid restorecond wpa_supplicant cups-config-daemon kdump NetworkManager rpcgssd dnsmasq killall nfs rpcidmapd ypbind dund krb524 nfslock rpcsvcgssd yum-updatesd firstboot lm_sensors nscd saslauthd
do
chkconfig $i off > /dev/null 2>&1
done
date

```
