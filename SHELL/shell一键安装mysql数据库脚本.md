#shell一键安装mysql数据库脚本
```shell
# 基于mysql5.5.34
#!/bin/bash
# Version: NO.1
WORK_HOME="/usr/local/"
mysqlDir="/usr/local/mysql/"
mysqlPass="your_password"

[ $(id -u) != "0" ] && echo "Error: You must be root to run this script" && exit 1
if ! which whiptail &>/dev/null; then yum install newt -y;fi
if ! which wget &>/dev/null; then yum install wget -y;fi


install_mysql() {
#       安装CMAKE
        yum groupinstall "Development tools" -y
        yum -y install  cmake openssl-devel ncurses-devel
        #添加mysql系统用户和系统组
        groupadd -r mysql
        useradd -g mysql -r mysql
        #下载 解压 编译 安装Mysql
#       cd && [ ! -f mysql-5.5.34.tar.gz ] && downFile "$1" "mysql-5.5.34.tar.gz" "Download Mysql 5.5.34"
#        cd && [ ! -f mysql-5.5.34.tar.gz ] && curl -O "http://192.168.1.96:8980/mysql-5.5.34.tar.gz"
        cd /root
        [ -f mysql-5.5.34 ] && rm -rf mysql-5.5.34
        tar xf mysql-5.5.34.tar.gz -C $WORK_HOME && cd $WORK_HOME/mysql-5.5.34
        cmake -DCMAKE_INSTALL_PREFIX=$mysqlDir \
-DMYSQL_DATADIR=/mydata/data/ \
-DWITH_SSL=system \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_SPHINX_STORAGE_ENGINE=1 \
-DWITH_ARIA_STORAGE_ENGINE=1 \
-DWITH_XTRADB_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_FEDERATEDX_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_EMBEDDED_SERVER=1 \
-DWITH_READLINE=1 \
-DENABLED_LOCAL_INFILE=1 \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DWITH_EXTRA_CHARSETS=all \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
        make -j $(awk '{if($1=="processor"){i++}}END{print i}' /proc/cpuinfo) && make install
        [ $? != 0 ] && exit 1

        cp ${mysqlDir}support-files/mysql.server /etc/rc.d/init.d/mysqld
        chmod +x /etc/rc.d/init.d/mysqld
        #配置配置文件
        \cp ${mysqlDir}support-files/my-large.cnf /etc/my.cnf
        sed -i "/query_cache_size/a datadir = /mydata/data/" /etc/my.cnf

        #初始化Mysql
        chmod -R 755 /usr/local/mysql/
        mkdir -pv /mydata/data
        mkdir /backup
        chown -R mysql:mysql /backup
        chown -R mysql:mysql /mydata/data/
        cd ${mysqlDir}
        ${mysqlDir}scripts/mysql_install_db --user=mysql --datadir=/mydata/data --basedir=${mysqlDir}
        echo "export PATH=${mysqlDir}bin:\$PATH" > /etc/profile.d/mysql5534.sh
        source /etc/profile.d/mysql5534.sh
        echo "/usr/local/mysql/lib" >/etc/ld.so.conf.d/mysqld.conf
        ldconfig -v | grep mysql
        ln -sv /usr/local/mysql/include/ /usr/include/mysqld

        chkconfig --add mysqld
        chkconfig mysqld on
        #启动Mysql
        service mysqld start
        #检测Mysql启动正常否，不正常就退出脚本
        ss -tnl | grep ':3306' &>/dev/null && [ $? != 0 ] && exit 1
        mysql <<< "use mysql;
update user set password=PASSWORD('$mysqlPass') WHERE USER='root';
delete from user where user='';
delete from user where password='';
SELECT USER,PASSWORD,HOST FROM user;
FLUSH PRIVILEGES;"
        mysql -uroot -p$mysqlPass <<< status && [ $? != 0 ] && exit 1

        #添加zabbix的数据库和用户
        mysql -uroot -p$mysqlPass <<< "USE mysql;
FLUSH PRIVILEGES;"
}
install_mysql






```
