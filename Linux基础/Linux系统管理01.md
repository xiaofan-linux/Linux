#### Linux 系统管理 01——系统命令精讲

###### 一、Linux 命令的分类
- 1. 内部命令:
  ```shell
    属于 Shell 解释器的一部分
  ```
- 2. 外部命令:
  ```shell
  独立于 Shell 解释器之外的程序
  ```
- 3. type 命令，查看命令是外部命令还是内部命令:
  ```shell
  [root@www ~]# type cd
  cd is a shell builtin 						//cd 是一个内部命令
  [root@www ~]# type ifconfig
  ifconfig is /sbin/ifconfig 				//ifconfig 是一个外部命令
  ```

###### 二、Linux 命令格式
- 1. Linux 命令的通用格式:
  ```shell
    命令字 [选项] [参数]
  ```
- 2. 选项:用于调节命令的具体功能
```shell
  “-”引导短格式选项，例如“ls -a”
  “--”引导长格式选项，例如“ls --help”
  注意:多个短格式选项可以合并，例如“ls -alh” 但是多个长格式选项，不能合并。
```
- 3、参数:命令的对象，如文件、目录名等
```shell
  例如:
  [root@www ~]# ls -alh /etc ls——命令字;-alh——选项;/etc——参数
```

###### 三、命令快捷键
  ```shell
  tab 键:自动补齐文件名，命令等;按两次 tab 键，系统将输出可用的所有名称列表。反斜杠“\”:强行换行
  ctrl+U:快速删除光标之前所有字符(可视为剪切)
  ctrl+K:快速删除光标之后所有字符(可视为剪切)
  ctrl+Y:黏贴刚才所删除(剪切)的字符
  ctrl+L:清屏
  ctrl+C:取消当前命令行编辑;结束当前执行的命令
  ctrl+D:从 shell 提示中注销关闭，类似输入 exit
  ctrl+A:把光标移动到行首，类似于 Home 键
  ctrl+E:把光标移动到行尾，类似于 End 键
  ctrl+Z:转入后台运行
  ctrl+R:在历史命令中查找(常用并且很好用)
  ```

###### 四、帮助命令
- 1、help 内部命令帮助，查看 bash 内部命令的帮助
```shell
  用法 1:help 内部命令字
  [root@www ~]# help cd
  用法 2:命令字 --help 即命令的“--help”选项，适用于大多数外部命令
  [root@www ~]# ls --help
```

- 2、man 用来提供在线帮助，使用权限是所有用户。在 Linux 系统中存储着一部联机使用的 手册，以供用户在终端上查找。使用 man 命令可以调阅其中的帮助信息，非常方便实用。
  - (1)用法:man 命令字
  ```shell
  man [-acdfhkKtwW] [-m system] [-p string] [-C config_file] [-Mpath] [-P pager] [-S section_list][section] name ...
  ```
  - (2)示例:
  ```shell
  -C config_file:指定设定文件 man.conf，缺省值是/etc/man.conf。
  ```
  - (4)代号 代表內容
  ```shell
  [root@www ~]# man 1 man
  [root@www ~]# man 7 man
  ```
  ```shell
  1 使用者在 shell 中可以操作的指令或可执行档
  2 系統核心可呼叫的函数与工具等
  3 一些常用的函数(function)与函数库(library)，大部分是 C 的函数库(libc)
  4 装置档案的说明，通常在/dev 下的档案
  5 设定档或者是某些档案的格式
  6 游戏(games)
  7 惯例与协定等，例如 Linux 档案系统、网络协定、ASCII code 等等的說明
  8 系統管理員可用的管理指令
  9 跟 kernel 有关的文件
  ```
  - (5)由于手册页 man page 是用 less 程序来看的(可以方便地使屏幕上翻和下翻), 所以 在 man page 里可以使用 less 的所有选项。

###### 五、ls(list)命令详解
- 1、作用:
  ```shell
  列表显示目录内的文件及目录，结合不同选项实现不同的作用。
  ```
- 2、格式:
  ```shell
  ls [选项] 目录或文件名
  ```
- 3、常用选项:
  ```shell
  -l:以长格式(long)显示文件和目录的列表
  -a:显示所有(all)子目录和文件的信息
  -A:与-a 基本类似，但有两个特殊隐藏目录“.”和“..”不显示
  -d:显示目录(directory)本身的属性，常与-l 同时使用
  -h:以更人性化(human)的方式显示出目录或文件的大小，常与-l 同时使用
  -R:以递归(recursive)的方式显示目录及其子目录中的所有内容
  ```
- 4、示例:
  ```shell
  [root@harbor ~]# ls /root/
  functions  README
  [root@harbor ~]# ls -R /root/
  /root/:
  functions  README
  ```

###### 六、du(disk usage)命令详解
- 1、作用:
  ```shell
  用于统计制定目录或文件所占用磁盘空间的大小
  ```
- 2、格式:
  ```shell
  du [选项] 目录或文件名
  ```
- 3、常见选项:
  ```shell
  -a:统计磁盘空间占用时所有的文件，而不仅仅是统计目录
  -s:只统计所占用空间总的(summary)大小
  -h:以更人性化(human)的方式显示出目录或文件的大小
  ```
- 4、示例:
  ```shell
  [root@www ~]# du -sh test/
  16K test/
  ```

###### 七、touch 命令
- 1、作用:
  ```shell
  创建空文件，用于测试。若当前文件已存在时，将更新该文件的时间戳
  ```
- 2、格式:
  ```shell
  touch 文件名
  ```
- 3、示例:
  ```shell
  [root@harbor ~]# touch test
  [root@harbor ~]# stat test
    文件："test"
    大小：0         	块：0          IO 块：4096   普通空文件
  设备：fd00h/64768d	Inode：101553684   硬链接：1
  权限：(0644/-rw-r--r--)  Uid：(    0/    root)   Gid：(    0/    root)
  最近访问：2019-05-09 22:29:00.251050723 +0800
  最近更改：2019-05-09 22:29:00.251050723 +0800
  最近改动：2019-05-09 22:29:00.251050723 +0800
  创建时间：-
  ```

###### 八、mkdir(make directory)命令
- 1、作用:
  ```shell
  创建新目录
  ```
- 2、格式:
  ```shell
  mkdir [选项] 目录位置及名称
  ```
- 3、常用选项:
  ```shell
  -p 一次性创建嵌套的多层目录
  -v 显示详细
  -m 跳出当前的 umask 值
  ```
- 4、示例:
  ```shell
  [root@harbor ~]# mkdir -pv aaa/bbb/ccc
  mkdir: 已创建目录 "aaa"
  mkdir: 已创建目录 "aaa/bbb"
  mkdir: 已创建目录 "aaa/bbb/ccc"
  ```

###### 九、cp(copy)命令
- 1、作用:
  ```shell
  复制文件或目录
  ```
- 2、格式:
  ```shell
  cp [选项] 源文件或目录 目标文件或目录
  ```
- 3、常用选项:
  ```shell
  -f 覆盖同名文件或目录，强制(force)复制
  -i 提醒用户确认(interactive，交互式)
  -p 保持(preserve)源文件权限、属性、属主及时间标记等不变
  -r 递归(recursive)复制
  ```
- 4、示例:
  ```shell
  [root@harbor ~]# ls
  aaa  functions  README  test
  [root@harbor ~]# cp -r test aaa/
  [root@harbor ~]# ls aaa/
  bbb  test
  ```

###### 十、rm(remove)命令
- 1、作用:
  ```shell
  删除制定的文件或目录
  ```
- 2、格式:
  ```shell
  rm [选项] 要删除的文件或目录
  ```
- 3、常用选项:
  ```shell
  -f 不提示，直接强制删除
  -i 提示用户确认
  -r 递归式删除整个目录树
  ```
- 4、示例:
  ```shell
  [root@www /]# rm -rf test 【此命令危险，建议进入到文件夹后删除】 建议如下操作:
  [root@www /]# cd test/
  [root@www test]# rm -rf *
  ```

###### 十一、mv(move)命令
- 1、作用:
  ```shell
  将指定文件或目录转移位置(剪切)，如果目标位置与源位置相同，则相当于执行 重命名操作
  ```
- 2、格式:
  ```shell
  mv [选项] 源文件或目录 目标文件或目录
  ```
- 3、示例:
  ```shell
  [root@harbor ~]# ls
  aaa  functions  README  test
  [root@harbor ~]# mv README aaa/
  [root@harbor ~]# ls aaa/
  bbb  README  test
  ```

###### 十二、which 命令
- 1、作用:
  ```shell
  查找 Linux 命令程序所在的位置
  ```
- 2、格式:
  ```shell
  which 命令|程序名
  ```
- 3、示例:
  ```shell
  [root@www ~]# which du
  /usr/bin/du
  注意:默认当只找到第一个目标后不再继续查找，若需查找全部，加选项-a。
  ```

###### 十三、find 命令
- 1、作用:
  ```shell
  精细查找文件或目录
  ```
- 2、格式:
  ```shell
  find [查找范围] [查找条件表达式]
  ```
- 3、常用查找条件:
  ```shell
  -name 按名称查找
  -size 按大小查找
  -user 按属性查找
  -type 按类型查找
  ```
  ```shell
  例:find /etc –name “resol*.conf”
  例:find /etc –size +1M 【k，M，G】
  例:find /etc –user root
  例:find /boot –type d 【d 目录;f 普通文件;b 块设备;c 字符设备文件】
  ```
- 4、逻辑运算符
  - (1)-a (and)逻辑“与”运算
    ```shell
    [root@www ~]# find /boot -size +1M -a -name "vm*"
    /boot/vmlinuz-2.6.32-431.el6.x86_64
    ```
  - (2)-o (or)逻辑“或”运算
    ```shell
    [root@www ~]# find /boot -size +1M -o -name "vm*"
    /boot/vmlinuz-2.6.32-431.el6.x86_64
    /boot/initramfs-2.6.32-431.el6.x86_64.img
    /boot/System.map-2.6.32-431.el6.x86_64
    ```
