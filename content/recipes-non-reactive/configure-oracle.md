+++
categories = ["recipes"]
tags = ["persistence","hikari","database connection pool","anti patterns"]
summary = "Configure Oracle datasource in microservice"
title = "Configure Oracle"
date = 2020-12-09T14:02:27-05:00
weight = 2
+++

## CONTEXT
This recipe is part of a cookbook for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.   
This recipe deals with configuring persistence in the microservice.  
The datasource bean with preconfigured _Hikari_ connection pool is used to fulfill persistence needs in the microservice.

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

## SOLUTION

1. Determine and record the following **oracle database connection details** 

   | Property        | Details  |
      | :---          |    :----   | 
   | jdbc url  |  `jdbc:oracle:thin:@<host>:<port>:<schema>` [thin] OR `jdbc:oracle:oci:@<host>:<port>:<schema>` [oci] |
   | driver classname | `oracle.jdbc.OracleDriver`    |
   | schema     | database schema name  | 
   | dialect    | Hibernate dialect based on the Oracle database version `org.hibernate.dialect.Oracle12cDialect` | 
   | username | username  | 
   | password | encrypted password| 


1. Determine and record the **database connection pool requirements** for the new microservice.  
  The pool configuration can be tailored to your microservice needs after reviewing the  [**anti-patterns**](https://github.com/pbelathur/spring-boot-performance-analysis).

   | Property        | Suggested Values  |
         | :---          |    :----   | 
   | max-pool-size  |  10  |
   | idle-pool-size | 2 | 
   | connection-timeout | 250ms  |
   | idle-timeout     | 800ms  | 
   | max-lifetime    | 3000ms | 

 
 
1. Navigate to the `<microservice>` directory
   
1. Update the _database connection_ and _connection pool requirements_ in `src/main/resources/application.yml`

   ```yml
      application:
        name:
        description:
        id: <wells fargo distributed-id>
        persistence:
           oracle:
               name: <user friendly database name>
               url: <jdbc url format>
               username:
               password: <encrypted password>
               schema: 
               dialect: <hibernate oracle dialect>
               connection-pool:
                 max-pool-size: <less than 10>
                 idle-pool-size: <max-pool-size/5>
                 connection-timeout: <in millisecs>
                 idle-timeout: <in millisecs>
                 max-lifetime: <in millisecs>
   ```

### Validation

1. Open a command window in the `<microservice-name>` directory

1. Validate the new microservice
   - can be built locally: `gradlew bootJar`
   - runs locally: `gradlew bootRun`

1. Verify microservice health  in the browser

   - `http://localhost:8080/actuator/info`
     
   - `http://localhost:8080/actuator/health`  
      the _datasource details_ and _uptime status_ should be displayed in the browser.
   
   - `http://localhost:8080/actuator/beans`  
     search for _HikariDataSource_ in the browser.
     
## NOTES
- [database connection pool **anti-patterns**](https://github.com/pbelathur/spring-boot-performance-analysis)