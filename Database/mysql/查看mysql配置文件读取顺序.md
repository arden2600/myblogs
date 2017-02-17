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
那么，就可以从结果中知道mysql服务启动时候，那几个配置文件的读取顺序了。后读取的配置文件中，若有与之前相同的配置项，那么是会覆盖的。
