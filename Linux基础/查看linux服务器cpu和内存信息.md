#查看linux服务器cpu和内存信息
```
查看CPU详细信息

1、查看CPU物理个数

grep 'physical id' /proc/cpuinfo | sort -u | wc -l

2、查看CPU核心数

grep 'core id' /proc/cpuinfo | sort -u | wc -l

3、查看CPU线程数

grep 'processor' /proc/cpuinfo | sort -u | wc -l

4、查看CPU型号

dmidecode -s processor-version

5、查看CPU详细信息

cat /proc/cpuinfo

查看内存详细信息

1、查看内存条数目

dmidecode | grep -P -A5 "Memory\s+Device" | grep Size | grep -v Range | grep -v "No Module Installed" | wc -l

2、查看每个内存条大小

dmidecode | grep -P -A5 "Memory\s+Device" | grep Size | grep -v Range | grep -v "No Module Installed"

3、查看每个内存条频率

dmidecode | grep -A16 "Memory Device" | grep 'Speed'

4、查看内存条厂商

dmidecode | grep -A16 "Memory Device" | grep Manufacturer

5、查看内存条类型DDR3、DDR4

dmidecode | grep -A16 "Memory Device" | grep "Part Number"

6、计算总的内存

dmidecode | grep -P -A5 "Memory\s+Device" | grep Size | grep -v Range | grep -v "No Module Installed" | awk 'BEGIN{sum=0}{sum+=$2};END{print sum $3}'

7、查看当前插槽数

dmidecode | grep -P -A5 "Memory Device" | grep Size | wc -l



附录：

查看服务器型号

dmidecode | grep 'Product Name'

查看服务器硬盘信息

cat /proc/scsi/scsi


```
