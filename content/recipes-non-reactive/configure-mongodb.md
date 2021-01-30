+++
draft = false
categories = ["recipes"]
date = "2021-01-29T14:45:27-06:00"
lastmod = "2021-01-29"
summary = "Configure MongoDB"
description = "Configure MongoDB"
tags = ["application development", "Spring", "Spring Boot", "MongoDB", "persistence"]
title = "Configure MongoDB"
weight = 2
+++

## CONTEXT
This is the second recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
This recipe deals with configuring persistence in the microservice.  

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

## SOLUTION

1. Determine and record the following **mongodb database connection details** 

   | Property        | Details  |
   | :---            |    :----   | 
   | uri | mongodb://\<host>:\<port>/\<database name> 
   | database-name | database  name  |
 
1. Navigate to the `<microservice>` directory

1. Update the _database connection_ as environment variables `src/main/resources/application.yml`

    ```yml
    mongodb:
      hosts: ${mongoDBUrl}
      database-name: ${databaseName}
    ```
1. Create a new class `MongoProperties` . This class will load the MongoDB properties from `src/main/resources/application.yml`

    ```java
       @ConfigurationProperties("mongodb")
       @Component
       @Data
       public class MongoProperties {
          private List<String> hosts;
          private String database;
     }
    ```

1. Create new class `MongoDBConfiguration`. This class will extend `AbstractMongoClientConfiguration`.
   The purpose of this class is to load MongoDB connection url and database.

    ```java
    @Configuration
    @AllArgsConstructor
    public class MongoDBConfiguration extends AbstractMongoClientConfiguration {
    
        private static final String CONNECTION_URL = "mongodb://%s/%s";
    
        private final MongoProperties mongoProperties;
    
        @Override
        protected String getDatabaseName() {
            return mongoProperties.getDatabase();
        }
    
        @Bean
        MongoTransactionManager transactionManager(MongoDatabaseFactory dbFactory) {
            return new MongoTransactionManager(dbFactory);
        }
    
        private String getMongoDBConnectionUrl() {
            return String.format(CONNECTION_URL, mongoProperties.getHosts().get(0), mongoProperties.getDatabase());
        }
    
        @Override
        public MongoClient mongoClient() {
            final ConnectionString connectionString = new ConnectionString(getMongoDBConnectionUrl());
            final MongoClientSettings mongoClientSettings = MongoClientSettings.builder()
                    .applyConnectionString(connectionString)
                    .build();
            return MongoClients.create(mongoClientSettings);
        }
    }
    ```

### Validation

1. Open a command window in the `<microservice-name>` directory

1. Validate the new microservice
   - can be built locally: `gradlew bootJar`
   - runs locally: `gradlew bootRun`

1. Verify microservice health in the browser

   - `http://localhost:8080/actuator/info`
     
   - `http://localhost:8080/actuator/health`  
      the _datasource details_ and _uptime status_ should be displayed in the browser.
   
   - `http://localhost:8080/actuator/beans`  
     search for _mongo_ in the browser and you should see the following
     ```json
      "mongo": {
         "status": "UP",
         "details": {
         "version": "4.2.0"
         }
      },    
     ```


