### MySQL授权管理

###### 授权命令格式
  ```shell
  GRANT ALL [PRIVILEGES] ON db.tbl TO 'username'@'host' IDENTIFIED BY 'password';
  # db:数据库名称,可以使用*通配
  # tbl: 表名称,可以使用*通配
  # username: 用户名称
  # host: 主机
  # password: 用户密码
  ```

###### 1.实例语句
```shell
# 给本地用户授权某个数据库的所有权限
# grant 权限 on 数据库.表名 to 用户名@"本地回环地址或localhost" identified by "密码"；
grant all privileges on xiaofan.* to xiaofan@'localhost' identified by '123';
grant all privileges on xiaofan.* to xiaofan@'127.0.0.1' identified by '123';
# 注意：上述两条命令都是对xiaofan用户开放xiaofan数据库的所有权限
#       但是上述两台命令的针对的xiaofan用户是不一样的
#       一个是xiaifan@localhost用户，一个是xiaofan@127.0.0.1,mysql会认为这是两个用户
```
```shell
# 授权是privileges关键字可省，示例如下：
grant all on xiaofan.* to xiaofan@'127.0.0.1' identified by '123';
```
```shell
# 给远程用户授权
# grant  权限 on 数据库名.* to 用户名@'远程主机地址或对应的主机名' identified by '密码';
grant all privileges on xiaofan.* to xiaofan@'%' identified by '123';
# 上述命令比较危险，表示xiaofan用户可以通过任意主机连接到xiaofan数据库,并且拥有xiaofan库的所有权限
# 我们可以稍微缩小一下xiaofan用户能够连接数据库的IP地址范围,比如，只允许xiaofan用户通过192.168网段的地址连接xiaofan数据库
grant all privileges on xiaofan.* to xiaofan@'192.168.%.%' identified by '123';
# 使用上述命令授权后，需要使用如下命令刷新：
flush privileges;
```
```shell
# 我们也可以授权用户某个数据库的某个权限
# 比如，只授权用户对于某个数据库的查询权限
grant select on xiaofan.* to xiafan@'192.168.%.%';
# 也可以授权用户某个的数据库的多个权限  比如，授权用户
grant insert,delete,update,select on xiaofan.* to xiaofan@'192.168.%.%';
# 也可以将同样的权限同时授权于多个用户，比如：
grant select on xiaofan.* to xiaofan@'localhost',xiaofan@'127.0.0.1';
# 对某张表的某个字段授权
grant select(name,host) on mysql.user to xiaofan@'localhost';
# 上面只授权xiaofan用户对mysql库user表的name字段和host字段的查询权限
# 虽然user还有其他字段，但对xiaofan用户无法查看其他字段，也不能查询其他字段
# 当xiaofan用户查询user表结构(desc)，只能看到name/user字段
# 如果xiaofan用户使用select * from mysql.user;这样的语句查询user表，也出现报错信息，提示有没有其他表的查询权限。
```
```shell
# 如果shanghai数据库中有一张表的名称为user,同时，shanghai数据库中有一个函数也叫user,
# mysql管理员只想把user函数授予xiaofan用户,而不是user表授予xiaofan用户，
# 我们可以通过function关键字指明被操作的对象函数，而不是表
grant execute on funtion shanghai.user to xiaofan@'192.168.%.%';
# 上面语句表示授权xiaofan用户对shanghai数据量user函数拥有执行权限、
# 同时，也可以使用procedure关键字，指明被操作的对象是存储过程
grant execute on procedure shanghai.user to xiaofan@'192.168.%.%';
# 上面语句表示授权xiaofan用户对shanghai库中的user存储过程拥有执行权限
```
```shell
# 用户被创建时，mysql会自动授予其usage权限，usage权限只能用于登陆数据库，不能执行其他操作
 # 如果用户有可能会跨越不安全的网络连接到数据库，我们可以强制用户使用ssl建立会话
 grant usage on *.* to 'xiaofan'@'192.178.%.%' require ssl;
 # 上述示例表示'xiaofan'@'192.168.%.%'用户连接当前mysql中的所有数据库时都必须使用ssl建立会话
 # 如果想要取消上述的ssl连接限制，可以使用如下命令，撤销强制使用ssl建立会话的限制
 grant usage on *.* to 'xiaofan'@'192.168.%.%' require none;
 ```
 ```shell
 # 假设，root用户授权了xiaofan用户某些权限，那么xiaofan用户是否将已经拥有的权限授权给别的用户？
 # xiaofan用户能否将已有权限授权于其他用户取决于xiaofan用户是否拥有grant选项
 # 如果在授权xiaofan用户时，搭配了grant选项，则xiaofan用户有权将已拥有的权限授予其他用户，但是这样做比较危险
 # 一般情况下应由管理员统一授权，但是此处用于演示
 grant select on dome.* xiaofan@'192.168.%.%' with grant option;
 # 上述命令表示xiaofan用户被授予了dome数据库的select的权限，同时xiaofan用户也能将此权限授予其他用户，
 # 而且xiaofan用户也能授予其他用户select权限时使用with grant option，所以这很危险，请勿随意使用此选项。
 # 除了上面提到的grant option ，管理员还可以通过如下选项对用户进行一些其他的限制
 MAX_QUERIES_PER_HOUR:限制用户每小时执行的查询语句数量
 MAX_UPDATES_PER_HOUR:限制用户每小时执行的更新语句数量
 MAX_CONNECTIONS_PRR_HOUR:限制用户每小时连接数据量的次数
 MAX_USER_CONNECTIONS:限制用户使用当前账号同时连接服务器的连接数量
 # 上述各限制选项的示例如下:
 grant select on *.* to xiaofan@'192.168.%.%' identified by '123' with MAX_QUERIES_PER_HOUR 20;
 GRANT select ON *.* TO xiaofan@'192.168.$.$' identified by '123' with MAX_UPDATES_PER_HOUR 10;
 GRANT select ON *.* TO xiaofan@'192.168.%.%' identified by '123' with MAX_CONNECTIONS_PRR_HOUR 5;
 GRANT select ON *.* TO xiaofan@'192.168.%.%' identified by '123' with MAX_USER_CONNECTIONS 5;
 # 如果将上述限制对应的数字修改为0, 则表示不限制
 ```
##### 查看授权
```shell
# 查看授权可以从两个方面查看，从用户的角度查看权限，或者从数据库的角度查看权限
# 从用户的角度查看权限表示查看对应的都能操作哪些数据库
# 从数据库的角度查看权限表示查看指定数据库都对哪些用户开放了哪些权限
# 从数据库用户的角度查看授权的语句如下:
shou grants for 用户名;
# 比如我们需要查看xiaofan@localhost这个用户对哪些库有哪些权限，可以使用如下语句:
show grants for xiaofan@localhost;
# 从数据库的角度查看权限,可以使用如下语句:
select 8 from mysql.db where Db="你要查看的数据库";
# 示例如下:
select * from mysql.db where Db="xiaofan"\G;
```
###### 删除权限
```shell
# 删除授权/撤销授权的常用语句如下:
revoke "要移除的权限" on 数据库.表 from 用户@host;
# 比如删除xiaofan@localhost用户对于word数据库的所有操作权限，语句如下
revoke all on word.* from xiaofan@'localhost';
# 也可以删除指定的权限，示例如下:
revoke select update on word.* from xiaofan@'localhost';
```
