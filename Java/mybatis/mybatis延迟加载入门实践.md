## mybatis延迟加载入门实践 <br>
项目框架结构为: springmvc+mybatis+mysql+maven+junit进行整合测试。就是想想看看mybatis延迟加载的实际效果。那么什么是延迟加载?延迟加载有什么作用呢? 就我自己的看来：延迟加载，就是某某东西在某某时刻进行加载，只是那个时刻晚了些，不在第一时间进行加载。其实这也算是一种思想吧，在java领域用的挺多，当然都有利弊相关，在我们软件工程领域，有句话说得好:**永远没有最好的，只有最适合的**，要的不是最好的，要的是最符合当前客观现实环境的解决方案。<br>
那这个`延迟加载`在mybatis中代表什么意思呢？指的是关联表的加载时刻，比如有两个表A,B。表A中有指向B的关联，或者说是外键都行。当程序查询表A，是否立刻关联查询出A记录关联的表B中的数据，这就叫是否延迟加载。下面来做个demo来进行实践看看吧:<br>

<hr>
  * 项目目录结构如下图:<br>
  项目的搭建过程可以参考[这里](http://wenku.baidu.com/view/4503d560c77da26924c5b0a1)，里面的junit测试例子也参考了mybatis官方例子。<br>
  ![image](https://github.com/arden2600/myblogs/blob/master/Java/images/mybatis/mybatis%E5%BB%B6%E8%BF%9F%E5%8A%A0%E8%BD%BD%E5%85%A5%E9%97%A8%E5%AE%9E%E8%B7%B5_2016_11_29.png) 
<br>
  * maven的pom.xml文件:<br>
  ```xml
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>eyas.springmvc</groupId>
    <artifactId>springmvc</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>war</packaging>

    <properties>
        <spring.version>3.2.4.RELEASE</spring.version>
        <mybatis.version>3.2.4</mybatis.version>
        <!-- log4j日志文件管理包版本  -->
        <slf4j.version>1.6.6</slf4j.version>
        <log4j.version>1.2.9</log4j.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-oxm</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.2.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/c3p0/c3p0 -->
        <dependency>
            <groupId>c3p0</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.1.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.36</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.9.13</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/cglib/cglib -->
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.2.0</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.ow2.asm/asm -->
        <dependency>
            <groupId>org.ow2.asm</groupId>
            <artifactId>asm</artifactId>
            <version>5.0.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.ant/ant -->
        <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant</artifactId>
            <version>1.9.4</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.ant/ant-launcher -->
        <dependency>
            <groupId>org.apache.ant</groupId>
            <artifactId>ant-launcher</artifactId>
            <version>1.9.4</version> 
        </dependency>
        <!-- log end  -->
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <artifactId>maven-war-plugin</artifactId>
            </plugin>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.7</source>
                    <target>1.7</target>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
 ```
<br>
  * JAVA POJO 对象:<br>
  这里参考网上使用了学生和老师，班级三者关系来进行设计。pojo类中的一对多，多对多，多对一都是重要表关联关系<br>
  **班级类(Classes)** <br>
```java
public class Classes{
  //定义实体类属性，与数据库class表字段对应
  private int id;
  private String name;
  
  /**
  * class表有一个teacher_id字段，所以在该类中定义一个teacher属性，
  * 用于维护teacher和class两个表的一对一关系。通过该类teacher属性知道班级由那位老师负责
  */
  private Teacher teacher;
  
  //一个班级有多个学生
  private List<Student> students;
  //setter and getter
  
  @Override
  public String toString(){
     return "Classes [id=" + id + ", name=" + name + ", teacher=" + teacher
                + ", students=" + students + "]";
  }
}
```
<br>
  **老师类(teacher)**<br>
```java
public class Teacher{
  private int id;
  private String name;
  
  //getter and setter
  
  @Override
  public void toString(){
        return "Teacher [id=" + id + ", name=" + name + "]";
    }
  }
}
```
<br>
  **学生类(student)**<br>
```java
  public class Student{
    private int id;
    lprivate String name;
    
    //setter and getter
    @Override
    public String toString() {
        return "Student [id=" + id + ", name=" + name + "]";
    }
  }
```
<br>
  * mapper文件:ClassMapper.xml <br>
  主要就是靠班级类Classes对三个表的关联关联关系给连接起来。班级对教师是一对一关系，班级对学生是多对多关系。延迟加载就是与表关系关联。<br>
```xml
<!--嵌套查询处理：通常是多条sql进行关联查询-->
<select id="getClassesWithStudentNestedSelect" parameterType="int" resultMap="ClassResultMap">
  select * from class where c_id=#{id}
</select>

<select id="getTeacherById" parameterType="int" returnType="teacher">
  select * from teacher where i_id=#{id}
</select>

<resultMap id="ClassResultMap">
  <id property="id" column="id"/>
  <result property="name" column="cname"/>
  <!--班级与老师的一对一-->
  <association property="teacher" column="teacher_id" select="getTeacherById"/>
  <!--班级与学生的一对多关系-->
  <collection property="students" ofType="Student" column="c_id" select="getStudentsByClassId"/>
</resultMap>
```
<br>
  * Junit测试:<br>
  在进行测试前，需要对mybatis的配置文件进行配置，在mybatis-config.xml中加上:<br>
  最重要的是启用延迟加载功能。<br>
```xml
  <!--全局映射器启用缓存-->
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="aggressiveLazyLoading" value="true"/>
```
<br>
  测试类ClassAndTeacherTest.java
```java
public class ClassAndTeacherTest {

  private SqlSessionFactory factory;
  
  @Before
  public void init() throws IOException{
  
    string resource = "conf/mybatis-config.xml";
    InputStream is = Resources.getResourceAsStream(resource);
    factory = new SqlSessionFactoryBuilder().build(is);
  }
   /*
    * 延迟加载测试
    */
    @Test
    public void getClassesWithAssociation(){
      SqlSession session = factory.openSession();
      try{
        ClassesDao mapper = session.getMapper(ClassesDao.class);
        //查询id=1班级类信息
        Classes tclass = mapper.getClassesWithStudentNestedSelect(1);
        System.out.println(tclass.getName());
        System.out.println(tclass.getTeacher().getName());
        System.out.println(tclass.getStudents());
        
         }finally{
          session.close();
         }
    }

}

```
<br>
    **仅仅查询classes内容，不显式的访问classes关联的老师和学生属性信息:**<br>
    根据查询的班级classes内容，是否查询关联的其他表信息，观察sql输出信息,只有一条sql，只是进行了一次sql查询。<br>
```java
ClassesDao mapper = session.getMapper(ClassesDao.class);
Classes clazz2 = mapper.getClassesWithStudentNestedSelect(1);
System.out.println(clazz2.getName()); //仅仅查询classes内容

//输出sql信息，可以根据输出看到仅仅是只有一条sql.
2016-11-29 10:44:13,606 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - ooo Using Connection [com.mysql.jdbc.JDBC4Connection@16e8722a]
2016-11-29 10:44:13,606 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - ==>  Preparing: ***select * from class where c_id=?*** 
2016-11-29 10:44:13,637 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - ==> Parameters: 1(Integer)
2016-11-29 10:44:13,700 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - <==      Total: 1
class_a
```
<br>
    **不仅仅查询classes内容，还关联查询教师和学生信息**<br>
    通过classes实体对应的班级表进行了关联查询,可以看到进行了三次sql查询。因为显式的查询了，所以进行了sql查询。<br>
```java
ClassesDao mapper = session.getMapper(ClassesDao.class);
Classes clazz2 = mapper.getClassesWithStudentNestedSelect(1);
System.out.println(clazz2.getName());
System.out.println(clazz2.getTeacher().getName());  
System.out.println(clazz2.getStudents());

//output:可以看到查询了三次sql。
2016-11-29 10:48:57,996 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - ooo Using Connection [com.mysql.jdbc.JDBC4Connection@512327c]
2016-11-29 10:48:57,996 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - ==>  Preparing: ***select * from class where c_id=?*** 
2016-11-29 10:48:58,028 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - ==> Parameters: 1(Integer)
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getClassesWithStudentNestedSelect] - <==      Total: 1
class_a
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getTeacherById] - ooo Using Connection [com.mysql.jdbc.JDBC4Connection@512327c]
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getTeacherById] - ==>  Preparing: ***select t_id as id, t_name as name from teacher where t_id = ?*** 
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getTeacherById] - ==> Parameters: 1(Integer)
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getTeacherById] - <==      Total: 1
teacher1
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getStudentsByClassId] - ooo Using Connection [com.mysql.jdbc.JDBC4Connection@512327c]
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getStudentsByClassId] - ==>  Preparing: ***SELECT s_id id, s_name name FROM student WHERE class_id=?*** 
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getStudentsByClassId] - ==> Parameters: 1(Integer)
2016-11-29 10:48:58,090 [main] DEBUG [cn.springmvc.dao.ClassesDao.getStudentsByClassId] - <==      Total: 3
[Student [id=1, name=student_A], Student [id=2, name=student_B], Student [id=3, name=student_C]]

```
<br>
  * 关闭延迟查询，重新测试<br>
  可以关闭mybatis的延迟加载功能，即在mybatis-config.xml中注释掉以下代码:<br>
```xml
<!--        <setting name="lazyLoadingEnabled" value="true" /> -->
<!--        <setting name="aggressiveLazyLoading" value="false" /> -->
```
<br>
然后在进行查询，在此省略。但是结果当然是，当关闭延迟加载时候，在查询classes内容时候，无论是否查询关联的老师和学生表信息，都会发出sql分别查询三个表。<br>[项目源代码](http://pan.baidu.com/s/1mivywEs)
