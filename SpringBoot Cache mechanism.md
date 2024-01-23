# Spring Boot Cache

Before going SpringBoot cache mechanism, let understand what’s the Cache is.
•	Cache is a part of temporary memory (RAM). It lies between the application and the persistent database.
•	Caching is a mechanism used to increase the performance of a system. It is a process to store and access data from the cache.
•	It stores the recently used data. This helps to reduces the number of database hits as much as possible.
 


## Types of Caching
There are four types of caching are as follows :
1.	Database Caching.
2.	In-memory Caching.
3.	Web Server Caching.
4.	CDN Caching

1. Database Caching: 
Hibernate first level cache is an example of database caching.

2. In-memory Caching: 
In this type of caching, data is stored in RAM. EhCache and Redis are examples of in-memory caching. EhCahe is a simple in-memory cache while Redis is advanced.

3. Web Server Caching :
 It is cached for the first time when a user visits the page. If the user requests the same next time, the cache serves a copy of the page.

4. CDN Caching (Content Delivery Network) :
It improves delivery of the content by replicating common requested files such as Html Pages, images, videos, etc. across distributed set of caching servers.

In SpringBoot we use mostly use Database Caching & In-memory Caching

### Database Cache
Take Hibernate, Every fresh session having its own cache memory, Caching is a mechanism for storing the loaded objects into a cache memory.
The advantage of cache mechanism is, whenever again we want to load the same object from the database then instead of hitting the database once again, it loads from the local cache memory only, so that the no. of round trips between an application and a database server got decreased.
It means caching mechanism increases the performance of the application.
In hibernate we have two levels of caching
1.	First Level Cache (Session Cache)
2.	Second Level Cache (Session Factory Cache/ JVM Level Cache)

**1.First Level Cache**
•	By default, for each hibernate application, the first level cache is automatically enabled. We can’t Enable/Disable first level cache
•	the first level cache is associated with the session object and scope of the cache is limited to one session only
•	When we load an object for the first time from the database then the object will be loaded from the database and the loaded object will be stored in the cache memory maintained by that session object
•	If we load the same object once again, with in the same session, then the object will be loaded from the local cache memory not from the database
•	If we load the same object by opening other session, then again the object will load from the database and the loaded object will be stored in the cache memory maintained by this new session

```
Example:
Session session = factory.openSession();
Object ob1 = session.get(Actor.class, new Integer(101)); //1
 
Object ob2 = session.get(Actor.class, new Integer(101));//2
Object ob3 = session.get(Actor.class, new Integer(101));//3
session.close();//4
 
Session ses2 = factory.openSession();
Object ob5 = ses2.get(Actor.class, new Integer(101));//5
```

1, We are loaded object with id 101, now it will load the object from the database only as its the first time, and keeps this object in the session cache
2,3 i tried to load the same object 2 times, but here the object will be loaded from the stored cache only not from the database, as we are in the same session
4, we close the first session, so the cache memory related this session also will be destroyed
5, again i created one new session and loaded the same object with id 101, but this time hibernate will loads the object from the database
if we want to remove the objects that are stored in the cache memory, then we need to call either evict() or clear() methods

2.Second Level Cache
Whenever we are loading any object from the database, then hibernate verify whether that object is available in the local cache(first level cache) memory of that particular session, if not available then hibernate verify whether the object is available in global cache(second level cache), if not available then hibernate will hit the database and loads the object from there, and then first stores in the local cache of the session , then in the global cache
SessionFactory holds the second level cache data. It is global for all the session objects and not enabled by default.
Different vendors have provided the implementation of Second Level Cache
1.	EH Cache (Basic In-Memory)
2.	Redis Cache (Advanced In-Memory)
3.	Memcached 
4.	JBoss Cache

SpringBoot Cache API (Common for Both EhCache & Redis)
Spring boot provides a Cache Abstraction API that allow us to use different cache providers to cache objects.
 
When the caching is enabled then the application first looks for required object in the cache instead of fetching from database. If it does not find that object in cache then only it access from database.


Spring Boot Caching Annotations

1. @EnableCaching (Class Level)
It is a class level annotation. It is used to enable caching in spring boot application. By default it setup a CacheManager and creates in-memory cache using ConcurrentHashMap.

We can specify this in SpringBoot main Application class & Configuration class
//Application.java
@EnableCaching
public class Application {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
    }    
}


//CacheConfig.java
@Configuration
@EnableCaching
public class CacheConfig {

}

 

2. @Cacheable
Cacheable  annotation is used to mark a method as cacheable. When a method is called, its result is stored in the memory cache. Subsequent invocations of the method with the same provided arguments will retrieve the cached result instead of invoking the method again.

@Cacheable(“employees”)
public Employee findById(int id) {
// some code
}

-	employees: name of the cache
parameter	Description
value / cacheNames	Name of the cache in where results of method execution are stored.
key	The key for the cache entries as Spring Expression Language (SpEL). If the parameter is not specified, a key is created for all method parameters by default.
keyGenerator	Name of a bean that implements the KeyGenerator interface and thus allows the creation of a user-defined cache key.
condition	Condition as Spring Expression Language (SpEL) that specifies when a result is to be cached.
unless	Condition as Spring Expression Language (SpEL) that specifies when a result should not be cached.




3. @CachePut
It is a method level annotation. It is used to update the cache before invoking the method.


4. @CacheEvict
It is a method level annotation. It is used to remove the data from the cache.


5. @Caching 
Let’s say you have a scenario where you want to use multiple annotations of the same type for caching a method. @Caching annotations allow you to use group multiple cache-related annotations together. 

@Caching(evict = {
@CacheEvict(“address”), 
@CacheEvict(value=“employee”, key=”#employee.id”)
})
public Employee getEmployee(Employee employee) {
// some code
}




6. @CacheConfig
It is a class level annotation. It is used to share common properties such as cache name, cache manager to all methods annotated with cache annotations.

When a class is declared with this annotation then it provides default setting for any cache operation defined in that class. Using this annotation, we do need to declare things multiple times.

For example,
@Service
@CacheConfig(cacheNames=”employees”)
public class EmployeeService {  
@Cacheable
  public Employee findById(int id) {
    // some code
  }
}




Spring Boot Cache Providers
The cache providers allow us to configure cache transparently and explicitly in an application.
The following steps are needed in order to configure any of these cache providers :
1.	Add the annotation @EnableCaching in the configuration file.
2.	Add the required caching maven dependencies 
3.	Add the configuration.xml for the cache provider in the root class path.



The following are the cache provider supported by Spring Boot framework :
1.	JCache (JSR-107)
2.	EhCache
3.	Hazelcast
4.	Infinispan
5.	Couchbase
6.	Redis
7.	Caffeine



EhCache
The EhCache is an open source Java based cache used to boost performance. It stores the cache in memory and disk (SSD).
EhCache used a file called ehcache.xml. The EhCacheCacheManager is automatically configured if the application found the file on the classpath.
If we want to use EhCache then we need to add the following dependency :
<dependency>
  <groupId>org.ehcache</groupId>
  <artifactId>ehcache</artifactId>
</dependency>




Redis
Redis is a popular in-memory data structure. It is a keystore-based data structure which is used to persist data.
The RedisCacheManager is automatically configured when we configure Redis. The default configuration is set by using property spring.cache.redis.*.
If we want to use Redis then we need to add the following dependency :
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-data-redis</artifactId>
</dependency>




SpringBoot + EhCache Example


1. Add Maven Dependencies

<!-- EhCache -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
   <groupId>javax.cache</groupId>
   <artifactId>cache-api</artifactId>
   <version>1.1.1</version>
</dependency>
<dependency>
   <groupId>org.ehcache</groupId>
   <artifactId>ehcache</artifactId>
   <version>3.8.1</version>
</dependency>
<!-- EhCache -->


2. add ext/ehcache.xml file in resources folder with Cache  details


<?xml version="1.0" encoding="UTF-8"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:jsr107="http://www.ehcache.org/v3/jsr107" 
   xmlns="http://www.ehcache.org/v3"
   xsi:schemaLocation="
      http://www.ehcache.org/v3 http://www.ehcache.org/schema/ehcache-core-3.6.xsd
      http://www.ehcache.org/v3/jsr107 http://www.ehcache.org/schema/ehcache-107-ext-3.6.xsd">
   
    <service>
        <jsr107:defaults enable-management="true" enable-statistics="true"/>
    </service>
    
    <persistence directory="./cache"/>
    
    <cache alias="GenericCache" uses-template="generic-cache"></cache>
   <cache alias="PreAuthUsers" uses-template="preauthusers-cache"></cache>
   <cache alias="employees" uses-template="employees-cache"></cache>
    
    <!-- Cache Templates-->
    <cache-template name="generic-cache">
      <expiry>
           <tti unit="seconds">300</tti>
       </expiry>
       <heap unit="entries">2000</heap>
    </cache-template>

    <cache-template name="preauthusers-cache">
      <expiry>
           <tti unit="seconds">300</tti>
       </expiry>
       <heap unit="entries">2000</heap>
    </cache-template>


   <cache-template name="employees-cache">
      <expiry>
         <tti unit="seconds">300</tti>
      </expiry>
      <heap unit="entries">2000</heap>
   </cache-template>
</config>



3. add  ext/ehcache.xml  classpath to application.yaml configuration.
spring:
  cache:
    jcache:
      config: classpath:ext/ehcache.xml



2. Create CacheConfig Class & Add @EnableCaching 
@Configuration
@EnableCaching
public class CacheConfig {

}



3.Open Service Class & add appropriate Cache annotations on the top of the methods.


@Override
@Cacheable(value = "employees", key = "#id")
public EmployeeDto getEmployeeById(Long id) {
    var entity = employeeRepository.findById(id).orElseThrow(() -> new EntityNotFoundException(Employee.class, id));
    return employeeMapper.toDto(entity);
}

@Override
@Cacheable(value = "employees")
public List<EmployeeDto> getAllEmployees() {
    List<Employee> employees = employeeRepository.findAll();
    return employeeMapper.mapEntityListToDtoListForEmployee(employees);
}


@Override
@Transactional(propagation = Propagation.REQUIRES_NEW)
@CacheEvict(cacheNames = "employees", allEntries = true)
public void delete(Long employeeId) {
    try {
        var entity = this.employeeRepository.findById(employeeId).orElseThrow(() -> new EntityNotFoundException(Employee.class, employeeId));
        employeeRepository.delete(entity);
    } catch (Exception ex) {
        log.error("Error while deleting", ex);
        throw ex;
    }
}




Yup! That’s all. Call the API now & check the logs. Or you can use below API to get cached data.

@RestController
@RequestMapping("/cache")
@Slf4j
public class CacheController {
    @Autowired
    CacheManager cacheManager;

    @GetMapping("/all")
    public Map<String, Object> getAllCachedData() {
        Map<String, Object> cachedDataMap = new HashMap<>();

        // Get the names of all caches managed by the CacheManager
        for (String cacheName : cacheManager.getCacheNames()) {
            Cache cache = cacheManager.getCache(cacheName);
            log.info("Cache Name: {} , Cache: {} ", cacheName, cacheManager);

            cachedDataMap.put(cacheName, cache);
        }
        return cachedDataMap;
    }
}



SpringBoot + Redis Cache Example


First You need to Install Redis Server in your system. Default Running in 127.0.0.1:6379
satyakaveti@Satyas-MacBook-Pro-2 ~ % brew services info redis
redis (homebrew.mxcl.redis)
Running: ✔
Loaded: ✔
Schedulable: ✘
User: satyakaveti
PID: 911


1. Add maven dependency 
<!-- redis  -->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>


2. Update application.yaml configuration.
spring:
  cache:
    type: redis
    host: localhost
    port: 6379
    redis:
      time-to-live: 60000


** Make sure all the DTO classes implements Serializable , otherwise will get Exception


That’s All. We have already implemented cache code. Hit the get employees API , check redis UI (GUI Tool: RedisInsight)

 






![image](https://github.com/smlcodes/OnePageTutorials/assets/20472904/1b2f2409-aa62-4fe3-9246-51078ec2e46c)
