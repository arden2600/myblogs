## tomcat多实例部署实践 <br>
apache真是牛逼的，搞了很多成功有用的项目。tomcat服务器通常我们用，有时候真不知道tomcat还有可以部署多实例这东西。就是前几天的一个小型上线项目被tomcat的多实例给坑了，日志路径搞错了。所以打算看看tomcat多实例的相关内容，并且进行了实践操作配置，发现是个很简单的东西。但是，好处挺多的 :) <br>
<br>
那么单个tomcat实例部署多个应用与多个tomcat实例部署多个应用的原理和区别是什么的？可以看看这篇文章:[tomcat单实例与多实例](http://www.cher7.com/?id=12919) <br>
<br>
以下只是说说实际动手操作的过程与结果:(仅供参考哈)<br>
  * 下载tomcat压缩包并解压创建以下目录结构:<br>
  主文件夹为tomcat7<br>
```xml
--tomcat7
  --apache-tomcat-7.0.53(解压后tomcat)
  --tomcat-ins(多实例部署主文件夹)
    --application01(应用1)
      --conf(从apache-tomcat-7.0.53处复制修改端口)
      --logs
      --temp
      --webapps
        --springmvc(应用项目)
      --work
      --start.bat(该应用启动脚本)
    --application02(应用2)
      --conf(从apache-tomcat-7.0.53处复制修改端口)
      --logs
      --temp
      --webapps
        --springmvc(应用项目)
      --work
      --start.bat(该应用启动脚本)
```
<br>
其中，在tomcat-ins目录下创建实例的时候，除了将conf目录从解压后tomcat的下载包中复制过来，其他的子文件可以通过mkdir命令创建。(windows版本)可以通过`tree`指令在cmd控制台查看项目的结构<br>

<br>
  * 修改每个tomcat实例conf目录下的server.xml配置文件，包括以下几个部分:<br>
  端口自己设定，只要不冲突即可。
```xml
<Server port="8012" shutdown="SHUTDOWN">
<Connector port="8089" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" URIEncoding="UTF-8" />
<Connector port="8010" protocol="AJP/1.3" redirectPort="8443" />
```
<br>

  * 部署项目:<br>
  项目的部署可以通过eclipse或者其他IDE部署，也可以自己把编译后的项目文件拷贝到tomcat服务的webapps文件夹下进行发布，根据自己需求部署。<br>
  <br>
  * 编写每个实例启动脚本(windows为例):<br>
  脚本的JAVA_HOME等路径按照自己情况设定。%CD%表示当前所在路径。
```bat
@echo off
set JAVA_HOME=D:\Program Files\Java\jdk1.7.0_75
set PATH=%JAVA_HOME%\bin;%PATH%
set CATALINA_BASE=%CD%
cd ../../apache-tomcat-7.0.53/bin
catalina.bat start
```
<br>
  * 启动tomcat服务:<br>
  我自己开了三个tomcat，一个是apache-tomcat-7.0.53，主tomcat服务和tomcat-ins目录下的两个tomcat实例。主服务的端口号是8088，ins下两个实例端口号分别是8089和8090。然后可以分别运行apache-tomcat-7.0.53/bin/startup.bat和application01、application02下的start.bat，启动它们。部署在它们webapps目录下的应用将会发布成功了，可以访问。<br>
  三个端口运行部署同一个应用的界面分别如下:<br>
  ![image](https://github.com/arden2600/myblogs/blob/master/ServerR/images/tomcat_multi_ins_1_2016_12_1.png) <br>
  ![image](https://github.com/arden2600/myblogs/blob/master/ServerR/images/tomcat_multi_ins_2_2016_12_1.png) <br>
  ![image](https://github.com/arden2600/myblogs/blob/master/ServerR/images/tomcat_multi_ins_3_2016_12_1.png) <br>
  <br>
  可以看到，三个端口都可以成功访问属于自己项目下的资源文件，虽然是同一个项目分别部署到三个tomcat服务上，但是三个项目都是独立的，完全解耦，不会相互依赖，任意一个项目down掉或者重新发布都不会影响其他服务。好处也就显而易见了... :)
