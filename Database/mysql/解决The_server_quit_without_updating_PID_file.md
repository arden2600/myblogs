##解决"The server quit without updating PID file."问题思路集合<br>
在运行或者安装mysql的时候，总会在二次安装，或者修改了一些linux主机配置，就会引起一些mysql关联配置文件的错误。所以可以罗列出一些解决方案。<br>
这个问题是当重新修改主机名(HOSTNAME)或者其他的一些与系统用户名配置，导致mysql服务启动失败，出现以下错误信息:<br>
`The server quit without updating PID file (....)`,则可以参考以下解决方案:<br>
<br>
  * 启动mysql时候，出现以下错误:<br>
  ![image](https://github.com/arden2600/myblogs/blob/master/Database/mysql/images/mysql_start_fail_1.png)
  这个时候呢，需要将之前/etc/mysql下my.cnf配置文件(如果存在)备份后删除。或者是将一些与用户名环境相关的配置文件也删除掉<br>
```linux
# cd /etc/mysql
# cp my.cnf my.cnf.bak
# rm -rf my.cnf
```
<br>
  * 尝试重启mysql服务，出现以下错误:<br>
  ![image](https://github.com/arden2600/myblogs/blob/master/Database/mysql/images/mysql_start_fail_2.png)
  这个时候，需要使用命令`cd /usr/local/mysql/data`进入mysql安装目录的data子目录：<br>
  在通过`la`命令可以看到如下有`ib_log*`开头的文件:<br>
  ![image](https://github.com/arden2600/myblogs/blob/master/Database/mysql/images/mysql_data_list.png)
  现在只是需要删除那两个文件即可。两个文件是记录mysq_Innodb引擎日志相关信息的，与之前用户的PID文件有关联。<br>
```linux
# rm -rf ib_logfile0
# rm -rf ib_logfile1
```
  * 最后重启mysql服务，即可正常启动mysql了。
  
  
