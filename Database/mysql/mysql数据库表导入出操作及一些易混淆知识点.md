## mysql表结构和数据命令行的导入导出<br>
有时候需要在命令行对mysql数据库表和表内数据进行导入导出，那么可以是用mysqlsump命令来操作。<br>
<br>
 * 导出表结构和数据:<br>
 操作格式: `mysqldump -uroot -ppassword database table1 table2 >  foo.sql` <br>
  这样可以将数据库database中的表table1，table2以sql形式导出到foo.sql文件中。其中`-uroot`参数表示访问数据库的用户名是root,如果还有密码，则可以是用`-p`参数加上密码即可。<br>
```shell
  path> mysqldump -uroot -pmysql output_table > c:\mysqldata\output_table.sql
```
 * 导入数据或者数据库到mysql<br>
    mysql数据导入也是很方便的，主要的就是注意导入的sql文件内容的正确性，还要注意文件内容的编码，不然会出现乱码。<br>
    导入格式: `mysql -hlocalhost -uroot database < input_table.sql` 便可以将input_table.sql中sql语句导入mysql并执行。<br>
<br>
 * 以下是数据库结构或者数据相关内容具体操作例子:<br>
   * 导出整个数据库<br>
    **命令:** mysqldump -h数据库主机Ip -u用户名 -p密码 数据库名称 > 导出的文件名<br>
    C:\>mysqldump -hlocalhost -uroot -pmysql test > C:\test.sql<br>
   * 导出一个表或者多个表，包括表结构和数据<br>
    **命令：**mysqldump -h数据库主机Ip -u用户名 -p密码 数据库名称 表名1 表明2 > 导出的文件名<br>
    C:\> mysqldump -hlocalhost -uroot -pmysql test table1 table2 > c:\test_table.sql<br>
   * 若是只是想仅仅导出数据库或者表的结构，不需要数据，那么可以添加`-d`参数<br>
    导出一个数据库结构: mysqldump -h数据库主机Ip -u用户名 -p密码 -d 数据库名称 > 导出的文件名<br>
    导出数据库中表的结构:mysqldump -h数据库主机Ip -u用户名 -p密码 -d 数据库名称 表名1 表明2 > 导出的文件名。<br>
    数据库和表结构的导出相差不大，仅仅是添加-d参数即可。<br>
<br>
## mysql一些混淆知识点总结:<br>
 总结一些mysql一些基础的，不注意会容易混淆的知识点。
 * **char和varchar的区别，特点:**<br>
  VERCHAR是一个可变长度的字符串。<br>
  CHAR是一个定长的字符串。<br>
  首先我们在是用char或者varchar时候，必须指定长度，例如char(20),verchar(20)。<br>
  如果存储字符串长度为5，那么char就会在系统内存中开辟20个长度来存储长度为5的字符串；而verchar就会自动缩减，使用5个长度来存储信息，可以节省空间。由于verchar会动态对字符串内容增加缩减，那么会对性能有些影响，相对而言，在某种程度上，char性能更高，但适用性verchar更普遍。<br>
 * **Timestamp与Datetime类型区别:**<br>
  Timestamp在进行insert操作时候，若是`没有赋值，则会取系统当前时间值`。<br>
  如果对记录进行update操作，没有处理timestamp字段，那么它的值也就修改为当前系统时间，从而记录这条记录修改时间。<br>
  Datetime字段，若是没有主动直接赋值的话，那么默认就是null值。<br>
   **drop table 与delete table 以及 truncate table区别？**<br>
   * 若是在处理内容方面不同：<br>
      Drop table是用于删除表结构。Delete table是删除表的数据，不删除表结构。Truncate table 也是用于删除表中记录。<br>
      <br>
   * 那么同样是删除数据库表数据的delete与truncate table的区别在那里?<br>
    1.delete from**`是一条一条记录删除`**。`truncate table它的作用是先将表结构直接删除，那么数据也没有了。然后在重新创建表结构`。<br>
    2.两者处理数据的方式不同，从而导致两者在处理数据性能上的差异：`在删除所有数据操作上，truncate性能更高。`<br>
    3.事务上：`delete`是DML语句，`是受事务控制`的：在一个事务氛围控制内，若是delete from删除指定表中数据，可以通过rollback回滚恢复删除内容的。<br>
      `truncate` 是DCL语句，`不受事务控制`。意味着在删除表记录后，通常是不能通过事务回滚方式来恢复表记录的。<br>
 * **having 与 where的区别:**<br>
    1.**使用位置不同**：where是在group by之前使用的，而having是在group by之后使用。<br>
    2.**是否可以使用分组函数**: where后面不可以使用分组函数，而having后面可以使用分组函数。<br>
<br>
 * **常见query关键字执行顺序：select [..] from [table]  where [conditions] group by [..] having [conditions]  order by [...]**<br>
  select语句解析顺序:<br>
  1.0 From 表  ：先执行from语句查询指定表。代表从那张表中查询数据，指定数据源。<br>
  2.0 where条件：通过where关键字后的条件过滤一部分数据，将表中所有不符合条件记录过滤掉。<br>
  3.0 Group by ：得到的表记录中，根据指定字段进行分组。<br>
  4.0 Having   ：用于在分组后使用指定条件对分组后的结果进行再次过滤。<br>
  5.0 select   ：在得到的所有表字段的所有记录中，指定要查询那些数据【列属性】。将那些列字段记录数据提取到结果集。<br>
  6.0 order by ：最后在根据指定的列字段对最后显示的结果集进行排序。<br>
  
