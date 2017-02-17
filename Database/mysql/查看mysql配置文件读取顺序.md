## 如何查看mysql配置文件的读取优先级<br>
查看mysql的配置文件的启动或者说是读取顺序。对已一些mysql配置文件，有时候会被其他位置的配置文件覆盖出现问题，那么如何去查看mysql服务器启动时，配置文件的读取顺序呢？<br>
系统环境:<br>
<table border="border">
  <tr>
    <th>Mysql版本</th>
    <th>linux版本</th>
  </tr>
  <tr>
    <td>5.6</td>
    <td>6.5</td>
  </tr>
<table>
<br>
  * 先查看mysqld服务进程所在位置：(msyql 启动与否都可以查到)<br>
```linux
   #which mysqld
```
可以看到mysql服务的进程情况。
<br>
  * 使用mysqld命令执行以下命令:<br>
```linux
#mysqld  --verbose --help|grep -A 1 'Default options'
```
<br>
可以看到输出结果：<br>
```linux
Default options are read from the following files in the given order:<br>
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf 
```
<br>
那么，就可以从结果中知道mysql服务启动时候，那几个配置文件的读取顺序了。后读取的配置文件中，若有与之前相同的配置项，那么是会覆盖的。<br>
<br>
## 处理".mysql_history"文件小技巧<br>
当mysql服务器在运行的时候，mysql用户在"mysql\>"这个shell中的执行命令都会被记录下来。通常存在`$HOME/.mysql_history`中。<br>
所以在若是在mysql中进行大量的SQL操作，日积月累文件容量也会增长，若是在一些特定环境中节省容量，或者基于其他一些考虑(例如安全等)，要将历史内容清空，那么可以利用Linux系统上的`/dev/null`这个无底洞文件。<br>
`dev/null`这个特殊的文件类似"黑洞"或者"无底洞"。所有存在其里面的内容都会无效或者‘消失’，既不占容量也不可查看内容。所以我们可以使用软连接，将.mysql_history 文件链接到黑洞之中。<br>
<br>
---
  * 可以先查找.mysql_history文件(注意点号):<br>
```linux
  $find / -name .mysql_history   /*在根目录/下，根据文件名来查找文件*/<br>
```
<br>
  * 将.mysql_history文件删除：<br>
```linux
$cd ~
$rm -rf .mysql_history
```
<br>
  * 建立软连接:<br>
```linux
  $ ln -s /dev/null ~/.mysql_history
```
<br>
  * 验证效果:<br>
  进入mysql命令行执行操作命令，在退出。<br>
  查看.mysql_history文件内容: `$cat ~/.mysql_history`,可以发现，文件中什么内容也没有，之前操作的mysql命令也没有记录下来。<br>
  若是仅仅是某次想清空该文件中历史操作记录，可以操作:<br>
  ```linux
    $ cat /dev/null > ~/.mysql_history
  ```
