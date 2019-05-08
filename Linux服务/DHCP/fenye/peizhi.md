# DHCP配置文件详解
####1.DHCP服务器相关配置文件
        端口号：
            ipv4：udp67  udp68
            ipv6：udp546	udp547
        服务名称：dhcpd
        注意：修改配置文件前进行备份！！！
        主配置文件：/etc/dhcp/dhcp.conf
        模板文件：/usr/share/doc/dhcp*/dhcpd.conf.sample
####2.配置文件详解：
    option domain-name 			#设置所在DNS域
    option domain-name-servers	 #设置DNS服务器地址
    default-lease-time			 #设置默认租约时间，单位是秒
    max-lease-time			     #设置最大租约时间，单位是秒
    log-facility local7			#指日志设备
    ddns-update-style 			 #设定DNS的更新方式
    ddns-update-style 			 #标识权威服务器（多台DHCP服务器也只有权威服务器生效）

    Subnet 网段声明，作用于整个子网段
        range						#设置用于分配的IP地址池
        option subnet-mask		   #设置客户机的子网掩码
        option routers参数		   #设置客户机的默认网关地址
        option broadcast-address	 #设置客户机广播地址
    host主机声明，作用于某台主机
        hardware ethernet			#设置目标主机MAC地址
        fixed-address				#设置为其分配的保留IP地址
        filename “vmunix.passacaglia”	#启动文件名称，用于无盘工作站