---
title: Maven_SSH_Oracle项目搭建全流程
date: 2017-06-27 23:02:44
tags: [JavaWeb]
---


[TOC]
# 系统搭建

**github:** https://github.com/seawaylee/maven-ssh-quickstart

# 1 SSH框架搭建

## 1.1 SVN创建项目
```
cd /svn
sudo svnadmin create nwrd2017    # 创建下面
sudo chown -R www-data:www-data /svn  # 修改权限
cd /svn/auth
sudo vim dav_svn.authz  # 为项目配置用户及权限
sudo htpasswd dav_svn.passwd ${username}  # 为新用户设置密码
sudo service apache2 restart   # 重启项目
```
访问 `https://${SVN_IP}/svn/${project_name}` 查看项目结构


<!--more-->

## 1.2 SSH框架搭建 - Maven

**项目结构**
![](14985736654697.jpg)



### 1.2.1 创建MavenWeb项目
![](14985714190385.jpg)

### 1.2.2 引入SSH相关jar包


```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.edu.ncut</groupId>
    <artifactId>NWRDServer</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>NWRDServer</name>
    <url>http://localhost:8080/NWRDServer</url>


    <properties>
        <spring.version>4.3.9.RELEASE</spring.version>
        <hibernate.version>5.1.0.Final</hibernate.version>

        <ojdbc.version>11.1.0.7.0</ojdbc.version>

        <!--下面这两个是springAOP需要用到-->
        <aspectjweaver.version>1.7.2</aspectjweaver.version>
        <persistence-api.version>2.1</persistence-api.version>


        <junit.version>4.12</junit.version>
        <druid.version>1.0.4</druid.version>
        <jedis.version>2.9.0</jedis.version>

        <slf4j.version>1.7.7</slf4j.version>
        <log4j.version>1.2.17</log4j.version>

        <javaee-api.version>7.0</javaee-api.version>
        <jstl.version>1.2</jstl.version>
        <jsp-api.version>2.0</jsp-api.version>
        <servlet-api.version>2.4</servlet-api.version>

        <commons-fileupload.version>1.3.1</commons-fileupload.version>
        <commons-io.version>2.4</commons-io.version>
        <commons-lang3.version>3.3.2</commons-lang3.version>
        <commons-email.version>1.3.2</commons-email.version>
        <commons-beanutils.version>1.9.2</commons-beanutils.version>
        <commons-pool2.version>2.4.2</commons-pool2.version>

        <httpclient.version>4.3.3</httpclient.version>
        <jackson-databind.version>2.8.7</jackson-databind.version>
        <fastjson.version>1.1.43</fastjson.version>

        <!--架包版本变量 end-->

        <!--插件版本变量 start-->
        <tomcat7-maven-plugin.version>2.2</tomcat7-maven-plugin.version>
        <maven-compiler-plugin.version>3.1</maven-compiler-plugin.version>
        <maven-war-plugin.version>2.3</maven-war-plugin.version>
        <maven-resources-plugin.version>2.6</maven-resources-plugin.version>
        <maven-install-plugin.version>2.4</maven-install-plugin.version>
        <maven-clean-plugin.version>2.5</maven-clean-plugin.version>
        <maven-antrun-plugin.version>1.7</maven-antrun-plugin.version>
        <maven-dependency-plugin.version>2.5.1</maven-dependency-plugin.version>
        <maven-source-plugin.version>2.2.1</maven-source-plugin.version>
        <!--插件版本变量 end-->

        <!--其他变量 start-->
        <war-name.version>NWRDServer</war-name.version>
        <jdk.version>1.7</jdk.version>

        <tomcat-port.version>8080</tomcat-port.version>
        <tomcat-uri-encoding.version>UTF-8</tomcat-uri-encoding.version>
        <tomcat-path.version>/</tomcat-path.version>
        <!--其他变量 end-->

        <!--这个配置是为了解决下面两个警告-->
        <!--Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!-->
        <!--File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!-->
        <!--指定Maven用什么编码来读取源码及文档-->
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <!--指定Maven用什么编码来呈现站点的HTML文件-->
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>


    </properties>

    <dependencies>

        <!--Spring -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-messaging</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-oxm</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--这个jar 文件包含Spring MVC 框架相关的所有类。包括框架的Servlets，Web MVC框架，控制器和视图支持。当然，如果你的应用使用了独立的MVC 框架，则无需这个JAR 文件里的任何类。 外部依赖spring-web-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-orm</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--这个jar 文件包含支持UI模版（Velocity，FreeMarker，JasperReports），邮件服务，脚本服务(JRuby)，缓存Cache（EHCache），任务计划Scheduling（uartz）方面的类。 外部依赖spring-context-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>${aspectjweaver.version}</version>
        </dependency>

        <dependency>
            <groupId>javax.persistence</groupId>
            <artifactId>persistence-api</artifactId>
            <version>${persistence-api.version}</version>
        </dependency>


        <!-- Hibernate -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>

        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-ehcache</artifactId>
            <version>${hibernate.version}</version>
        </dependency>


        <!-- 二级缓存ehcache -->
        <dependency>
            <groupId>net.sf.ehcache</groupId>
            <artifactId>ehcache</artifactId>
            <version>2.9.0</version>
        </dependency>


        <dependency>
            <!--这个组件具体可以看这里介绍:http://www.oschina.net/p/druid-->
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid.version}</version>
        </dependency>

        <!-- JSTL标签类 -->
        <dependency>
            <groupId>jstl</groupId>
            <artifactId>jstl</artifactId>
            <version>${jstl.version}</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>${servlet-api.version}</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jsp-api</artifactId>
            <version>${jsp-api.version}</version>
            <scope>provided</scope>
        </dependency>

        <!-- java ee jar 包 -->
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>${javaee-api.version}</version>
            <scope>provided</scope>
        </dependency>

        <!--单元测试-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- 日志文件管理包 start -->
        <!--下面这三个是配套使用：http://blog.csdn.net/woshiwxw765/article/details/7624556-->
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
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
        <!-- 日志文件管理包 end -->


        <!--JSON处理-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>${jackson-databind.version}</version>
        </dependency>

        <!-- 上传组件包 -->
        <dependency>
            <groupId>commons-fileupload</groupId>
            <artifactId>commons-fileupload</artifactId>
            <version>${commons-fileupload.version}</version>
        </dependency>

        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>${commons-io.version}</version>
        </dependency>


        <!--apache工具包-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>${commons-lang3.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
            <version>${commons-pool2.version}</version>
        </dependency>

        <dependency>
            <groupId>oracle</groupId>
            <artifactId>ojdbc6</artifactId>
            <version>${ojdbc.version}</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>


    </dependencies>


    <build>
        <finalName>NWRDServer</finalName>
        <resources>
            <!--表示把java目录下的有关xml文件,properties文件编译/打包的时候放在resource目录下-->
            <resource>
                <directory>${basedir}/src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
            </resource>
            <resource>
                <directory>${basedir}/src/main/resources</directory>
            </resource>
        </resources>
        <plugins>
            <!-- Compiler 插件, 设定JDK版本 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>${maven-compiler-plugin.version}</version>
                <configuration>
                    <source>${jdk.version}</source>
                    <target>${jdk.version}</target>
                    <encoding>UTF-8</encoding>
                    <showWarnings>true</showWarnings>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

- 注意各jar包之间的依赖、版本号
- oracle驱动由于授权问题，无法从maven仓库直接导入，可以先从官网下载ojdbc6，然后手动存储到maven-repository

[OJDBC6下载地址](http://download.oracle.com/otn/utilities_drivers/jdbc/11204/ojdbc6.jar?AuthParam=1498553400_8025275bf16d0c68258bc163e60b198a)

**手动安装**
`mvn install:install-file -DgroupId=com.oracle -DartifactId=ojdbc14 -Dversion=11.1.0.7.0 -Dpackaging=jar -Dfile=ojdbc.jar`
**拷贝到mvn-repository**
`cp ~/app/repository/oracle/11.1.0.7.0/ojdbc14-11.1.0.7.0.jar`

### 1.2.3 创建相关配置文件

**web.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" version="3.0">

    <display-name>NWRDServer</display-name>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>


    <!-- Spring和Hibernate的配置文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/applicationContext*.xml</param-value>
    </context-param>
    <!--配置JVM环境变量来解决log4j文件输出位置不正确问题-->
    <context-param>
        <param-name>webAppRootKey</param-name>
        <param-value>webApp.root</param-value>
    </context-param>
    <context-param>
        <param-name>log4jConfigLocation</param-name>
        <param-value>classpath:log4j.properties</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
    </listener>

    <!-- Spring监听器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 防止Spring内存溢出监听器 -->
    <listener>
        <listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
    </listener>

    <!-- 配置SESSION超时，单位是分钟 -->
    <session-config>
        <session-timeout>15</session-timeout>
    </session-config>

    <!-- ############################################ filter start  ############################################ -->
    <!-- 编码过滤器 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <async-supported>true</async-supported>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!-- druid 数据源，用于采集 web-jdbc 关联监控的数据 -->
    <!-- 具体参考官网：https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_%E9%85%8D%E7%BD%AEWebStatFilter-->
    <filter>
        <filter-name>DruidWebStatFilter</filter-name>
        <filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
        <init-param>
            <param-name>exclusions</param-name>
            <param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
        </init-param>
        <init-param>
            <param-name>profileEnable</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>DruidWebStatFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>


    <!-- ############################################ filter end  ############################################ -->


    <!-- ############################################ servlet start  ############################################ -->


    <!-- Spring MVC servlet -->
    <servlet>
        <servlet-name>SpringMVC</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/spring-mvc.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringMVC</servlet-name>
        <!-- 此处可以可以配置成 *.do ，对应struts的后缀习惯 -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>/index.jsp</welcome-file>
    </welcome-file-list>


    <!--展示Druid的统计信息-->
    <!--具体可以看官网信息：https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatViewServlet%E9%85%8D%E7%BD%AE-->
    <servlet>
        <servlet-name>DruidStatView</servlet-name>
        <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DruidStatView</servlet-name>
        <!--访问路径eg：http://localhost:8080/druid/index.html -->
        <url-pattern>/druid/*</url-pattern>
    </servlet-mapping>


    <!-- ############################################ servlet end  ############################################ -->


</web-app>

```
**applicationContext.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
http://www.springframework.org/schema/context
http://www.springframework.org/schema/context/spring-context-4.3.xsd">

    <!--上面的xsd最好和当前使用的Spring版本号一致,如果换了Spring版本,这个最好也跟着改-->

    <!-- 引入属性文件 放在最开头   ,在使用spring之前就引入,里面的变量才能被引用-->
    <context:property-placeholder location="classpath*:properties/*.properties"/>
    <!--
    引入属性文件也可以用这种写法
    <bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="location" value="classpath:config.properties" />
    </bean>
    -->

    <!-- 自动扫描(需要自动注入的类，对于那些类上有注解：@Repository、@Service、@Controller、@Component都进行注入) -->
    <!--因为 Spring MVC 是 Spring 的子容器，所以我们在 Spring MVC 的配置中再指定扫描 Controller 的注解，这里是父容器的配置地方-->
    <!--注解注入的相关材料可以看：-->
    <!--http://blog.csdn.net/u012763117/article/details/17253849-->
    <!--http://casheen.iteye.com/blog/295348-->
    <!--http://blog.csdn.net/zhang854429783/article/details/6785574-->
    <context:component-scan base-package="cn.edu.ncut.nwrd.dao,cn.edu.ncut.nwrd.service,cn.edu.ncut.nwrd.common,cn.edu.ncut.nwrd.test"/>


</beans>


```

**applicationContext-dataSource.xml**

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:tx="http://www.springframework.org/schema/tx" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd
       http://www.springframework.org/schema/tx
       http://www.springframework.org/schema/tx/spring-tx-4.2.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 引入properties文件 -->
    <context:property-placeholder location="classpath*:/properties/config.properties"/>

    <!-- 引入jdbc配置文件 -->
    <bean id="propertyConfigurer"
          class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath*:/properties/config.properties</value>
            </list>
        </property>
    </bean>

    <!-- 持久层异常转换为Sping类型异常-->
    <bean id="persistenceExceptionTranslationPostProcessor" class="org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor"></bean>



    <!-- 使用阿里的druid配置数据源 start-->
    <!--具体查看官网信息：https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_DruidDataSource%E5%8F%82%E8%80%83%E9%85%8D%E7%BD%AE-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
    <!--这三个变量读取config.properties的-->
    <property name="driverClassName" value="oracle.jdbc.driver.OracleDriver"/>
    <property name="url"
              value="jdbc:oracle:thin:@${jdbc.host}:${jdbc.port}:${jdbc.database}"/>
    <property name="username" value="${jdbc.user}"/>
    <property name="password" value="${jdbc.password}"/>

    <!-- 初始化连接大小 -->
    <property name="initialSize" value="1"/>
    <!-- 初始化连接池最大使用连接数量 -->
    <property name="maxActive" value="20"/>
    <!-- 初始化连接池最小空闲 -->
    <property name="minIdle" value="1"/>
    <!-- 获取连接最大等待时间，单位毫秒-->
    <property name="maxWait" value="60000"/>
    <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
    <property name="timeBetweenEvictionRunsMillis" value="60000"/>
    <!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
    <property name="minEvictableIdleTimeMillis" value="25200000"/>
    <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
    <!--如果用Oracle，则把poolPreparedStatements配置为true，mysql可以配置为false。分库分表较多的数据库，建议配置为false。-->
    <property name="poolPreparedStatements" value="false"/>
    <property name="maxPoolPreparedStatementPerConnectionSize" value="20"/>
    <property name="validationQuery" value="${validation_query}"/>
    <property name="testWhileIdle" value="true"/>
    <property name="testOnBorrow" value="false"/>
    <property name="testOnReturn" value="false"/>
    <!--当程序存在缺陷时，申请的连接忘记关闭，这时候，就存在连接泄漏了。Druid提供了RemoveAbandanded相关配置，用来关闭长时间不使用的连接-->
    <!--配置removeAbandoned对性能会有一些影响，建议怀疑存在泄漏之后再打开。在上面的配置中，如果连接超过30分钟未关闭，就会被强行回收，并且日志记录连接申请时的调用堆栈。-->
    <!--具体查看官网信息：https://github.com/alibaba/druid/wiki/%E8%BF%9E%E6%8E%A5%E6%B3%84%E6%BC%8F%E7%9B%91%E6%B5%8B-->
    <!-- 打开removeAbandoned功能 -->
    <property name="removeAbandoned" value="true"/>
    <!-- 1800秒，也就是30分钟 -->
    <property name="removeAbandonedTimeout" value="1800"/>
    <!-- 关闭abanded连接时输出错误日志 -->
    <property name="logAbandoned" value="true"/>
    <!-- 配置监控统计拦截的filters-->
    <!--官网信息：https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_StatFilter-->
    <!--mergeSql可以合并输出的sql，方便查看，但是在mybatis框架中使用这个则无法监控sql,需要用stat-->
    <!--<property name="filters" value="mergeSql,log4j"/>-->
    <!--<property name="filters" value="mergeSql,wall"/>-->
    <!--<property name="filters" value="stat"/>-->
    <!--<property name="filters" value="mergeSql"/>-->
    <property name="filters" value="stat,log4j"/>
</bean>
    <!-- 使用阿里的druid配置数据源 end-->


    <bean id="sessionFactory"
          class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
        <property name="dataSource">
            <ref bean="dataSource"/>
        </property>
        <property name="hibernateProperties">
            <props>
                <prop key="hibernate.dialect">${jdbc.hibernate.dialect}</prop>
                <prop key="hibernate.show_sql">true</prop>
                <prop key="javax.persistence.validation.mode">none</prop>
            </props>
        </property>
        <property name="packagesToScan" value="cn.edu.ncut.nwrd.**.model.**"/>
    </bean>


    <bean id="txManager"
          class="org.springframework.orm.hibernate5.HibernateTransactionManager">
        <property name="sessionFactory" ref="sessionFactory"/>
    </bean>


    <!--使用基于注解方式配置事务 -->
    <tx:annotation-driven transaction-manager="txManager"></tx:annotation-driven>
</beans>
```

**applicationContext-transaction.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
http://www.springframework.org/schema/tx
http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
http://www.springframework.org/schema/aop
http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">

    <!--上面的xsd最好和当前使用的Spring版本号一致,如果换了Spring版本,这个最好也跟着改-->

    <!-- Druid 和 Spring 关联监控配置 start-->
    <!-- 具体可以查阅官网：https://github.com/alibaba/druid/wiki/%E9%85%8D%E7%BD%AE_Druid%E5%92%8CSpring%E5%85%B3%E8%81%94%E7%9B%91%E6%8E%A7%E9%85%8D%E7%BD%AE-->
    <bean id="druid-stat-interceptor" class="com.alibaba.druid.support.spring.stat.DruidStatInterceptor" />
    <bean id="druid-stat-pointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut" scope="prototype">
        <property name="patterns">
            <list>
                <value>cn.edu.ncut.service.*</value>
                <!--如果使用的是 hibernate 则这里也要扫描路径，但是 mybatis 不需要-->
                <!--<value>com.youmeek.ssm.module.*.dao.*</value>-->
            </list>
        </property>
    </bean>

    <aop:config proxy-target-class="true">
        <!-- pointcut-ref="druid-stat-pointcut" 这个报红没事-->
        <aop:advisor advice-ref="druid-stat-interceptor" pointcut-ref="druid-stat-pointcut" />
    </aop:config>
    <!-- Druid 和 Spring 关联监控配置 end-->


    <!-- (事务管理器)transaction manager, use JtaTransactionManager for global tx -->
    <!--http://www.mybatis.org/spring/zh/transactions.html-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
    </bean>


    <!-- 一种方式:注解方式配置事物,扫描@Transactional注解的类定义事务，配置上service实现类(下面还有一个方法名拦截方式,两个同时配置也是可以使用的，但是不建议两者一起使用) -->
    <!--<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>-->

    <!-- 一种方式:拦截器方式配置事物start 配置了该方式之后,在方法里面使用注解方式配置事务也是没有作用的 -->
    <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--以这些单词开头的方法自动加入事务-->
            <!--更多参数和使用方法可以参考:-->
            <!--http://wuhenjia.blog.163.com/blog/static/93469449201123010594395-->
            <!--http://baobao707.iteye.com/blog/415446-->
            <!--http://jinnianshilongnian.iteye.com/blog/1442376-->
            <!--http://winters1224.blog.51cto.com/3021203/807500-->
            <!--如果是方法中直接抛顶层Exception异常的话,propagation="REQUIRED"是不顶用的,还需要配置rollback-for属性-->


            <!--<tx:method name="delete*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.Exception" no-rollback-for="java.lang.RuntimeException"/>-->
            <!--<tx:method name="insert*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.RuntimeException" />-->
            <!--<tx:method name="update*" propagation="REQUIRED" read-only="false" rollback-for="java.lang.Exception" /> -->

            <tx:method name="add*" propagation="REQUIRED"/>
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="register*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="modify*" propagation="REQUIRED"/>
            <tx:method name="edit*" propagation="REQUIRED"/>
            <tx:method name="batch*"  propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="remove*" propagation="REQUIRED"/>
            <tx:method name="time*" propagation="REQUIRED"/><!--定时器方法-->
            <tx:method name="repair" propagation="REQUIRED"/>
            <tx:method name="deleteAndRepair" propagation="REQUIRED"/>

            <!--以这些单词开头的方法不加入事务-->
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="load*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="search*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="datagrid*" propagation="SUPPORTS" read-only="true"/>

            <tx:method name="*" propagation="SUPPORTS"/>
        </tx:attributes>
    </tx:advice>


    <aop:config>
        <!--把这个拦截器配置到com.youmeek.ssh.service下(包括子包)下的以impl目录下类的,任意方法-->
        <!--
        execution的语法表示:在impl包中定义的任意方法的执行，更多方式可以参考：
        http://lavasoft.blog.51cto.com/62575/172292/
        http://blog.csdn.net/partner4java/article/details/7015946
        -->
        <aop:pointcut id="transactionPointcut" expression="execution(* cn.edu.ncut.service.impl.*.*(..) )"/>
        <!--<aop:pointcut id="transactionPointcut" expression="execution(* *..*Service*.*(..))"/>-->
        <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice"/>
    </aop:config>
    <!--一种方式:拦截器方式配置事物end-->

</beans>

```

**spring-mvc.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-4.3.xsd http://www.springframework.org/schema/mvc
                        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!-- 自动扫描该包，使SpringMVC认为包下用了@controller注解的类是控制器 -->
    <context:component-scan base-package="cn.edu.ncut.nwrd.controller"/>

    <!-- 配置注解驱动 -->
    <mvc:annotation-driven/>

    <!--静态资源映射-->
    <!--
    http://perfy315.iteye.com/blog/2008763
    mapping="/js/**"
    表示当浏览器有静态资源请求的时候，并且请求url路径中带有：/js/，则这个资源跑到webapp目录下的/WEB-INF/statics/js/去找
    比如我们在 JSP 中引入一个 js 文件：src="${webRoot}/js/jQuery-core/jquery-1.6.1.min.js
    -->
    <!--<mvc:resources mapping="/css/**" location="/WEB-INF/statics/css/"/>-->
    <!--<mvc:resources mapping="/js/**" location="/WEB-INF/statics/js/"/>-->
    <!--<mvc:resources mapping="/images/**" location="/WEB-INF/statics/images/"/>-->


    <!-- 对模型视图名称的解析，即在模型视图名称添加前后缀(如果最后一个还是表示文件夹,则最后的斜杠不要漏了) 使用JSP-->
    <!-- 默认的视图解析器 在上边的解析错误时使用 (默认使用html)- -->
    <bean id="defaultViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/view/"/><!--设置JSP文件的目录位置-->
        <property name="suffix" value=".jsp"/>
    </bean>


    <!-- 文件上传 start 配置文件上传，如果没有使用文件上传可以不用配置，当然如果不配，那么配置文件中也不必引入上传组件包 -->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!-- 默认编码 -->
        <property name="defaultEncoding" value="UTF-8"/>
        <!-- 文件大小最大值 上传文件大小限制为10M，10*1024*1024 -->
        <property name="maxUploadSize" value="10485760"/>
        <!-- 内存中的最大值 -->
        <property name="maxInMemorySize" value="4096"/>
    </bean>
    <!--文件上传 end-->
</beans>
```

**log4j.properties**

```
#本属性文件只能放在 resources 根目录下
#同时使用两种记录,一种控制台,一种文件方式（文件大小到达指定尺寸的时候产生一个新的文件）
#log4j.rootLogger=trace,appenderNameConsole,appenderNameRollingFile
#只输出到控制台，不输出到文件，级别：all > trace > debug > info > warn > error
#log4j.rootLogger=info,appenderNameConsole,appenderNameRollingFile
log4j.rootLogger=info,appenderNameConsole
#控制台输出
log4j.appender.appenderNameConsole=org.apache.log4j.ConsoleAppender
log4j.appender.appenderNameConsole.Target=System.out
log4j.appender.appenderNameConsole.layout=org.apache.log4j.PatternLayout
log4j.appender.appenderNameConsole.layout.ConversionPattern=[%d{yyyy-MM-dd HH:mm:ss}] --- [%p] --- [%F:%L] --- [%m] --- %n
#这个用来输出mybatis执行sql语句.其中 com.youmeek.ssm.manage.mapper 表示mapper.xml中的namespace,这里只是前缀表示所有这个前缀下的都输出,也可以写完整namespace.
#======================================================
#输出日志到硬盘，文件大小到达指定尺寸的时候产生一个新的文件
#log4j.appender.appenderNameRollingFile=org.apache.log4j.RollingFileAppender
#log4j.appender.appenderNameRollingFile.File=${webApp.root}/WEB-INF/logs/dmes.log
#log4j.appender.appenderNameRollingFile.MaxFileSize=50MB
#log4j.appender.appenderNameRollingFile.Threshold=ALL
#log4j.appender.appenderNameRollingFile.layout=org.apache.log4j.PatternLayout
#log4j.appender.appenderNameRollingFile.layout.ConversionPattern=[%d{yyyy-MM-dd HH:mm:ss}] --- [%p] --- [%F:%L] --- [%m] --- %n



```

**config.properties**

```
jdbc.host=马赛克
jdbc.port=马赛克
jdbc.database=马赛克
jdbc.user=马赛克
jdbc.password=马赛克
jdbc.hibernate.dialect=org.hibernate.dialect.Oracle10gDialect
validation_query=SELECT 1 FROM DUAL

```

## 1.3 创建数据库表


```sql

  CREATE TABLE "NWRD"."TB_USER" 
   (	"ID" NUMBER(10,0) NOT NULL ENABLE, 
	"NAME" VARCHAR2(200 BYTE), 
	"BIRTHDAY" DATE, 
	"CREATE_TIME" TIMESTAMP (6) 
```

## 1.4 创建MVC模型

### 1.4.1 使用Idea的Hibernate逆向工程

- 右键项目 Add Frameworks Support 勾选Hibernate
- ProjectStructure -> Facets -> 添加HibernateConfiguration
![](14985742174970.jpg)

- View -> Tool Windows -> Persistence
![](14985745107431.jpg)

- 生成Model

![](14985745611732.jpg)

注意绿色部位 需要手动修改为 java.util.Date

Idea的数据库客户端插件值得推荐使用

![](14985749908685.jpg)


### 1.4.2 Model

**修改Model的表名、添加主键序列绑定、添加toStirng方法**

```java
package cn.edu.ncut.nwrd.model;

import cn.edu.ncut.nwrd.model.base.BaseModel;

import javax.persistence.*;
import java.sql.Timestamp;
import java.util.Date;

/**
 * @author NikoBelic
 * @create 2017/6/27 18:16
 */
@Entity
@Table(name = "TB_USER", schema = "NWRD", catalog = "")
public class UserObj extends BaseModel
{
    private Integer id;
    private String name;
    private Date birthday;
    private Timestamp createTime;

    @Id
    @Column(name = "ID")
    @SequenceGenerator(name = "TB_USER_PK_GENERATOR", sequenceName = "SEQ_USER",allocationSize=1)
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "TB_USER_PK_GENERATOR")
    public Integer getId()
    {
        return id;
    }

    public void setId(Integer id)
    {
        this.id = id;
    }

    @Basic
    @Column(name = "NAME")
    public String getName()
    {
        return name;
    }

    public void setName(String name)
    {
        this.name = name;
    }

    @Basic
    @Column(name = "BIRTHDAY")
    public Date getBirthday()
    {
        return birthday;
    }

    public void setBirthday(Date birthday)
    {
        this.birthday = birthday;
    }

    @Basic
    @Column(name = "CREATE_TIME")
    public Timestamp getCreateTime()
    {
        return createTime;
    }

    public void setCreateTime(Timestamp createTime)
    {
        this.createTime = createTime;
    }

    @Override
    public boolean equals(Object o)
    {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;

        UserObj userObj = (UserObj) o;

        if (id != null ? !id.equals(userObj.id) : userObj.id != null)
            return false;
        if (name != null ? !name.equals(userObj.name) : userObj.name != null)
            return false;
        if (birthday != null ? !birthday.equals(userObj.birthday) : userObj.birthday != null)
            return false;
        if (createTime != null ? !createTime.equals(userObj.createTime) : userObj.createTime != null)
            return false;

        return true;
    }

    @Override
    public int hashCode()
    {
        int result = id != null ? id.hashCode() : 0;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        result = 31 * result + (birthday != null ? birthday.hashCode() : 0);
        result = 31 * result + (createTime != null ? createTime.hashCode() : 0);
        return result;
    }

    @Override
    public String toString()
    {
        return "UserObj{" + "id=" + id + ", name='" + name + '\'' + ", birthday=" + birthday + ", createTime=" + createTime + '}';
    }
}

```

**BaseModel**


```java
package cn.edu.ncut.nwrd.model.base;

import org.apache.log4j.Logger;

import java.io.Serializable;

public abstract class BaseModel implements Serializable
{
    private static final long serialVersionUID = 2387253001746505718L;

    private final static Logger logger = Logger.getLogger(BaseModel.class);
    // date formats
    protected static final String DATE_FORMAT = "yyyy-MM-dd";

    protected static final String TIME_FORMAT = "HH:mm:ss";

    protected static final String DATE_TIME_FORMAT = "yyyy-MM-dd'T'HH:mm:ss";

    protected static final String TIMESTAMP_FORMAT = "yyyy-MM-dd HH:mm:ss.S";


}

```

### 1.4.3 Dao


**BaseDao**


```java
package cn.edu.ncut.nwrd.common.basedao;

import java.io.Serializable;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

public interface Dao<T extends Serializable> {
	T find(Serializable id);

	void save(T entity);

	void update(T entity);

	void merge(T entity);

	/**
	 * 更新时首先进行查询，实体应包含主键，实验于不把字段改为NULL的更新
	 * 
	 * @param entity
	 */
    void updateWithQuery(T entity);

	void delete(Serializable id);

	long getCount();

	long getCount(String where, Map<String, Object> params);

	/**
	 * 根据条件查询，获得所有记录
	 */
    List<T> findAll();

	/**
	 * 根据条件查询，获得记录集
	 * 
	 * @param where
	 *            查询条件语句
	 * @param params
	 *            查询条件参数
	 * @param orderby
	 *            排序属性 value asc/desc
	 */
    List<T> findAll(String where, Map<String, ?> params, LinkedHashMap<String, String> orderby);

	/**
	 * 根据条件查询，获得记录集 查询时对数据加行锁
	 *
	 * @param where
	 *            查询条件语句
	 * @param params
	 *            查询条件参数
	 * @param orderby
	 *            排序属性 value asc/desc
	 */
    List<T> findAllWithLock(String where, Map<String, ?> params, LinkedHashMap<String, String> orderby);
	/**
	 * 根据条件查询，获得记录集
	 * 
	 * @param firstResult
	 *            第一条记录号,为-1时不进行筛选
	 * @param maxResult
	 *            返回记录最大数,为-1时不进行筛选
	 * @param where
	 *            查询条件语句
	 * @param params
	 *            查询条件参数
	 * @param orderby
	 *            排序属性 value asc/desc
	 */
    List<T> findAll(int firstResult, int maxResult, String where, Map<String, Object> params, String orderby);

	/**
	 * 根据条件查询，获得记录集,只查询fields的字段
	 * 
	 * @param firstResult
	 *            第一条记录号,为-1时不进行筛选
	 * @param maxResult
	 *            返回记录最大数,为-1时不进行筛选
	 * @param where
	 *            查询条件语句
	 * @param params
	 *            查询条件参数
	 * @param orderby
	 *            order by 属性 asc/desc
	 * @param fields
	 *            需要查询的字段
	 */
    QueryResult<Map<String, Object>> getScrollData(int firstResult, int maxResult, String where, Map<String, Object> params, String orderby, String[] fields);

	/**
	 * 根据条件查询，获得记录集
	 * 
	 * @param firstResult
	 *            第一条记录号,为-1时不进行筛选
	 * @param maxResult
	 *            返回记录最大数,为-1时不进行筛选
	 * @param where
	 *            查询条件语句
	 * @param params
	 *            查询条件参数
	 * @param orderby
	 *            order by 属性 asc/desc
	 */
    QueryResult<T> getScrollData(int firstResult, int maxResult, String where, Map<String, Object> params, String orderby);

	/**
	 * 根据条件查询，获得记录集
	 * 
	 * @param firstResult
	 *            第一条记录号,为-1时不进行筛选
	 * @param maxResult
	 *            返回记录最大数,为-1时不进行筛选
	 * @param where
	 *            查询条件语句
	 * @param params
	 *            查询条件参数
	 * @param orderby
	 *            排序属性 value asc/desc
	 */
    QueryResult<T> getScrollData(int firstResult, int maxResult, String where, Map<String, Object> params, LinkedHashMap<String, String> orderby);

	/**
	 * 根据条件查询，获得记录集
	 * 
	 * @param firstResult
	 *            第一条记录号
	 * @param maxResult
	 *            返回记录最大数
	 * @param orderby
	 *            排序属性 value asc/desc
	 */
    QueryResult<T> getScrollData(int firstResult, int maxResult, LinkedHashMap<String, String> orderby);

	/**
	 * 根据条件查询，获得记录集
	 * 
	 * @param where
	 *            查询条件语句
	 * @param params
	 *            查询条件参数
	 */
    QueryResult<T> getScrollData(String where, Map<String, Object> params);

	QueryResult<T> getScrollData();

}

```

**DaoSupport**


```java
package cn.edu.ncut.nwrd.common.basedao;

import org.hibernate.LockMode;
import org.hibernate.Query;
import org.hibernate.SessionFactory;
import org.springframework.transaction.annotation.Transactional;

import javax.annotation.Resource;
import javax.persistence.Id;
import java.io.Serializable;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

@Transactional
public abstract class DaoSupport<T extends Serializable> implements Dao<T> {

	@Resource
	protected SessionFactory sessionFactory;
	protected Class<?> entityClass;
	protected String entityName;
	protected Method getIdMethod;

	public DaoSupport() {
		if (!"BaseDaoImpl".equals(this.getClass().getSimpleName())) {
			entityClass = getEntityClass();
			entityName = getEntityName();
			getIdMethod = findGetIdMethod();
		}
	}

	@SuppressWarnings("unchecked")
	public Class<T> getEntityClass() {
		Type parentType = getClass().getGenericSuperclass();
		if (parentType instanceof ParameterizedType) {
			ParameterizedType ptype = (ParameterizedType) parentType;
			return (Class<T>) ptype.getActualTypeArguments()[0];
		}
		return null;
	}

	public String getEntityName() {
		return this.entityClass.getSimpleName();
	}

	@SuppressWarnings("unchecked")
	@Transactional(readOnly = true)
	public T find(Serializable id) {
		return (T) sessionFactory.getCurrentSession().get(entityClass, id);
	}

	public void save(T entity) {
		sessionFactory.getCurrentSession().persist(entity);
	}

	public void update(T entity) {
		sessionFactory.getCurrentSession().update(entity);
	}
	
	public void merge(T entity) {
		sessionFactory.getCurrentSession().merge(entity);
	}

	public void updateWithQuery(T entity) {
		try {
			Serializable id = (Serializable) getIdMethod.invoke(entity);
			T entitycur = find(id);

			Field[] fields = entityClass.getDeclaredFields();
			for (Field field : fields) {
				String filedName = field.getName();
				String ufilename = filedName.substring(0, 1).toUpperCase()
						+ filedName.substring(1);
				String getfiledMethodName = "get" + ufilename;
				String setfiledMethodName = "set" + ufilename;
				try {
					Method getMethod = entityClass
							.getMethod(getfiledMethodName);
					Object f = getMethod.invoke(entity);
					if (f != null) {
						Class<?> type = getMethod.getReturnType();
						Method setMethod = entityClass.getMethod(
								setfiledMethodName, type);
						setMethod.invoke(entitycur, f);
					}
				} catch (NoSuchMethodException e) {
				}

			}
			this.update(entitycur);
		} catch (Exception e) {
			e.printStackTrace();
		}

	}

	public void delete(Serializable id) {
		sessionFactory.getCurrentSession().delete(
				sessionFactory.getCurrentSession().load(entityClass, id));
	}

	@Transactional(readOnly = true)
	public long getCount() {
		return (Long) sessionFactory.getCurrentSession()
				.createQuery("select count(o) from " + entityName + " o")
				.iterate().next();
	}

	@Transactional(readOnly = true)
	public long getCount(String where, Map<String, Object> params) {
		String whereql = where != null && !"".equals(where.trim()) ? " where "
				+ where : "";
		Query query = sessionFactory.getCurrentSession().createQuery(
				"select count(o) from " + entityName + " o " + whereql);
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		return (Long) query.iterate().next();
	}

	@SuppressWarnings("unchecked")
	@Transactional(readOnly = true)
	public List<T> findAll(String where, Map<String, ?> params,
			LinkedHashMap<String, String> orderby) {
		String whereql = where != null && !"".equals(where.trim()) ? " where "
				+ where : "";
		Query query = sessionFactory.getCurrentSession().createQuery(
				"from " + entityName + " o " + whereql + buildOrderby(orderby));
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		return query.list();
	}
	
	@SuppressWarnings("unchecked")
	@Transactional(readOnly = true)
	public List<T> findAll(int firstResult,int maxResult,String where, Map<String, Object> params,
			String orderby) {
		String whereql = where != null && !"".equals(where.trim()) ? " where "
				+ where : "";
		orderby = orderby == null ? "" : orderby;
		Query query = sessionFactory.getCurrentSession().createQuery(
				"from " + entityName + " o " + whereql + orderby);
		if (firstResult != -1 && maxResult != -1)
			query.setFirstResult(firstResult).setMaxResults(maxResult);
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		return query.list();
	}

	@SuppressWarnings("unchecked")
	@Transactional(readOnly = true)
	public List<T> findAllWithLock(String where, Map<String, ?> params, LinkedHashMap<String, String> orderby)
	{
		String whereql = where != null && !"".equals(where.trim()) ? " where "
				+ where : "";
		Query query = sessionFactory.getCurrentSession().createQuery(
				"from " + entityName + " o " + whereql + buildOrderby(orderby)).setLockMode("o", LockMode.UPGRADE);
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		return query.list();
	}



	public List<T> findAll() {
		return findAll(null, null, null);
	}

	@SuppressWarnings("unchecked")
	@Transactional(readOnly = true)
	public QueryResult<T> getScrollData(int firstResult, int maxResult,
			String where, Map<String, Object> params, String orderby) {
		String whereql = where != null && !"".equals(where.trim()) ? " where "
				+ where : "";
		orderby = orderby == null ? "" : orderby;
		Query query = sessionFactory.getCurrentSession().createQuery(
				"from " + entityName + " o " + whereql + orderby);
		if (firstResult != -1 && maxResult != -1)
			query.setFirstResult(firstResult).setMaxResults(maxResult);
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		QueryResult<T> qr = new QueryResult<T>();
		qr.setResultlist(query.list());
		query = sessionFactory.getCurrentSession().createQuery(
				"select count(o) from " + entityName + " o " + whereql);
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		qr.setRecordtotal((Long) query.iterate().next());
		return qr;
	}

	@SuppressWarnings("unchecked")
	@Transactional(readOnly = true)
	public QueryResult<Map<String, Object>> getScrollData(int firstResult,
			int maxResult, String where, Map<String, Object> params,
			String orderby, String[] fields) {
		String whereql = where != null && !"".equals(where.trim()) ? " where "
				+ where : "";
		String strFields = buildFields(fields);
		orderby = orderby == null ? "" : orderby;
		Query query = sessionFactory.getCurrentSession().createQuery(
				"select " + strFields + " from " + entityName + " o " + whereql
						+ orderby);
		if (firstResult != -1 && maxResult != -1)
			query.setFirstResult(firstResult).setMaxResults(maxResult);
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		QueryResult<Map<String, Object>> qr = new QueryResult<Map<String, Object>>();
		qr.setResultlist(query.list());
		query = sessionFactory.getCurrentSession().createQuery(
				"select count(o) from " + entityName + " o " + whereql);
		if (where != null && !"".equals(where.trim())) {
			setParameter(query, params);
		}
		qr.setRecordtotal((Long) query.iterate().next());
		return qr;
	}

	public QueryResult<T> getScrollData(int firstResult, int maxResult,
			String where, Map<String, Object> params,
			LinkedHashMap<String, String> orderby) {
		return getScrollData(firstResult, maxResult, where, params,
				buildOrderby(orderby));
	}

	public QueryResult<T> getScrollData(String where, Map<String, Object> params) {
		return getScrollData(-1, -1, where, params, "");
	}

	public QueryResult<T> getScrollData(int firstResult, int maxResult,
			LinkedHashMap<String, String> orderby) {
		return getScrollData(firstResult, maxResult, "", null, orderby);
	}
	
	/**
	 * 获得记录集
	 */
	public QueryResult<T> getScrollData() {
		return getScrollData(-1, -1, "", null, "");
	}

	/**

	/**
	 * 构建排序语句
	 * 
	 * @param orderby
	 *            key排序属性 value asc/desc
	 * @return
	 */
	protected static String buildOrderby(LinkedHashMap<String, String> orderby) {
		StringBuilder sb = new StringBuilder();
		if (orderby != null && !orderby.isEmpty()) {
			sb.append(" order by ");
			for (Map.Entry<String, String> entry : orderby.entrySet()) {
				sb.append("o.").append(entry.getKey()).append(" ")
						.append(entry.getValue()).append(",");
			}
			sb.deleteCharAt(sb.length() - 1);
		}
		return sb.toString();
	}

	protected static void setParameter(Query query, Map<String, ?> params) {
		if (params == null || params.size() == 0)
			return;
		query.setProperties(params);
	}

	protected static String buildFields(String[] fields) {
		StringBuilder sb = new StringBuilder("new map(");
		for (String field : fields) {
			if (field.lastIndexOf(".") >= 0)
				sb.append(field.toLowerCase()).append(" as ")
						.append(field.substring(field.lastIndexOf(".")+1))
						.append(",");
			else
				sb.append(field.toLowerCase()).append(" as ").append(field)
						.append(",");
		}
		sb.deleteCharAt(sb.length() - 1);
		sb.append(")");
		return sb.toString();
	}

	protected Method findGetIdMethod() {
		Method[] methods = entityClass.getMethods();
		for (Method method : methods) {
			if (method.isAnnotationPresent(Id.class)) {
				return method;
			}
		}
		return null;
	}
	
	
}

```

**QueryResult**


```java
package cn.edu.ncut.nwrd.common.basedao;

import org.apache.log4j.Logger;
import org.dom4j.Element;

import java.lang.reflect.Method;
import java.util.List;
import java.util.Map;

public class QueryResult<T> {
	
	private final static Logger logger = Logger
			.getLogger(QueryResult.class);
	
	private List<T> resultlist;
	private long recordtotal;

	public void setResultlist(List<T> resultlist) {
		this.resultlist = resultlist;
	}

	public List<T> getResultlist() {
		return resultlist;
	}

	public void setRecordtotal(long recordtotal) {
		this.recordtotal = recordtotal;
	}

	public long getRecordtotal() {
		return recordtotal;
	}


	@SuppressWarnings("unchecked")
	private void addData(String[] fields,
                         Map<String, Object> fieldMap, Element element, T record)
			throws Exception {
		Class<T> cla = (Class<T>) resultlist.get(0).getClass();

		for (String field : fields) {
			Object fieldItem = null;
			if (fieldMap != null) {
				fieldItem = fieldMap.get(field);
				if (fieldItem != null
						&& String.class.isAssignableFrom(fieldItem
								.getClass())) {
					if ("hide".equals(fieldItem))
						continue;
				}
			}
			
			Object obj;
			if (Map.class.isAssignableFrom(cla)) {
				if(field.lastIndexOf(".")>=0)
					field=field.substring(field.lastIndexOf(".")+1);
				obj = ((Map<String, Object>) record).get(field);
			} else {
				String methodName = "get" + field.substring(0, 1).toUpperCase()
						+ field.substring(1).toLowerCase();
				Method m = cla.getMethod(methodName);
				obj = m.invoke(record);
			}

			String text = buildData(field, fieldItem, obj, record);

			element.addElement(field).addText(text);
		}
	}
	
	@SuppressWarnings("unchecked")
	private String buildData(String field,
			Object fieldItem, Object data, T record) {
		if(field.lastIndexOf(".")>=0)
			field=field.substring(field.lastIndexOf(".")+1);
		if (data == null) return "";
		String text = "";
		if (fieldItem != null){
			if(Map.class.isAssignableFrom(fieldItem.getClass())){
				Map<Object, String> map = (Map<Object, String>) fieldItem;
				text = map.get(data);
			}else if((Method.class.isAssignableFrom(fieldItem.getClass()))){
				Method method = (Method) fieldItem;
				try {
					text = (String) method.invoke(null, data, record);
				} catch (Exception e) {
					logger.error(e);
				}
			}
		} else {
			text = data.toString();
		}
		return text;
	}
}

```

```java
package cn.edu.ncut.nwrd.dao;

import cn.edu.ncut.nwrd.common.basedao.Dao;
import cn.edu.ncut.nwrd.model.UserObj;
public interface UserDao extends Dao<UserObj>
{

}

```


```java
package cn.edu.ncut.nwrd.dao;

import cn.edu.ncut.nwrd.common.basedao.DaoSupport;
import cn.edu.ncut.nwrd.model.UserObj;
import org.apache.log4j.Logger;
import org.springframework.stereotype.Repository;

/**
 * @author NikoBelic
 * @create 2017/6/27 15:12
 */
@Repository("userDao")
public class UserDaoImpl extends DaoSupport<UserObj> implements UserDao
{
    private final static Logger logger = Logger.getLogger(UserDaoImpl.class);
}

```

### 1.4.4 Service


```java
package cn.edu.ncut.nwrd.service;

import cn.edu.ncut.nwrd.model.UserObj;

import java.util.List;

/**
 * @author NikoBelic
 * @create 2017/6/27 21:56
 */
public interface UserService
{
    List<UserObj> getAllUser();
}

```


```java
package cn.edu.ncut.nwrd.service;

import cn.edu.ncut.nwrd.dao.UserDao;
import cn.edu.ncut.nwrd.model.UserObj;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author NikoBelic
 * @create 2017/6/27 21:57
 */
@Service
public class UserServiceImpl implements UserService
{
    @Autowired
    private UserDao userDao;


    @Override
    public List<UserObj> getAllUser()
    {
        List<UserObj> userList = userDao.findAll();
        return userList;
    }
}

```

### 1.4.5 Controller

```java
package cn.edu.ncut.nwrd.controller;

import cn.edu.ncut.nwrd.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.servlet.ModelAndView;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;

/**
 * @author NikoBelic
 * @create 2017/6/27 21:59
 */
@Controller
@RequestMapping("/user")
public class UserController
{
    @Autowired
    private UserService userService;

    @RequestMapping(value = "/getAllUser")
    public Object getAllUser(HttpServletRequest request, HttpServletResponse response)
    {
        HashMap<String, Object> result = new HashMap<>();
        result.put("userList", userService.getAllUser());
        return new ModelAndView("/user/list", result);
    }
}

```

### 1.4.6 视图


```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
         pageEncoding="UTF-8"%>
<%@ include file="../commons/taglibs.jsp" %>
<%@ taglib tagdir="/WEB-INF/tags/simpletable" prefix="simpletable"%>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>用户列表</title>
</head>
<body>

<table>
    <thead>
        <tr>
            <td>编号</td>
            <td>用户名</td>
            <td>出生日期</td>
    </thead>
    <tbody>
        <c:forEach items="${userList}" var="user">
            <tr>
                <td>${user.id}</td>
                <td>${user.name}</td>
                <td>${user.birthday}</td>
            </tr>
        </c:forEach>
    </tbody>
</table>
</body>
</html>

```
### 1.4.7 渲染结果
![](14985748203266.jpg)


### 1.4.8 测试Spring+Hibernate


```java
package cn.edu.ncut.nwrd.test;

import cn.edu.ncut.nwrd.dao.LogDao;
import cn.edu.ncut.nwrd.dao.UserDao;
import cn.edu.ncut.nwrd.model.LogObj;
import cn.edu.ncut.nwrd.model.UserObj;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import javax.annotation.Resource;
import java.sql.Timestamp;
import java.util.Date;
import java.util.List;

/**
 * @author NikoBelic
 * @create 2017/6/27 15:13
 */
@ContextConfiguration(locations = "classpath:spring/*.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class TestSSH
{
    @Resource
    private UserDao userDao;

    @Resource
    private LogDao logDao;

    @Test
    public void testAddUser()
    {
        testGetAll();
        UserObj user = new UserObj();
        user.setName("YJM");
        user.setBirthday(new Date());
        user.setCreateTime(new Timestamp(System.currentTimeMillis() / 1000));
        userDao.save(user);

        testGetAll();
    }

    @Test
    public void testGetUser()
    {
        UserObj userObj = userDao.find(5);
        System.out.println("userObj = " + userObj);
    }

    @Test
    public void testUpdateUser()
    {
        testGetAll();
        UserObj userObj = userDao.find(5);
        if (userObj != null)
        {
            userObj.setName("新全国作品登记");
            userDao.update(userObj);
        }
        testGetAll();
    }

    @Test
    public void testDeleteUser()
    {
        testGetAll();
        userDao.delete(3);
        testGetAll();
    }

    @Test
    public void testGetAll()
    {
        System.out.println("查询");
        List<UserObj> userList = userDao.findAll();
        for (UserObj userDO : userList)
        {
            System.out.println("userObj = " + userDO);
        }
    }

    @Test
    public void testAddLog()
    {
        LogObj log = new LogObj();
        log.setMsg("测试日志");
        logDao.save(log);
    }

}

```

# 2 Oracle数据库实例搭建

- Server创建实例
- Client连接实例
    - 创建临时表空间、表空间
    - 创建用户
    - 给用户分配表空间
- 测试连接


# 3 问题总结

1. Oracle11g区分大小写，创建表和字段时我使用的是小写，但是Hibernate访问Oracle时会将SQL中的表名、字段名转换为大写，导致出现 “未找到表或视图”
    
解决：将Oracle表名、字段名都改为大写

2. Maven 无法引入 Oracle驱动问题。上面已经说了解决方法。

3. 创建包文件夹时，不小心创建成了 python package 导致import后面的包名都标红，查了半天才发现是创建错了包类型。

