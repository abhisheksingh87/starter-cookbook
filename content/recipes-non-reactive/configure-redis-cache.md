+++
categories = ["recipes"]
tags = ["starter","spring-boot-app","microservice"]
summary = "Configure Redis Cache in microservice"
title = "Configure Redis Cache"
date = 2020-12-09T14:02:27-05:00

+++                     

## Context
This is the third recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
This recipe deals with configuring the microservice to use a distributed (Redis) cache for data caching needs.
### Prerequisite

- **STEP 1: CREATE APPLICATION USING STARTER** is completed.

## Solution

1. Determine and record the following **redis cache connection details**

   | Property        | Details  |
         | :---          |    :----   | 
   | jdbc url  |  `jdbc:oracle:thin:@<host>:<port>:<schema>` [thin] OR `jdbc:oracle:oci:@<host>:<port>:<schema>` [oci] |
   | driver classname | `oracle.jdbc.OracleDriver`    |
   | schema     | database schema name  | 
   | dialect    | Hibernate dialect based on the Oracle database version `org.hibernate.dialect.Oracle12cDialect` | 
   | username | schema username | 
   | password | schema password | 

1. Determine and record the **cache connection pool requirements** for the new microservice

   | Property        | Suggested Values  |
            | :---          |    :----   | 
   | maximumPoolSize  |  10  |
   | idlePoolSize | 2 | 
   | connectionTimeout | 250ms  |
   | idleTimeout     | 800ms  | 
   | maxLifetime    | 3000ms | 
 

1. Navigate to the `<microservice>` directory

1. Update `src/main/resources/application.yml` with Redis (_connection, credentials and connection pool_) details.

   ```yml
      application:
        name:
        description:
        id: <wells fargo distributed-id>
        persistence:
          .....
          .....
       cache:
         redis:
           name: <redis server name> 
           host: <hostname or ip address of the redis cache>
           port:
           username:
           password:
           timeout: <in ms>
           time-to-live: <in minutes>
           enable-repository: <true or false>
           connection-pool:
             max-active:
             max-idle:
             max-wait: <in ms>
             min-idle:
             cluster-nodes: <comma seperated host:port pairs>
             sentinel-nodes: <comma seperated host:port pairs>
   ```

### Validation  

1. Open a command window in the `<microservice-name>` directory

1. Validate the new microservice can be built locally: `gradlew bootJar`

1. Validate the new microservice runs locally: `gradlew bootRun`

1. Verify microservice health and info in the browser

   - `http://localhost:8080/actuator/info`

   - `http://localhost:8080/actuator/health`  
     the _redis cache details_ and _uptime status_ should be displayed in the browser.

## How to enable caching in your application ?

   - add `@EnableCaching` to one of your `configuration classes`. (preferably the 
     class annotated with `@SpringBootApplication`)


   - add `@Cacheable` to the methods you want to enable caching.


   - optionally, add `@CacheEvict` to remove/delete the cached 
     object.


Here is a example that demonstrates the annotations in a typical service.

   ```java
   @Service
   public class ItemService {
  
   private final ItemRepository itemRepository;
   
       public ItemService(ItemRepository itemRepository) {
           this.itemRepository = itemRepository;
       }
   
       public List<Item> items() {
           return itemRepository.findAll();
       }
   
       @Cacheable(value = "items", key = "#id")
       public Item getItem(Integer id) {
           Item item = itemRepository.findById(id);
           return item;
       }
   
       public Item createItem(Item item) {
           return itemRepository.save(item);
       }
   
       @CacheEvict(value = "items", key = "#id")
       public Item updateItem(Integer id, Item request) {
           Item item = getItem(id);
           item.setPrice(request.getPrice());
           item.setProductName(request.getProductName());
           return itemRepository.save(item);
       }
   }
   ```

## Next Step
Configure the Messaging

## Notes and References

### Recommendations
- **Cacheable objects must be Serializable**

   The reason is due to how redis stores java objects. The safest way to store 
   objects outside JVM is to write them into serialized bytes. To do that, those classes must implement Serializable.


- **Set the TTL**

  Take advantage of expiring keys and Redis will clean up for you


- **Choosing the Proper Eviction Policy**

  When your Redis instance fills up, Redis will attempt to evict keys.
  Depending on your use case, we recommend _volatile-lru_ assuming you have expiring keys. If you’re running like a cache and don’t have an expiry set, you could consider _allkeys-lru_.
  

- **Try not to cache large objects.**

   Large objects in the cache will cause performance issues.


- **Make sure all applications using the same cache are at the same version.**

   Cached objects created by application with the version A may not be 
   compatible to the application with the version B. These type of situations
   will yield unpredictable results.



   