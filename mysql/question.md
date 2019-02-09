# 问题

## 插入存在即更新

```sql
INSERT INTO `students` (`name`, `age`) VALUES ('ruesin', '20') ON DUPLICATE KEY UPDATE `has` = '1';
```

## 批量插入
```sql
INSERT INTO `students` (`name`, `age`)
VALUES
('xiaoming', '21'),
('ermao', '22');
	 
INSERT INTO `students`
VALUES
('huniu', '21'),
('goudan', '22');

INSERT INTO `students` (`name`, `age`) VALUES ('xiaohu','22'),('xiaoming','21') ON DUPLICATE KEY UPDATE `age` = `age`;

```

## 批量更新
```sql
replace into students (`name`, `age`) values ('xiaohu','22'),('xiaoming','21');

insert into test_tbl (`name`, `age`) values ('xiaohu','22'),('xiaoming','21') on duplicate key update `age`=values(`age`);
```

## int、varchar长度

int类型数据的字节大小是固定的4个字节；

但是int(5)和int(11)区别在于，显示的数据位数一个是5位一个是11位，在开启zerofill(填充零）情况下,若int(5)存储的数字长度是小于5的则会在不足位数的前面补充0,但是如果int(5)中存储的数字长度大于5位的话，则按照实际存储的显示(数据大小在int类型的4个字节范围内即可）,也就是说int(M)的M不代表数据的长度；

varchar(20)中的20表示的是varchar数据的数据长度最大是20，超过则数据库不会存储；

总结: int(M) M表示的不是数据的最大长度，只是数据宽度，并不影响存储多少位长度的数据；
     varchar(M) M表示的是varchar类型数据在数据库中存储的最大长度，超过则不存；

## 统计大于N的次数
where的条件是字段；而having 的条件可以是字段，也可以是聚集函数；

重要的是，where是筛选源数据，having多与group by 一起使用，并且条件常常是聚集函数；当有group by 时，having在group  by 条件的后面，而where 在group by的前面。

聚集函数：sum，count，avg ...等等；
```sql 
-- 查询每个年级总分数大于200的用哪些；
select  grade ,sum(score) from tables group by grade having sum(score) >200; 

-- 及格两门以上的学生
select count(`name`) as cnt from tables where score > 60 group by `grade` having cnt > 2;
```


