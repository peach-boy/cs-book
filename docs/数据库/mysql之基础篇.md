# mysql之基础篇

## 一、数据类型

### 数值类型
|类型|大小|范围(有符号)|范围(无符号)|用途|
|-|-|-|-|-|
|TINYINT	|1 byte|	(-128，127)|	(0，255)|	小整数值|
|SMALLINT	|2 bytes|   (-32 768，32 767)|	(0，65 535)|大整数值|
|MEDIUMINT	|3 bytes|   (-8 388 608，8 388 607)|	(0，16 777 215)	大整数值|
|INT或INTEGER	|4 bytes|	(-2 147 483 648，2 147 483 647)	|(0，4 294 967 295)|	大整数值|
|BIGINT	|8 bytes	|(-9,223,372,036,854,775,808，9 223 372 036 854 775 807)|(0，18 446 744 073 709 551 615)	|极大整数值|
|FLOAT|	4 bytes|	(-3.402 823 466 E+38，-1.175 494 351 E-38)，0，(1.175 494 351 E-38，3.402 823 |466 351 E+38)	0，(1.175 494 351 E-38，3.402 823 466 E+38)|	单精度浮点数值
|DOUBLE	|8 bytes	|(-1.797 693 134 862 315 7 E+308，-2.225 073 858 507 201 4 E-308)，0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)|	0，(2.225 073 858 507 201 4 E-308，1.797 693 134 862 315 7 E+308)	|双精度 浮点数值|
|DECIMAL|	对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2|	依赖于M和D的值|	依赖于M和D的值	|小数值|


### 日期和时间类型
|类型|大小|范围(有符号)|格式|用途|
|-|-|-|-|-|
|DATE	|3|	1000-01-01/9999-12-31|	YYYY-MM-DD|	日期值|
|TIME	|3	|'-838:59:59'/'838:59:59'|	HH:MM:SS|	时间值或持续时间|
|YEAR	|1|	1901/2155|	YYYY|	年份值|
|DATETIME	|8|	1000-01-01 00:00:00/9999-12-31 23:59:59	|YYYY-MM-DD HH:MM:SS|混合日期和时值|
|TIMESTAMP	|4	|1970-01-01 00:00:00/2038 结束时间是第 2147483647 秒，北京时间 2038-1-19 11:14:07，格林尼治时间 2038年1月19日 凌晨 03:14:07 |YYYYMMDD HHMMSS|	混合日期和时间值，时间戳|



### 字符串类型
|类型|大小|用途|
|-|-|-|
|CHAR	|0-255 bytes|	定长字符串|
|VARCHAR	|0-65535 bytes|	变长字符串|
|TINYBLOB	|0-255 bytes|	不超过 255 个字符的二进制字符串|
|TINYTEXT	|0-255 bytes|	短文本字符串|
|BLOB	|0-65 535 bytes|	二进制形式的长文本数据|
|TEXT	|0-65 535 bytes|	长文本数据|
|MEDIUMBLOB	|0-16 777 215 bytes|	二进制形式的中等长度文本数据|
|MEDIUMTEXT	|0-16 777 215 bytes|	中等长度文本数据|
|LONGBLOB	|0-4 294 967 295 bytes|	二进制形式的极大文本数据|
|LONGTEXT|	|0-4 294 967 295 bytes|	极大文本数据|

PS:char(n) 和 varchar(n) 中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。



## 二、基本语句

### DQL(数据查询语句)

* select字句的顺序
```
select ->from ->where ->group by ->having ->order by ->limit
```

* 模糊查询 like
    * %：任意字符出现任意次数
    * _:只匹配单个字符

* 汇总 

* 分组:group by

* 分页:limit

* 排序:order by

* 子查询

* 连接查询：join

* 组合查询：union

### DML(数据操纵语句)
* insert：insert into table_name ( field1, field2,...fieldN ) VALUES ( value1, value2,...valueN );
* delete:delete from table_name [WHERE Clause]
* update:update table_name set userName='kobe',age=42 where id=1


### DDL(数据定义语句)
* 增加字段：add (通过after指定位置)
>alter table table_name add field_name field_type after field_name
* 删除字段：drop
>alter table table_name  DROP field
* 修改字段类型：mofidy 
>alter  table table_name modify [COLUMN] 字段名 新数据类型 新类型长度  新默认值  新注释;
>modify可以用来修改字段类型，是否必填，注释。例如：当修改比否非必填时，字段类型和注释如果不修改，也需要带上当前值，否则会被清空
* 修改字段名称 change
>alter  table table1 change old_column new_column varchar(100) DEFAULT 1.2 COMMENT '注释';
* 修改字段默认值
> alter table talble_name alter field set DEFAULT 1000;


### DCL(数据控制语句):管理用户，授权
#### 1.管理用户
* 添加用户：
语法：CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
* 删除用户：
语法：DROP USER '用户名'@'主机名';
* 修改用户密码：
    UPDATE USER SET PASSWORD = PASSWORD('新密码') WHERE USER = '用户名';
    UPDATE USER SET PASSWORD = PASSWORD('abc') WHERE USER = 'lisi';
    SET PASSWORD FOR '用户名'@'主机名' = PASSWORD('新密码');
    SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123');


#### 2.权限管理
* 授予权限
grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';
-- 给张三用户授予所有权限，在任意数据库任意表上
GRANT ALL ON . TO 'zhangsan'@'localhost';

#### 3.撤销权限
-- 撤销权限：
revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名';
REVOKE UPDATE ON db3.account FROM 'lisi'@'%';
 

### Q&A
* delete和truncate比较
    * delete：删除表中的行，不删除表本身。
    * truncate：如果想删除表中所有行，truncate更快。truncate实际上是删除原有的表并重新创建一张表，而不是逐行的删除数据。

