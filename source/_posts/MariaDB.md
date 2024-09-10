---
title: MariaDB
date: 2019-11-18 16:09:14
subtitle:
categories:
tags:
index_img: /images/mariadb.jpeg
banner_img: /images/mariadb.jpeg
---
# 启动/关闭  
启动关闭用'systemctl'命令
# `mysql`命令行工具:
- `-h、--help`: 后接服务器地址,若是本地127.0.0.1,可以省略
- `-p、--port`: 后接端口,默认是3306
- `-u、--user`: 连接MariaDB服务器时用的用户名
- `-p、--password`: 连接MariaDB服务器的密码
- `-D、--database`: 连接MariaDB服务器时要使用的数据库
- `--auto-rehash`: 在mysql客户端程序内输入表或列名时，使用TAB键可以自动补全
- `--batch`: 以批处理模式(非交互模式)运行mysql客户端程序
- `--execute、-e`: mysql客户端程序在连接MariaDB服务器的同时执行参数给出的语句
- `--skip-column-names、-N`: 在mysql客户端中不显示查询结果中的列名
- `--safe-updates、-U`: 以安全模式运行mysql客户端，安全模式下防止误操作
# 文件位置
默认数据库文件存放位置为:/var/lib/mysql
读取配置文件顺序:/etc/my.cnf、/etc/mysql/my.cnf、~/.my.cnf
# 创建、删除数据库
- `CREATE DATABASE 数据库名称 DEFAULT CHARACTER SET utf8mb4(指定字符集及排序方式)`
- `DROP DATABASE 数据库名称`
# 创建数据表
- `USE test`进入指定的数据库test
```sql
CREATE TABLE [IF NOT EXISTS] tab_test (
tid BIGINT NOT NULL AUTO_INCREMENT,
tname VARCHAR(100) NOT NULL,
tmemo TEXT NOT NULL,
PRIMARY KEY (tid),
INDEX ix_tname_tid (tname,tid)
) ENGINE=InnoDB;
```
约束分为表约束和列约束，上述NOT NULL为列约束跟在每个列定义后面,而PRIMARY KEY为表约束可以指定多个列。
完整性约束的基本语法格式：
[CONSTRAINT<约束名>]<约束类型>,中括号中内容可以省略,约束类型有:NULL/NOT NULL、UNIQUE、PRIMARY KEY、FOREIGN KEY、CHECK。 其中外键约束格式为:FOREIGN KEY REFERENCES <主表名>(<列名>),CHECK:CHECK(<约束条件>),约束条件举例:CHECK (Score>=0 AND Score<=100)
# 查看数据表
`SHOW CREATE TABLE test`
修改数据表格式
(待建设,各种情形不同处理,有点复杂)
# 删除数据表
`DROP TABLE <表名>`
# 插入元组数据
`INSERT INTO test (fd1,fd2) VALUES (1,'Matt') ON DUPLICATE KEY UPDATE fd2='Matt';`ON DUPLICATE KEY UPDATE选项可以确保若记录已存在则执行UPDATE,否则执行INSERT;
# 检索元祖数据
- `SELECT * FROM tab_test;`
- `SELECT * FROM tab_test WHERE fd1=1;`
条件查询可以有NOT、AND、OR(优先级从高到低);BETWEEN...AND...;IN(<值1>、<值2>);LIKE<字符串常量>('张％'代表姓张的人,`_力%`代表第二个字是力,%匹配0或多个字符);
## 统计汇总查询常用函数
- AVG:按列计算平均值
- SUM:按列计算值的总和
- MAX:求一列中的最大值
- MIN:求一列中的最小值
- COUNT:按列值计算个数
例子:
```sql
SELECT MAX(Score) AS MaxScore,
	   MIN(Score) AS MinScore,
	   MAX(Score)-MIN(Score) AS Diff 
	   FROM SC 
       WHERE(CNo='C1')
```
- `SELECT COUNT(DISTINCT Dept) AS DeptNum FROM S`DISTINCT关键字消除重复
- `SELECT fd2 FROM tab_test;`
```sql
SELECT SNo,COUNT(*) AS SC_Num
FROM SC
GROUP BY SNo
HAVING(COUNT(*)>=2)
```
这GROUP BY后面的属性表示若其相同，将在同一行呈现(相当于DISTINCT),但count将计算当前组的个数，而HAVING则是对GROUP BY进一步筛选,不能用where。
```sql
SELECT SNo,CNo,Score
FROM SC
WHERE CNo IN ('C2','C3','C4')
ORDER BY SNo,Score DESC
```
学号升序，分数降序排列
- `SELECT * FROM tab_test\G`注意这没有分号,按列输出记录
## 多表内连接查询
- 方法一:
```sql
SELECT T.TNo,TN,CNo
FROM T,TC
WHERE (T.TNo=TC.TNo) AND(TN='刘伟')
//这里TN=‘刘伟为查询条件’T.TNo=TC.TNo为连接条件，TNo为连接字段
```
- 方法二:
```sql
SELECT T.TNo,TN,CNo 
FROM T INNER JOIN TC
ON T.TNo=TC.TNo WHERE(TN='刘伟')
```
## 多表外连接查询
```sql
SELECT S.SNo,SN,CN,Score
FROM S
LEFT OUTER JOIN SC
ON S.SNo=SC.SNo
LEFT OUTER JOIN C
ON C.CNo=SC.CNo
//外链接不符合条件的将置为NULL
```
## 多表交叉查询
```sql
SELECT * FROM S CROSS JOIN C
//行数两个表行的乘积，列数为两个表的列数和
```
## 自连接查询(例子为查询所有比"刘伟"工资高的教师姓名、工资和刘伟的工资)
- 方法一:
```sql
SELECT X.TN,X.Sal AS Sal_a,Y.Sal AS Sal_b
FROM T AS X,T AS Y
WHERE X.Sal>Y.Sal AND Y.TN='刘伟'
```
- 方法二:
```sql
SELECT X.TN,X.Sal,Y.Sal
FROM T AS X INNER JOIN T AS Y
ON X.Sal>Y.Sal
AND Y.TN='刘伟'
```
## 普通子查询
```sql
SELECT TNo,TN
FROM T
WHERE Prof= (SELECT Prof 
		FROM T
		WHERE TN='刘伟')
//查询与刘伟老师相同职称的老师姓名与工号
SELECT TN
FROM T
WHERE (TNo = ANY (SELECT TNo
			FROM TC
			WHERE CNo = 'C5'))
//查询讲授课程号为C5的教师姓名
```
## 相关子查询(它先对外查询中每一条记录进行比对，这与普通子查询不同，普通子查询先执行子查询)
```sql
SELECT TN
FROM T
WHERE EXISTS (SELECT *
		FROM TC
		WHERE TNo=T.TNo AND CNo='C5')
//查询讲授课程为C5的教师姓名
```
## 合并查询(就是把结果合并到一起)
```sql
SELECT SNo AS 学号,SUM(Score) AS 总分
FROM SC
WHERE (SNo = 'S1')
GROUP BY SNo
UNION
SELECT SNo AS 学号,SUM(Score) AS 总分
FROM SC
WHERE (SNo = 'S5')
GROUP BY SNo
```
## 存储查询
```sql
SELECT SNo AS 学号,SUM(Score) AS 总分
INTO Cal_Table
FROM SC
GROUP BY SNo
//如果在新表名前面加个#则是临时表，关闭则消失
```
# 修改元祖数据
`UPDATE tab_test SET fd2='Brandon' WHERE fd1=1`如果没有WHERE则所有元祖的fd2都将改变，请一定小心
`REPLACE tab_test SET fd1=1,fd2='Matt';`若记录存在执行UPDATE,否则执行INSERT,最好用INSERT加ON DUPLICATE KEY UPDATE命令,资源消耗更小。
# 删除元祖数据
`DELETE FROM tab_test WHERE fd1=1;`同理没有WHERE将会删除所有元组
# 视图(是一个虚表,基于基本表，对其修改会影响基本表，但本身不占内存)
```sql
CREATE VIEW Sub_T
AS SELECT TNo,TN,Prof
FROM T
WHERE Dept = '计算机'
//创建一个计算机系老师情况的视图Sub_T
ALTER VIEW S_SC_C(SN,CN,Score)
AS SELECT SN,CN,Score
 FROM S,C,SC
 WHERE S.SNo=SC.SNo AND SC.CNo=C.CNo
//修改,就是查询内容的覆盖
```
对表的操作适用于视图，可以通过视图简化操作
# 索引(待建)
# 规则约束  
## 创建规则
`CREATE RULE age_rule AS@age >=18and @age <= 50`
## 将规则绑定到数据库的对象上，或者将规则从数据库的对象上松绑
用sp_bindrule绑定规则
`EXEC sp_bindrule 'age_rule','S.Age'`对已输入的数据不起作用
`EXEC sp_unbindrule 'S.Age'`解绑
## 删除规则
`DROP RULE age_rule`删除前必须先解绑
# 默认  
## 创建默认
`CREATE DEFAULT birthday_defa AS '1978-1-1'`
## 查看默认
`EXEC sp_helptext birthday_defa`
## 默认的绑定与解绑
`EXEC sp_bindefault 'birthday_defa' 'S.[Birthday]'`
`EXEC sp_unbinefault 'S.[Birthday]'`
## 删除默认
`DROP DEFAULT birthday_defa`
