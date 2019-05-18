### Linux开机启动流程


- 1.开机bios自检
- 2.MBR引导
- 3.加载GRUB菜单
- 4.加载内核（kernel）
- 5.运行INIT进程
- 6.读取 /etc/inittab配置文件
- 7.执行 /etc/rc.d/sysinit脚本
- 8.执行 /etc/rc.d/rc脚本
- 9.执行 /etc/rc.d/rc.local
- 10.启动 mingetty 进程 （加载命令行）

---
- rc.local：开机自动服务的一个存放位置
- init.d：存放系统所有管理进程的位置
- rc.x：里面存放了一些的脚本，脚本名字以K开头表示stop。脚本名字以S开头表示start。名字中的数字表示执行顺序，数字越
         小越优先执行
- init：最主要的功能就只准备软件运行的环境，包括系统的主机名称、网络配置、语系处理、文件系统格式及其他服务的启动等，而所有的动作都根据在/etc/inittab中的配置
