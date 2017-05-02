---
title: Chapter13 Spring Cache
date: 2017-03-09 23:50:48
tags: [JavaWeb,Spring]
---


缓存对于某些不要求实时获取最新数据的请求非常好用，如果再高并发环境下，数据库成为系统的性能瓶颈，使用缓存能够大幅度提升系统性能。
本文以Redis作为缓存容器，结合Spring来模拟一个缓存系统。
个人认为，如果你会使用Redis，则完全没有必要将其与Spring整合来实现缓存，自己使用Jedis工具来实现缓存更加灵活。但是如果你不懂Redis，那么使用SpringCache+Redis就可以了。

<!--more-->

## 13.1 Redis配置文件读取配置


```java
@Configuration
@PropertySource("classpath:redis.properties")
public class PropertyConfig
{
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
}
```

## 13.2 SpringCache配置类


```java
package data_persistent.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

/**
 * 缓存配置
 * @author NikoBelic
 * @create 24/01/2017 15:47
 */
@EnableCaching
@Configuration
public class CacheConfig
{

    @Value(value = "${host}")
    private String host;
    @Value(value = "${port}")
    private Integer port;
    @Value(value = "${keynum}")
    private Integer keynum;

    /**
     * 注入CacheManager
     * @Author NikoBelic
     * @Date 24/01/2017 16:05
     */
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate)
    {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        cacheManager.setDefaultExpiration(10);
        return cacheManager;
    }

    /**
     * 注入JedisConnectionFactory
     * @Author NikoBelic
     * @Date 24/01/2017 16:03
     */
    @Bean
    public JedisConnectionFactory redisConnectionFactory()
    {
        JedisConnectionFactory factory = new JedisConnectionFactory();
        factory.setHostName(host);
        factory.setPort(port);
        factory.setDatabase(keynum);
        //factory.afterPropertiesSet();
        return factory;
    }

    /**
     * 注入RedisTemplate
     * @Author NikoBelic
     * @Date 24/01/2017 16:02
     */
    @Bean
    public RedisTemplate<String,String> redisTemplate(RedisConnectionFactory redisCF)
    {
        RedisTemplate<String,String> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisCF);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    // *********************** Jedis 配置 ***************************

    // 以下配置与缓存无关,在本例中可有可无
    @Bean
    public JedisPoolConfig jedisPoolConfig()
    {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxWaitMillis(60 * 1000);
        jedisPoolConfig.setMaxTotal(1000);
        jedisPoolConfig.setMaxIdle(100);
        return jedisPoolConfig;
    }

    @Bean
    public JedisPool jedisPool(JedisPoolConfig jedisPoolConfig)
    {
        JedisPool jedisPool = new JedisPool(jedisPoolConfig,host,port,999999999);
        return jedisPool;
    }
    @Bean
    public Jedis jedis(JedisPool jedisPool)
    {
        Jedis jedis = jedisPool.getResource();
        jedis.select(keynum);
        return jedis;
    }
}

```

## 13.3 Dao层使用注解将结果缓存


```java
public interface UserDao
{
    @CachePut(value = "UserCaching",key = "#result.id")
    UserObj addUser(UserObj userObj);

    @Cacheable("UserCaching")
    UserObj findUserById(Integer id);

    void updateUserById(Integer id,UserObj userObj);

    @CacheEvict("UserCaaching")
    void deleteUserById(Integer id);
}

```

实现


```java
package data_persistent.dao;

import data_persistent.model.UserObj;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcOperations;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Repository;

import javax.inject.Inject;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;

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
     *
     * @Author NikoBelic
     * @Date 22/01/2017 16:25
     */
    public UserObj addUser(UserObj user)
    {
        KeyHolder holder = new GeneratedKeyHolder();
        int id = jdbcOperations.update(connection -> {
            PreparedStatement ps = connection.prepareStatement("insert into tb_user(username,password,role) values(?,?,?)", Statement.RETURN_GENERATED_KEYS);
            ps.setString(1, user.getUsername());
            ps.setString(2, user.getPassword());
            ps.setString(3, user.getRole());
            return ps;
        }, holder);
        user.setId(holder.getKey().intValue());
        return user;
    }

    public UserObj findUserById(Integer id)
    {
        System.out.println("findUserById si called...");
        return jdbcOperations.queryForObject("select * from tb_user t where t.id = ?",
                (rs, rowNum) -> {
                    return new UserObj(rs.getInt("id"), rs.getString("username"), rs.getString("password"), rs.getString("role"));
                }, id);
    }

    public void updateUserById(Integer id, UserObj userObj)
    {
        jdbcOperations.update("update tb_user t set t.username = ? ,t.password = ?,t.role = ? where t.id = ?",
                userObj.getUsername(), userObj.getPassword(), userObj.getRole(), id);
    }

    @Override
    public void deleteUserById(Integer id)
    {
        //jdbcOperations.update("delete from tb_user where id = ?",id);
        System.out.println("删除数据库中的用户" + id);
    }

}

```

## 10.4 测试


```java
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

    //@Autowired
    //Jedis jedis;
    @Autowired
    KVDao kvDao;

    // ***************** JDBCTemplate *******************/
    @Test
    public void addUser()
    {
        UserObj user = new UserObj();
        user.setUsername("蛤蟆皮");
        user.setPassword("123456");
        user.setRole("ADMIN");
        System.out.println(userDao.addUser(user));
    }

    @Test
    public void findUser() throws InterruptedException
    {
        System.out.println(userDao.findUserById(12));
        Thread.sleep(11 * 1000);
        System.out.println(userDao.findUserById(12));
    }

    @Test
    public void updateUser()
    {
        UserObj userObj = new UserObj(1,"阿西吧","password","ABC");
        userDao.updateUserById(1,userObj);
    }

    
    // ***************** Redis Caching *******************/

    @Test
    public void redis_conn()
    {
        System.out.println(kvDao.get("chinese"));
        kvDao.set("", "");
        System.out.println(kvDao.get("chinese"));
    }
    @Test
    public void test_cache()
    {
        System.out.println("Before Update...");
        System.out.println(userDao.findUserById(1));
        UserObj userObj = new UserObj(1,"Hey","密码","管理员");
        userDao.updateUserById(userObj.getId(),userObj);
        System.out.println("After Update...");
        System.out.println(userDao.findUserById(1));
    }
    @Test
    public void test_del_cache()
    {
        UserObj userObj = new UserObj(9,"Cache","密码","管理员");
        userObj = userDao.addUser(userObj);
        System.out.println(userDao.findUserById(userObj.getId()));
        userDao.deleteUserById(userObj.getId());
        System.out.println(userDao.findUserById(userObj.getId()));
    }
```

## 10.5 Redis数据库缓存结果


```commandline
127.0.0.1:6379[4]> keys *
1) "\xac\xed\x00\x05sr\x00\x11java.lang.Integer\x12\xe2\xa0\xa4\xf7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x00\x0b"
2) "\xac\xed\x00\x05sr\x00\x11java.lang.Integer\x12\xe2\xa0\xa4\xf7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xac\x95\x1d\x0b\x94\xe0\x8b\x02\x00\x00xp\x00\x00\x00\b"
3) "UserCaching~keys"
```

UserCaching~keys 是一个ZSet结构，是一个有序集合。

## 10.6 注解


| 注解 | 描述 |
| --- | --- |
| @Cacheable | 表明Spring在调用方法之前，首先应该在缓存中查找方法的返回值，如果这个值能够找到，就会返回缓存的值，否则的话，这个方法就会被调用，返回值会放到缓存之中。 |
| @CachePut | 表明Spring应该将方法的返回值放到缓存中，在方法的调用前并不会检查缓存，方法始终都会被调用。 |
| @CacheEvict | 表明Spring应该在缓存中清楚一个或多个条目 |
| @Caching | 这事一个分组的注解，能够同时应用多个其他的缓存注解 |



