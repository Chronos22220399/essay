# SqlServer 实验日志 -- sql

> 实验在 ssms 上进行，因此下面的一些语句只能在 ssms（sqlserver） 上运行

可以通过点语法跨数据库进行操作，例如：

```sql
select * from newdb.student;
```

> 该语法适用于所有语句

### 创建数据库

`create database databaseName`

### 使用 SQL 语句查看数据库属性

`sp_helpdb databaseName`

### 修改数据库属性

#### 定义文件组

```sql
alter database databaseName
add filegroup filegroupName go;
```

> 这里的 go 表示批次，即这段代码是一个批次的，T-sql 中以 go 表示批次，运行时有 go 则将 go 前面的语句当初一个批次，此时运行会按批次执行，不会像没有 go 那样一次运行全部了

#### 添加文件到文件组中

```sql
alter database databaseName
add file (
name = fileName,
filename = 'C:\\filePath\fileName.mdf(或者 ndf)',
size = 8mb,
filegrowth = 5%,
maxsize = 500mb
)
to filegroup filegroupName
go;
```

其中：

1. fileName 是逻辑名称（可能是我们直接使用的名称）
2. filename 后面的是文件的路径，这个文件必须是 mdf 文件或者 ndf 文件
3. size 表示创建的初始大小
4. filegrowth 表示文件大小的增长率
5. maxsize 表示文件被运行的最大大小
6. to file group filegroupName 表示将该文件添加到文件组 “filegroupName” 里

#### 设置默认文件组

```sql
alter database databaseName
modify filegroup filegroupName default
go;
```

### 创建表

```sql
create table tableName(
	columnName1 type [not null] [default someValue] [unqiue],
  columnName2 ...,
  ...
)
go;

-- 示例
create table Coursinfo(
  cno char(11) not null primary key,
  cname char(15) default null
)
go;
```

### 修改表的基本结构

```sql
-- 增
alter table tableName
add columnName type [not null] [default someValue] [unique]
go;

-- 删
alter table tableName
drop column columnName
go;

-- 改
alter table tableName
modify [column] columnName type [not null] [default someValue] [unique]
go;
```

### 删除表

```sql
drop table tableName
go;

-- or
drop table someDB.tableName
go;
```

> 前者删除当前数据库中的表，但无法跨数据库删除表；后者可以跨数据库删除表

### 创建索引

```sql
create [unique] [clustered] [nonclustered] index indexName on tableName(columnName1, ...)
[with index_property1, ...]
go;
```

> 索引可以由表中的多个元素构成，且一张表能有多个索引

1. unique 表示创建唯一索引
2. clustered 表示创建聚簇索引
3. nonclustered 表示创建非聚簇索引
4. index_property1 表示索引属性，索引属性可以有多个

#### 查看表中的索引信息

```sql
sp_helpindex tableName
go;
```

> 该语法适用于 T-sql

### 删除索引

```sql 
drop index indexName on tableName;
```

### 分离数据库

#### 查看数据库的文件信息（以便于寻找，然后附加）

```sql
exec sp_helpdb databaseName
go;
```

> exec 或 execute 是用来执行一个存储过程、批处理、脚本或动态SQL语句的命令。它可以用于执行已经定义好的存储过程，或者构建并运行动态生成的SQL语句。

#### 分离数据库

```sql
exec sp_detach_db databaseName, true
go;
```

### 附加数据库

```sql
exec sp_attach_db @dbname = 'databaseName',
@filename1 = 'filePath\fileName1.mdf',
@filename2 = 'filePath\fileName2.ndf',
@filename3 = 'filePath\fileName3.ldf'
go;
```

其中：

1. @dbname 表示将文件附加到名为 databaseName 的数据库中
2. @filename1/2/3/... 表示文件 'filePath\fileName1.mdf' / 'filePath\fileName2.ndf' / 'filePath\fileName3.ldf' / ... 就是要附加的文件

> 我们在实际附加时，可能只能在同名（同名不同类型）的文件中看到 mdf 和 ldf 类型的文件，这是正常的，mdf 是数据文件，ldf 是日志文件

### 删除数据库

```sql
drop database databaseName
go;
```

### 插入数据

```sql
insert into table(column1, ...)
values(value1, value2, ...)
go;
```

> value1 / 2 / 3 /... 的类型要和 column1 / 2 / 3 / ... 一一对应

### 查询数据

```sql
select * / someColumns1 from tableName
where condition1
[group by someColumns2]
[having condition2]
[order by someColumns3 [ASC] / [DESC]]
go;
```

其中：

1. *：表示全部选中（即所有列）
2. someColumns1/2/3/...：表示某些列（>= 1)
3. 排序默认为 ASC，即升序排列，DESC 表示降序排列