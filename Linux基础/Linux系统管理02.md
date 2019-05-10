#### Linux 系统管理 02——目录和文件管理

###### 一、 Linux 目录结构
- 1、 树形目录结构
  ```shell
  [root@harbor ~]# ls /
  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
  ```
- 2、根目录
  ```shell
  (1)所有分区、目录、文件等的位置起点
  (2)整个树形目录的结构中，使用独立的一个“/”表示
  ```

- 3、常见子目录的作用

    根目录|作用
    -|-
    /root|系统管理员root的宿主目录
    /home|普通用户的宿主目录
    /boot|系统内核启动文件
    /dev|Device,设备文件
    /etc|配置文件
    /bin|Binary(二进制)，所有用户可执行的命令
    /sbin|System Binary,管理员可执行的命令
    /usr|Unix Software resource ,应用程序
    /var|Variabilty(可变的)，日志文件等
###### 二、 查看文件内容——cat、tac、nl、more、less、head、tail
- ###### 1. cat、tac、nl
- (1)cat 查看文件内容，可同时显示多个文件的内容
  ```shell
  tac 查看文件内容，可同时显示多个文件的内容（反向）
  nl 将指定的文件添加行号标注后写到标准输出
  ```
- (2)格式:
  ```shell
  cat [选项] 文件名
  ```

- (3)常用选项:
  ```shell
  -n 显示内容的同时显示行号
  -A 显示所有的字符 (通常 windows 系统拷贝过来的文件无法直接 cat 到，需 要加此选项)
  ```
- (4)示例:
  ```shell
  [root@harbor ~]# cat tmp.txt
  bin
  boot
  dev
  etc
  home

  [root@harbor ~]# cat -nA tmp.txt
       1	bin$
       2	boot$
       3	dev$
       4	etc$
       5	home$

  [root@harbor ~]# tac tmp.txt
  var
  usr
  tmp
  sys
  sbin
  [root@harbor ~]# nl tmp.txt
       1	bin
       2	boot
       3	dev
       4	etc
       5	home
  ```
- ###### 2、more、less
- (1)more 全屏方式分页显示文件内容
  - 1>格式:
    ```shell
    more [选项] 文件名 (一般不用选项)
    ```
  - 2>快捷键:
    ```shell
    按 Enter 向下滚动一行
    按空格键向下滚动一页 ·按q键退出
    ```
  - 3>示例:
    ```shell
    [root@www ~]# more /etc/yum.repos.d/CentOS-Base.repo
    # CentOS-Base.repo
    #
    # The mirror system uses the connecting IP address of the client and the
    # update status of each mirror to pick mirrors that are updated to and
    # geographically close to the client.  You should use this for CentOS updates
    # unless you are manually picking other mirrors.
    #
    # If the mirrorlist= does not work for you, as a fall back you can try the
    # remarked out baseurl= line instead.
    #
    #

    [base]
    name=CentOS-$releasever - Base
    mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
    #baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
    gpgcheck=1
    gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    ```
- (2)less 与 more 基本相同，但扩展功能更多
    - 1>格式:
      ```shell
      less [选项] 文件名 (一般不用选项)
      ```
    - 2>快捷键
      ```shell
      按 PgUp、PgDn 键上下翻页
      按“/”键查找内容，“n”下一个，“N”上一个
      其他功能与 more 基本类似
      ```
    - 3>示例:
      ```shell
      [root@www ~]# less /etc/passwd
      systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
      dbus:x:81:81:System message bus:/:/sbin/nologin
      polkitd:x:999:998:User for polkitd:/:/sbin/nologin
      sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
      postfix:x:89:89::/var/spool/postfix:/sbin/nologin
      chrony:x:998:996::/var/lib/chrony:/sbin/nologin
      ntp:x:38:38::/etc/ntp:/sbin/nologin
      icinga:x:997:995:icinga:/var/spool/icinga2:/sbin/nologin
      nagios:x:996:993::/var/spool/nagios:/sbin/nologin
      rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
      mysql:x:27:27:MariaDB Server:/var/lib/mysql:/sbin/nologin
      apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
      named:x:25:25:Named:/var/named:/bin/false
      /etc/passwd (END)
      ```
- ###### 3、head、tail
- (1)head
  ```shell
  查看文件开头的一部分内容，默认显示 10 行，可加选项调节
  ```
  示例:
    ```shell
    [root@www ~]# head -5 /etc/passwd 【显示 passwd 文件的前 5 行内容】
    [root@harbor ~]#  head -5 /etc/passwd
    root:x:0:0:root:/root:/bin/bash
    bin:x:1:1:bin:/bin:/sbin/nologin
    daemon:x:2:2:daemon:/sbin:/sbin/nologin
    adm:x:3:4:adm:/var/adm:/sbin/nologin
    lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
    ```

- (2)tail
  ```shell
  查看文件结尾的一部分内容，默认显示 10 行，可加选项调节
  ```
  - 1>示例:
    ```shell
    [root@www ~]# tail -5 /etc/passwd 【显示 passwd 文件的前 5 行内容】
    [root@harbor ~]# tail -5 /etc/passwd
    nagios:x:996:993::/var/spool/nagios:/sbin/nologin
    rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
    mysql:x:27:27:MariaDB Server:/var/lib/mysql:/sbin/nologin
    apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
    named:x:25:25:Named:/var/named:/bin/false
    ```
  - 2>tail –f 动态更新尾部的信息，ctrl +C 终止

###### 三、统计文件内容——wc
- 1、作用:
  ```shell
  统计文件中的单词数量(Word Count)等
  ```
- 2、格式:
  ```shell
  wc [选项] …目标文件
  ```
- 3、常用选项:
  ```shell
  -l:统计行数
  -w:统计单词个数
  -c:统计字节数
  ```
- 4、示例:
  ```shell
  [root@www ~]# wc /etc/hosts
  2 10 158 /etc/hosts 【行数、单词数、字节数】
  [root@www ~]# wc -l /etc/hosts
  2 /etc/hosts
  [root@www ~]# wc -w /etc/hosts
  10 /etc/hosts
  [root@www ~]# wc -c /etc/hosts
  158 /etc/hosts
  ```
###### 四、检索和过滤——grep、egrep
- 1、grep
  - (1)作用:
    ```shel
    在文件中查找并显示包含指定字符串的行
    ```
  - (2)格式:
    ```shell
    grep [选项]… 查找条件 目标文件
    ```
  - (3)常用选项:
    ```shell
    -i:查找时忽略大小写
    -v:反转查找，输出与条件不相符的行
    ```
  - (4)“^…”、“…$”与“^$”
    ```shel
    1>“^...”表示以...开头
    2>“...$”表示以...结尾
    3>“^$”表示空行
    ```
  - (5)示例:
    ```shel
    [root@www ~]# grep "ftp" /etc/passwd
    ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
    【过滤掉以“#”开头的注释行以及空行】
    [root@www ~]# grep -v "^#" /etc/yum.conf |grep -v "^$"
    ```
- 2、egrep
  - (1)作用:
    ```shell
    增强型过滤
    ```
  - (2)格式:
    ```shell
    egrep [选项] “查找条件 1|查找条件 2|查找条件 3...” 目标文件
    ```
  - (3)常用选项与 grep 相同
  - (4)示例:
    ```shell
    [root@www ~]# egrep -v "^#|^$" /etc/yum.conf 【与 grep 示例中的作用相同】
    ```
###### 五、压缩和解压缩——gzip、gunzip、bzip2、bunzip2
- 1、gzip、gunzip
  - (1)gzip
  - 1>作用:
  ```shell
  压缩，选项为 1-9 的数字控制压缩级别，数字越大压缩级别越高。压缩后 文件格式为“.gz”
  ```

  - 2>格式:
    ```shell
    gzip [-9] 文件名
    ```
  - 3>示例:
    ```shell
    [root@harbor ~]# ls
    tmp.txt
    [root@harbor ~]# gzip -9 tmp.txt
    [root@harbor ~]# ls
    tmp.txt.gz
    [root@harbor ~]#
    ```
  - (2)gunzip、gzip –d
    - 1>作用:
      ```shell
      解压缩格式为.gz 的压缩文件
      ```
    - 2>格式:gunzip 文件名
      ```shell
      gzip -d 文件名
      ```
    - 3>示例:
      ```shell
      [root@harbor ~]# ls
      123.txt  tmp.txt.gz
      [root@harbor ~]# gunzip tmp.txt.gz
      [root@harbor ~]# ls
      123.txt  tmp.txt
      [root@harbor ~]# ls
      123.txt  tmp.txt.gz
      [root@harbor ~]# gzip -d tmp.txt.gz
      [root@harbor ~]# ls
      123.txt  tmp.txt
      ```
- 2、bizp2、bunzip2
  - (1)bzip2
    - 1>作用:
      ```shell
      压缩，选项为 1-9 的数字控制压缩级别，数字越大压缩级别越高。压缩后 文件格式为“.bz2”
      ```
    - 2>格式:
      ```shell
      bzip2 [-9] 文件名
      ```
    - 3>示例:
      ```shell
      [root@harbor ~]# ls
      123.txt
      [root@harbor ~]# bzip2 -9 123.txt
      [root@harbor ~]# ls
      123.txt.bz2
      ```
  - (2)bunzip2、bzip2 –d
    - 1>作用:
        ```shell
        解压缩格式为.bz2 的压缩文件
        ```
    - 2>格式:
        ```shell
        bunzip2 文件名
        bzip2 -d 文件名
        ```
    - 3>示例:
      ```shell
      [root@harbor ~]# ls
      123.txt
      [root@harbor ~]# bzip2 -9 123.txt
      [root@harbor ~]# ls
      123.txt.bz2
      [root@harbor ~]# gzip -d 123.txt.bz2
      [root@harbor ~]# ls
      123.txt
      ```

###### 六、归档命令——tar
- 1、作用:
  ```shell
  制作归档文件、释放归档文件
  ```
- 2、格式:
  ```shell
  1>归档:tar [选项 c…] 归档文件名 源文件或目录
  2>释放:tar [选项 x…] 归档文件名 [-C 目标目录]
  ```
- 3、常用选项:
  ```shell
  -c 创建.tar 格式的包文件
  -x 解开.tar 格式的包文件
  -v 输出详细信息
  -f 表示使用归档文件(后面需紧跟归档文件名)
  -p 打包时保留原始文件及目录的权限(不建议使用)
  -t 列表查看包内的文件
  -C 解包时指定释放的目标目录
  -z 调用 gzip 程序进行压缩或解压
  -j 调用 bzip2 程序进行压缩或解压
  -P 打包时保留文件及目录的绝对路径(不建议使用)
  注意:tar 命令的选项前可以省略“-”，在解压时无需选择“-z”或“-j”，命令可以自行识别
  ```

- 4、常用命令组合：
  ```shell
  tar -zcvf 归档文件名 源文件或目录
  tar -zxvf 归档文件名 [-C 目标目录]
  tar -jcvf 归档文件名 源文件或目录
  tar -jxvf 归档文件名 [-C 目标目录]
  ```
