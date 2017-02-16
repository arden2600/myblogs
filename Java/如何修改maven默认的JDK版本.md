## 如何修改maven默认jdk版本? <br>
* 由于JDK版本的不同会对项目的构建有很大的影响。而如今3.x 版本的maven默认的JDK是1.5。即使本地JDK不是1.5,在eclipse中构建maven项目时候，声明还是1.5版本。<br>

---

### 系统平台环境
  开发平台: Eclipse EE 4.5+ <br>
  JDK Version: 1.8 <br>
  Maven Version: 3.x <br>
<br>
如果想要配置成自己想使用的JDK，那么就要自己配置修改JDK版本。（配置最新JDK1.8）<br>
maven的配置文件：用户配置和全局配置<br>
  用户配置：   {user_home}/.m2/settings.xml <br>
  全局配置：   {maven_home}/conf/settings.xml

---
* 配置maven项目的pom.xml文件，添加如下内容：

 ```java
  <profile>  
      <id>jdk-1.8</id>  
  
      <activation>  
        <activeByDefault>true</activeByDefault>  
           <jdk>1.8</jdk>  
      </activation>  
      <properties>  
        <maven.compiler.source>1.8</maven.compiler.source>  
            <maven.compiler.target>1.8</maven.compiler.target>  
        <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>  
      </properties>  
        
    </profile>  
   ```
  最后在update以下项目即可将项目使用的JDK更改过来。
