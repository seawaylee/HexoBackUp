---
title: Redis学习（三）Java实现基于Jedis+Spring的通用工具类
date: 2017-03-09 21:49
tags: [Nosql,Redis]
---


**Spring整合Jedis**

Maven引入


```xml
  <!--Spring Data Redis-->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-redis</artifactId>
            <version>${spring-data-redis.version}</version>
        </dependency>
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>${jedis.version}</version>
        </dependency>

```

<!--more-->

Java配置类（也可以用XML）


```java
@Configuration
@EnableCaching
public class RedisCacheConfig extends CachingConfigurerSupport
{
    @Value("${redis_host}")
    String redisHost;

    @Value("${redis_port}")
    Integer redisPort;

    @Value("${redis_keynum}")
    Integer redisKeynum;

    Logger logger = LoggerFactory.getLogger(this.getClass().getSimpleName());


    public RedisCacheConfig()
    {
        System.out.println("Java配置类初始化");
    }

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
        JedisPool jedisPool = new JedisPool(jedisPoolConfig,redisHost,redisPort,999999999);
        return jedisPool;
    }
    @Bean
    public Jedis jedis(JedisPool jedisPool)
    {
        Jedis jedis = jedisPool.getResource();
        jedis.select(redisKeynum);
        return jedis;
    }


    @Bean
    public JedisConnectionFactory redisConnectionFactory()
    {
        JedisConnectionFactory factory = new JedisConnectionFactory();
        System.out.println(redisHost + ":" + redisPort + "-" + redisKeynum);
        logger.info(redisHost + ":" + redisPort + "-" + redisKeynum);
        factory.setHostName(redisHost);
        factory.setPort(redisPort);
        factory.setDatabase(redisKeynum);
        return factory;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory redisConnectionFactory)
    {
        RedisTemplate<String,String> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate)
    {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        cacheManager.setDefaultExpiration(0);;
        return cacheManager;
    }

}

```


**封装Jedis工具类**


```java
/**
 * Jedis工具类
 *
 * @author NikoBelic
 * @create 21/01/2017 13:41
 */
@Component
public class JedisUtils
{

    @Autowired
    Jedis jedis;

    private JedisUtils()
    {
    }

    // ********************** String Oprations *************************
    public String set(String key, String val)
    {
        return jedis.set(key, val);
    }

    public String get(String key)
    {
        return jedis.get(key);
    }

    public Long incrBy(String key, Integer increment)
    {
        return jedis.incrBy(key, increment);
    }

    public Long decrBy(String key, Integer decrement)
    {
        return jedis.incrBy(key, decrement);
    }


    // ********************** List Oprations *************************
    /**
     * 从左侧推入元素
     * @Author NikoBelic
     * @Date 21/01/2017 17:47
     */
    public <T extends Serializable> Long lPushObj(String key, T... obj)
    {
        byte[] serializedObj;
        long count = 0;
        for (T t : obj)
        {
            serializedObj = SerializationUtil.serialize(t);
            jedis.lpush(key.getBytes(),serializedObj);
            count++;
        }
        return count;
    }
    /**
     * 从列表左侧弹出数据
     * @Author NikoBelic
     * @Date 21/01/2017 17:47
     */
    public <T extends Serializable> T lPop(String key)
    {
        byte[] obj = jedis.lpop(key.getBytes());
        return SerializationUtil.deserialize(obj);
    }
    /**
     * 获取指定范围内的列表元素
     * @Author NikoBelic
     * @Date 21/01/2017 17:47
     */
    public <T extends Serializable> List<T> lRange(String key,int from,int to)
    {
        List<byte[]> byteList = jedis.lrange(key.getBytes(), from, to);
        List<T> objList = null;
        if (byteList.size() > 0)
        {
            objList = new ArrayList<T>();
            for (byte[] obj : byteList)
            {
                objList.add(SerializationUtil.deserialize(obj));
            }
        }
        return objList;
    }




    // ********************** Set Oprations *************************
    /**
     * 向集合中添加元素
     * @Author NikoBelic
     * @Date 21/01/2017 18:06
     */
    public <T extends Serializable> Long sAdd(String key, T... obj)
    {
        byte[] serializedObj;
        long count = 0;
        for (T t : obj)
        {
            serializedObj = SerializationUtil.serialize(t);
            jedis.sadd(key.getBytes(),serializedObj);
            count++;
        }
        return count;
    }
    /**
     * 返回集合中的所有元素
     * @Author NikoBelic
     * @Date 21/01/2017 18:06
     */
    public <T extends Serializable> Set<T> sMembers(String key)
    {
        Set<byte[]> byteList = jedis.smembers(key.getBytes());
        Set<T> objList = null;
        if (byteList.size() > 0)
        {
            objList = new HashSet<T>();
            for (byte[] obj : byteList)
            {
                objList.add(SerializationUtil.deserialize(obj));
            }
        }
        return objList;
    }
    /**
     * 判断对象是否存在于集合中
     * 注意:判断标准是列化后的字符串是否相同,即时不同的对象但序列化结果相同也将返回true
     * @Author NikoBelic
     * @Date 21/01/2017 18:04
     */
    public <T extends Serializable> Boolean sIsMember(String key, T obj)
    {
        byte[] serializedObj = SerializationUtil.serialize(obj);
        return jedis.sismember(key.getBytes(),serializedObj);
    }
    // ********************** Hash Oprations *************************
    /**
     * 向哈希表存储键值对数据
     * @Author NikoBelic
     * @Date 21/01/2017 18:41
     */
    public <T extends Serializable> String hmSet(String key, Map<String,T> map)
    {
        Map<byte[],byte[]> objMap;
        if (map.size() > 0)
        {
            objMap = new HashMap<>();
            for (Map.Entry<String, T> entry : map.entrySet())
            {
                objMap.put(entry.getKey().getBytes(), SerializationUtil.serialize(entry.getValue()));
            }
            return jedis.hmset(key.getBytes(), objMap);
        }
        return null;
    }
    /**
     * 从Hash中取出键值对数据
     * @Author NikoBelic
     * @Date 21/01/2017 18:41
     */
    public <T extends Serializable> List<T> hmGet(String key, String... fields)
    {
        List<T> resObjs = new ArrayList<T>();
        for (String field : fields)
        {
            resObjs.add(SerializationUtil.deserialize(jedis.hget(key.getBytes(),field.getBytes())));
        }
        return resObjs;
    }
    // ********************** ZSet Oprations *************************


}

```

**测试方法**


```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath*:spring/applicationContext*.xml"})
public class TestRedis
{
    @Autowired
    private JedisUtils jedisUtils;

    @Test
    public void testString()
    {
        System.out.println(jedisUtils.set("address", "ShangHai"));
        System.out.println(jedisUtils.get("address"));
        try
        {
            System.out.println(jedisUtils.incrBy("address", 100));
        } catch (Exception e)
        {
            System.out.println(e.getMessage() + "自增操作异常");
        }

        jedisUtils.set("myint", "10");
        System.out.println(jedisUtils.incrBy("myint", 100));
    }

    @Test
    public void testList()
    {
        System.out.println(jedisUtils.lPushObj("strList", "NikoBelic","Tom","Helen"));
        //System.out.println((String) jedisUtils.lPop("strList"));
        //System.out.println((String) jedisUtils.lPop("strList"));
        List<String> strList = jedisUtils.lRange("strList", 0, -1);
        for (String s : strList)
        {
            System.out.println(s);
        }

        //System.out.println(jedisUtils.lPushObj("objList", new User("NikoBelic", 18), new User("Tom", 15), new User("Marry", 20)));
        //System.out.println((User) jedisUtils.lPop("objList"));
        //System.out.println((User) jedisUtils.lPop("objList"));
        //List<User> objList = jedisUtils.lRange("objList", 0, -1);
        //for (User user : objList)
        //{
        //    System.out.println(user);
        //}
    }

    @Test
    public void testSet()
    {
        //System.out.println(jedisUtils.sAdd("strSet", "NikoBelic","Tom","Helen"));
        //Set<String> strList = jedisUtils.sMembers("strSet");
        //for (String s : strList)
        //{
        //    System.out.println(s);
        //}

        //System.out.println(jedisUtils.sAdd("objSet", new User("NikoBelic", 18), new User("Tom", 15), new User("Marry", 20)));
        //Set<User> objList = jedisUtils.sMembers("objSet");
        //for (User user : objList)
        //{
        //    System.out.println(user);
        //}
        //User myObj = new User("Exist Test",20);
        //jedisUtils.sAdd("objSet",myObj);
        System.out.println(jedisUtils.sIsMember("objSet",new User("ExistTest",20)));

    }

    @Test
    public void testHash()
    {
        //Map<String,String> map = new HashMap<>();
        //map.put("A","1");
        //map.put("B","2");
        //map.put("C","3");
        //System.out.println(jedisUtils.hmSet("strHash",map));

        //List<String> strs = jedisUtils.hmGet("strHash", "A", "B", "C");
        //for (String str : strs)
        //{
        //    System.out.println(str);
        //}



        Map<String,User> map = new HashMap<>();
        map.put("user1",new User("Niko",18));
        map.put("user2",new User("Tom",20));
        map.put("user3",new User("Marry",15));
        System.out.println(jedisUtils.hmSet("objHash",map));
        List<User> userList = jedisUtils.hmGet("objHash", "user1", "user2", "user3");
        for (User user : userList)
        {
            System.out.println(user);
        }

    }

}

class User implements Serializable
{
    private String name;
    private int age;

    public User(String name, int age)
    {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age='" + age + '\'' +
                '}';
    }
}



```


