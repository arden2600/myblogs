## 解决基于SpringMVC+MYSQL注解@Transaction事务无效问题:<br>
最近在使用springmvc结合mysql开发时候，遇到一个问题，基于注解的事务配置在程序运行中事务无效，即不进行事务回滚。如今mysql5.6版本都是使用Innodb存储引擎的，该引擎默认支持事务回滚的。下面说说如何解决该问题......<br>

 * 【代码块-service layout】Service.java
  ```java
  @Transactional(rollbackFor=Exceptions.class)
  public Map<String,Object> deleteStaffByIds(Long currentStaffId,List<Long> staffsIdList,
                                             List<Long> merchantsId,Long cityId) throws Exception{
      try{
        staffDao.deleteStaffAndMerchants(staffsIdList,merchantsId);
        staffDao.deleteStaffAndRole(staffsIdList);
        staffDao.deleteStaffAndCity(staffsId,cityId);
      } catch(){
        log.error("关联删除人员信息失败，失败信息： "+e.getMessage());  
        logService.insertWebLog(currentStaffId, sOperateClass, sMethodName, "失败",retMesg);  
        throw e;  
      }                                       
                                             
  }
  ```
  <br>
  上面代码块使用了基于@Transaction注解事务进行回滚，测试设置：使得在删除前面两个方法成功后，让最后第三个方法在执行操作数据库时候抛出异常。正常情况下会进行事务回滚，即前面两个方法操作的修改会被undo还原。但是在测试时候发现没有进行事务回滚。>_<|||..... <br>
  之后，在代码逻辑与事务注解规范都没有问题的情况下，检查了配置文件applicationContext.xml全局配置文件。<br>
<br>
  


---
  * applicationContext.xml全局配置文件:<br> 
```xml
<!--设置需要进行spring注解扫描的类包-->
<context:component-scan base-package="com.xiansky"/>
<!--配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager" p:dataSource-ref="dbDataSource"/>
<!--开启事务注解功能-->
<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
```
<br>
检查几遍，发现数据库连接池与事务管理器，注解扫描都没有问题。蛋疼啊...那就再检查springMvc配置文件。<br>
<br>
  * springMVC-servlet.xml配置文件:<br>
```xml
<!-- 当我们需要controller返回一个map的json对象时，可以设定<mvc:annotation-driven /> -->
<mvc:annotation-driven/>
<!--设置需要进行spring注解扫描的类包-->
<context:component-scan base-package="com.xiansky"/>
```
<br>
大体看上去，mvc的配置文件也没有什么异常，但是列，注意到mvc配置文件里面与全局配置文件里面有段相同的代码`<context:component-scan base-package="com.xiansky"/>`,就是这段配置配置IOC容器bean扫描路径的代码。<br>
之后，结合了一片文章，该篇文章在[这里](http://blog.csdn.net/z69183787/article/details/37819831) ,更加深刻的了解到了，当web容器启动的时候，若是项目中同时使用了`spring`和`springmvc`的话，那么会初始化`两个容器`，**父子级别的容器**。在容器装配bean的时候会根据配置的扫描包进行装配。问题就是出现在这了，对项目的`com.xiansky`目录下的包进行了`两次扫描`装配bean。<br>
<br>
**第一次**是在读取applicationContext配置文件时候，对应的父容器。**第二次**springmvc配置文件对应的子容器。<br>
所以，回过头看看上面的配置：注意到上述的配置文件：<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" /> 该段配置事务注解是放在applicationContext中，即`只是对父容器中扫描的类具有注解事务功能`，在springMVC中的配置并没有配置该内容，所以造成了：<br>
`在第一次扫描装配bean的时候，bean是具有事务功能的`。但是在springmvc容器的二次扫描重新装配，会**覆盖父类**容器中具有事务功能的bean，因为在mvc配置文件中没有开启事务管理，所以导致了，在上面的`service层的Service.java类被springmvc子容器装配，失去了事务回滚机制功能`。<br>
<br>


所以，综上有两种解决方法：<br>
  * A:在springMVC子容器扫描包时候，不扫描使用到事务的service层所有类。配置如下:<br>
```xml
<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true">
  <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
</tx:annotaion-driven>  
```
<br>这样service层的类就不回覆盖掉具有回滚事务的bean了。<br>
  * B:将父类applicationContext.xml配置文件中的事务注解驱动，剪切配置到子容器springMVC-servlet.xml配置文件中:<br>
  springMVC-servlet.xml配置文件部分内容如下:<br>
```xml
<context:component-scan basek-package="com.xiansky"/>
<tx:transaction-driven transaction-manager="transactionManager" proxy-target-class="true"/>
```
<br>
这样重新测试就搞定了。主要是意识到application.xml与springMVC-servlet.xml对应了两个具有父子级别的容器。
