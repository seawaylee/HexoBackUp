---
title: SpringInAction读书笔记(九) Spring Data 
date: 2017-03-09 23:49:13
tags: [JavaWeb,Spring]
---

本章将介绍如何使用Spring的JDBCTemplate、如何使用SpringData整合MongoDB、如何使用SpringData在运行时自动生成Repository。

<!--more-->

## 10.1 配置文件类

**全局配置**

```java
package data_persistent.config;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

/**
 * @author NikoBelic
 * @create 22/01/2017 15:46
 */
public class WebInitializer extends AbstractAnnotationConfigDispatcherServletInitializer
{
    public WebInitializer()
    {
        System.out.println("全局配置文件初始化");
    }

    /**
     * 指定ContextLoaderListener配置类
     * @Author NikoBelic
     * @Date 30/12/2016 10:48
     */
    @Override
    protected Class<?>[] getRootConfigClasses()
    {
        return new Class<?>[]{RootConfig.class};
    }


    /**
     * 指定SpringMVC配置类
     * @Author NikoBelic
     * @Date 30/12/2016 10:49
     */
    @Override
    protected Class<?>[] getServletConfigClasses()
    {
        return new Class<?>[]{WebConfig.class};
    }

    /**
     * 将DispatcherServlet映射到 "/"
     * @Author
     * @Date 30/12/2016 10:44
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}

```

**Spring上下文配置、数据源配置**

```java
package data_persistent.config;

import com.mongodb.Mongo;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.data.mongodb.core.MongoFactoryBean;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.repository.config.EnableMongoRepositories;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

import javax.sql.DataSource;

/**
 * Spring上下文配置类
 * 由ContextLoaderListener加载的bean
 * @author NikoBelic
 * @create 09/01/2017 20:30
 */
@ComponentScan(basePackages = "data_persistent",excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION,value = EnableWebMvc.class)})
@Configuration
@EnableMongoRepositories("data_persistent.mongodao") // 开启SpringData自动生成MongoRepository功能
public class RootConfig
{
    public RootConfig()
    {
        System.out.println("RootConfig初始化");
    }

    /**
     * 注入Mysql数据源
     * @Author NikoBelic
     * @Date 23/01/2017 11:31
     */
    @Bean
    public DataSource dataSource()
    {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost:3306/dmes?characterEncoding=UTF-8");
        ds.setUsername("root");
        ds.setPassword("lxw1993822");
        return ds;
    }

    /**
     * 注入JdbcTemplate
     * @Author NikoBelic
     * @Date 23/01/2017 11:30
     */
    @Bean
    public JdbcTemplate jdbcTemplate(DataSource dataSource)
    {
        return new JdbcTemplate(dataSource);
    }

    /**
     * 注入MongoFactory
     * @Author NikoBelic
     * @Date 23/01/2017 11:30
     */
    @Bean
    public MongoFactoryBean mongo()
    {
        MongoFactoryBean mongo = new MongoFactoryBean();
        mongo.setHost("localhost");
        return mongo;
    }
    /**
     * 注入MongoTempldate
     * MongoOperations是一个借口,MongoTemplate实现了它
     * @Author NikoBelic
     * @Date 23/01/2017 11:30
     */
    @Bean
    public MongoOperations  mongoTemplate(Mongo mongo)
    {
        return new MongoTemplate(mongo,"spring_in_action");
    }
}

```

**SpringMVC配置(未使用)**


```java
package data_persistent.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.multipart.MultipartResolver;
import org.springframework.web.multipart.support.StandardServletMultipartResolver;
import org.springframework.web.servlet.ViewResolver;
import org.springframework.web.servlet.config.annotation.DefaultServletHandlerConfigurer;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;

import java.io.IOException;

/**
 * @author NikoBelic
 * @create 09/01/2017 20:35
 */
@EnableWebMvc
@ComponentScan("data_persistent.controller")
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter
{
    public WebConfig()
    {
        System.out.println("SpringMVC配置初始化");
    }

    /**
     * 配置视图解析器
     * @Author NikoBelic
     * @Date 09/01/2017 20:37
     */
    @Bean
    public ViewResolver viewResolver()
    {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        resolver.setExposeContextBeansAsAttributes(true);
        resolver.setViewClass(JstlView.class);
        return resolver;
    }

    /**
     * 配置静态资源的处理
     * @Author NikoBelic
     * @Date 09/01/2017 20:37
     */
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    /**
     * 配置文件上传解析器
     * @Author NikoBelic
     * @Date 09/01/2017 20:38
     */
    @Bean
    public MultipartResolver multipartResolver() throws IOException {
        return new StandardServletMultipartResolver();
    }

}
```

## 10.2 Sprng + JdbcTemplate

**Model**


```java
package data_persistent.model;

/**
 * @author NikoBelic
 * @create 22/01/2017 15:58
 */
public class UserObj
{
    private Integer id;
    private String username;
    private String password;
    private String role;

    public UserObj() {
    }

    public UserObj(int id, String username, String password) {
        this.id = id;
        this.username = username;
        this.password = password;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getRole() {
        return role;
    }

    public void setRole(String role) {
        this.role = role;
    }

    @Override
    public String toString() {
        return "UserObj{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", role='" + role + '\'' +
                '}';
    }
}

```

**Dao**


```java
package data_persistent.dao;

import data_persistent.model.UserObj;

/**
 * @author NikoBelic
 * @create 22/01/2017 15:59
 */
public interface UserDao
{
    void addUser();
    UserObj findUserById(Integer id);
}

```

**DaoImpl**


```java
package data_persistent.dao;

import data_persistent.model.UserObj;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcOperations;
import org.springframework.stereotype.Repository;

import javax.inject.Inject;

/**
 * @author NikoBelic
 * @create 22/01/2017 16:01
 */
@Repository
public class UserDaoImpl implements UserDao
{
    @Autowired
    private JdbcOperations jdbcOperations;

    /**
     * 使用JDBCTemplate添加数据
     * @Author NikoBelic
     * @Date 22/01/2017 16:25
     */
    public void addUser()
    {
        UserObj user = new UserObj();
        user.setUsername("NikoBelic");
        user.setPassword("123456");
        user.setRole("ROLE_UNKNOWN");
        jdbcOperations.update("insert into tb_user(username,password,role) values(?,?,?)",
                user.getUsername(), user.getPassword(), user.getRole());
    }
    public UserObj findUserById(Integer id)
    {
        return jdbcOperations.queryForObject("select * from tb_user t where t.id = ?" ,
                (rs,rowNum) ->{
                    return new UserObj(rs.getInt("id"),rs.getString("username"),rs.getString("password"));
                },id);
    }


}

```

## 10.3 SpringData + MongoDB

**文档类**

```java
package data_persistent.document;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import java.util.Collection;
import java.util.HashSet;

/**
 * @author NikoBelic
 * @create 22/01/2017 17:38
 */
@Document(collection = "companys")
public class CompanyDoc
{
    @Id
    private Integer cid;
    private String companyName;
    private String location;
    private String financing;
    Collection<MemberDoc> members = new HashSet<>();

    public Integer getCid() {
        return cid;
    }

    public void setCid(Integer cid) {
        this.cid = cid;
    }

    public String getCompanyName() {
        return companyName;
    }

    public void setCompanyName(String companyName) {
        this.companyName = companyName;
    }

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    public String getFinancing() {
        return financing;
    }

    public void setFinancing(String financing) {
        this.financing = financing;
    }

    public Collection<MemberDoc> getMembers() {
        return members;
    }

    public void setMembers(Collection<MemberDoc> members) {
        this.members = members;
    }

    @Override
    public String toString() {
        return "CompanyDoc{" +
                "cid=" + cid +
                ", companyName='" + companyName + '\'' +
                ", location='" + location + '\'' +
                ", financing='" + financing + '\'' +
                ", members=" + members +
                '}';
    }
}

```


```java
package data_persistent.document;

/**
 * @author NikoBelic
 * @create 22/01/2017 18:00
 */
public class MemberDoc
{
    private Integer mid;
    private String name;
    private String age;
    private String job;

    public MemberDoc(Integer mid, String name, String age, String job) {
        this.mid = mid;
        this.name = name;
        this.age = age;
        this.job = job;
    }

    public Integer getMid() {
        return mid;
    }

    public void setMid(Integer mid) {
        this.mid = mid;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAge() {
        return age;
    }

    public void setAge(String age) {
        this.age = age;
    }

    public String getJob() {
        return job;
    }

    public void setJob(String job) {
        this.job = job;
    }

    @Override
    public String toString() {
        return "MemberDoc{" +
                "mid='" + mid + '\'' +
                ", name='" + name + '\'' +
                ", age='" + age + '\'' +
                ", job='" + job + '\'' +
                '}';
    }
}

```

**Mongo的持久化**
采用SpringData根据函数名自动生成Repository、使用Query自定义查询。

```java
package data_persistent.mongodao;

import data_persistent.document.CompanyDoc;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.mongodb.repository.Query;

import java.util.List;

/**
 * @author NikoBelic
 * @create 23/01/2017 12:57
 */
public interface CompanyRepo extends MongoRepository<CompanyDoc,Integer>
{
    List<CompanyDoc> findByLocationLike(String location);

    @Query("{'companyName':'犀牛科技'}")
    CompanyDoc findWithQuery();
}

```


## 10.4 测试类


```java
package data_persistent.test;

import com.mongodb.Mongo;
import data_persistent.config.RootConfig;
import data_persistent.dao.UserDao;
import data_persistent.document.CompanyDoc;
import data_persistent.document.MemberDoc;
import data_persistent.mongodao.CompanyRepo;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Collection;
import java.util.HashSet;
import java.util.List;

/**
 * @author NikoBelic
 * @create 22/01/2017 16:07
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = RootConfig.class)
public class PersistentTest
{
    @Autowired
    private UserDao userDao;

    @Autowired
    MongoOperations mongo;

    @Autowired
    CompanyRepo companyRepo;

    // ***************** JDBCTemplate *******************/
    @Test
    public void addUser()
    {
        userDao.addUser();
    }
    @Test
    public void findUser()
    {
        System.out.println(userDao.findUserById(1));
    }


    // ***************** MongoDB *******************/

    @Test
    public void mongo_add()
    {
        CompanyDoc company = new CompanyDoc();
        company.setCid(1);
        company.setCompanyName("犀牛科技");
        company.setLocation("北京工业大学");
        company.setFinancing("天使轮");
        Collection<MemberDoc> members = new HashSet<>();
        for (int i = 0; i < 3; i++)
        {
            members.add(new MemberDoc(i+1,"员工",(18+i) + "","码农" + (char)(97 + i)));
        }
        company.setMembers(members);
        mongo.insert(company,"companys");
    }

    @Test
    public void mongo_findAll()
    {
        //List<CompanyDoc> cList = mongo.findAll(CompanyDoc.class,"companys");
        //List<CompanyDoc> cList = companyRepo.findAll();
        //List<CompanyDoc> cList = companyRepo.findByLocationLike("北京");
        CompanyDoc c = companyRepo.findWithQuery();
        System.out.println(c);
        //for (CompanyDoc companyDoc : cList)
        //{
        //    System.out.println(companyDoc);
        //}
    }

    @Test
    public void mongo_advance()
    {
        List<CompanyDoc> companyDocs = mongo.find(Query.query(Criteria.where("companyName").regex("^犀牛*")), CompanyDoc.class, "companys");
        System.out.println(companyDocs);
    }

}

```


