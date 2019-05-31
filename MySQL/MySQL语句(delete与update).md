### MySQL语句(delete与update)

###### delete常用语句
> 删除数据的语句比较简单，主要是通过where子句给定删除的范围，而where子句的示例可以参考select常用语句，但是删除前请确定给定的条件没有任何问题，在不确定的情况下不要随意删除数据。

```shell
# 如下语句表示删除tb1中的所有数据，也就是清空tb1表，非常危险，切勿随意使用。
delete from tb1;
```
```shell
# 根据给定的条件删除数据，在不确定的情况下或者没有备份的情况下，请勿随意删除数据
delete from tb1 where age=22;
delete from tb1 where name rlike '^t.*';
```
```shell
# 如下语句表示，从tb1表中找出age>30的数据行，然后将这些行按照age进行降序排列，排列后删除第一个。
delete from tb1 where age > 30 order by age desc limit 1;
```


###### update常用语句
> 修改数据的语句也比较简单，主要是通过where子句给定修改的范围，而where子句的示例可以参考select常用语句，执行更新语句之前请确定给定的条件是正确的，因为不加任何条件的更新语句表示更新表中的所有字段，如果你不确定要这么做，这样是非常危险的，所以执行update语句之前，也要再三确定条件给定正确。

```shell
# 如下语句表示更新tb1表中所有行的age字段的值为28，这种语句比较危险，除非你确定这样做，否则切勿执行
update tb1 set age = 28;
```
```shell
# 如下语句表示将tb1表中id号为13的行中的name字段的值改为luffy.
update tb1 set name='luffy' where id=13;
```
```shell
# 如下语句同上，只是一次修改了多个字段的值
update tb1 set name='luffy',age=25 where id=13;
```
