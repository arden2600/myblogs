## 忘记MySQL root密码如何重置?<br>
有时候会将安装的mysql密码给忘记，或者在安装mysql时候，mysql会为我们随机生成密码，这并不是我们想要的，这时候，我们可以重新设定mysql的密码。<br>
系统环境:<br>
<table border="border">
  <tr>
    <th>mysql版本</th>
    <th>系统版本</th>
  </tr>
  <tr>
    <td>Mysql 5.7</td>
    <td>Ubuntu 15.x x64</td>
  </tr>
</table>
---
  * 首先应该先停止mysqld服务。(mysqld为安装mysql设定的服务进程名称)<br>
  ```linux
  # ps -ef|grep mysql
  # service mysqld stop
  ```
  <br>其中mysqld是在安装mysql后，安装目录中子目录support-files/mysql.server服务名。也是可以由用户自定义的。<br>
  * 将所有关于mysql的进程清理(pid为mysqld服务进程号)<br>
  ```linux
    # ps -ef|grep mysqld
    # kill -s 9 pid
  ```
    若是不想强制删除，也可以是用`service mysqld stop`进行停止mysql服务进程。<br>
  * 以`mysqld_safe`模式启动mysql:<br>
  ```linux
  # mysqld_safe --skip-grant-tables &
  ```
  若是没有将mysql主目录/bin路径配置到$PATH系统变量中，那么需要进入bin路径下执行该命令。执行完之后，若是出现错误:`mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended',则是表明mysqld/-safe进程启动失败，可以查看日志，解决办法参考[这里](http://www.ithov.com/server/130920.shtml).<br>
  * 进入mysql（注意现在是没有权限操作mysql表的）<br>
  ```linux
  #mysql -h localhost -uroot
  ```
  * 这个时候就可以修改root密码：(注意mysql5.7的user表的密码字段改变了)<br>
  可以查看user表中密码字段:<br>
  ```sql
  mysql> use mysql;
  mysql> desc user;
  ```
  可以看到如下内容:<br>
  ![image](https://github.com/arden2600/myblogs/blog/master/Database/mysql/images/forget-mysql-root-password.png)
  <br>
  * 设置密码:(code为自定义密码)<br>
  ```sql
  mysql> update user set authentication_string=PASSWORD('code') where user='root';
  ```
  **刷新权限:**<br>
  ```sql
  mysql> flush privileges;
  mysql> quit;
  ```
  退出。<br>
  * 重新登陆mysqld服务,root登陆:<br>
  ```linux
  # service mysqld start
  # mysql -hlocalhost -uroot -p
  Enter password:
  ```
  * 当在上一步设定密码，退出重新登陆之后，再一次 mysql server模式确定设置密码:<br>
  ```sql
  mysql> SET PASSWORD=PASSWORD('code')
  ```
  至此，root密码设定成功。主要注意的就是最新的MySQL5.7密码安全的设置有些小小改动，安装过程参照`README.TXT`也是可以知道小变动。
