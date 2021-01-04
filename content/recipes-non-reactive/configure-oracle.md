+++
categories = ["recipes"]
tags = ["persistence","hikari","database connection pool","anti patterns"]
summary = "Configure Oracle datasource in microservice"
title = "3. Configure Oracle"
date = 2020-12-09T14:02:27-05:00
weight = 2
+++

## CONTEXT
This recipe is part of a cookbook for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.   
This recipe deals with configuring Oracle to fulfill persistence needs in the microservice.  
A _Hikari_ datasource bean with connection pooling is created after completing this recipe. 

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

## SOLUTION

1. Create **environment variables** for **oracle database connection properties**, 
   these are defined in uDeploy and, are **never**   hardcoded in`application.yml`

   | Property      | Details  |
   | :---          |    :----   | 
   | `ORACLE_DB_URL`  |  `jdbc:oracle:thin:@<host>:<port>:<schema>` [thin] OR `jdbc:oracle:oci:@<host>:<port>:<schema>` [oci] |
   | `ORACLE_SCHEMA`     | database schema name  | 
   | `ORACLE_USERNAME` | username  | 
   | `ORACLE_PASSWORD` | encrypted password|
   
1. Configure the **database connection pool requirements** for the new microservice.  
  The pool configuration can be tailored to your microservice needs after reviewing the  [**anti-patterns**](https://github.com/pbelathur/spring-boot-performance-analysis).

   | Property        | Suggested Values  |
         | :---          |    :----   | 
   | max-pool-size  |  10  |
   | idle-pool-size | 2 | 
   | connection-timeout | 250ms  |
   | idle-timeout     | 800ms  | 
   | max-lifetime    | 3000ms |
 
1. Navigate to the `<microservice>` directory

1. Review the _database connection_ and _connection pool_ properties in `src/main/resources/application.yml`
   
   ```yml
   application:
     name:
     description:
     id: <wells fargo distributed-id>
     persistence:
      oracle:
        name: <user friendly database name>
        url: ${oracle.db.url}
        username: ${oracle.username}
        password: ${oracle.password}
        schema: ${oracle.schema}
        dialect: org.hibernate.dialect.Oracle12cDialect
        driver: oracle.jdbc.OracleDriver
        connection-pool:
          max-pool-size: 10
          idle-pool-size: 2
          connection-timeout: 500
          idle-timeout: 800
          max-lifetime: 3000
   ```
   - the `${placeholder}` is used for _automatic environment variable expansion_ during Gradle build.
   - the `placeholder` property is defined as an **environment variable** in uDeploy.
      - e.g.  environment variable: `ORACLE_DB_URL` corresponds to `${oracle.db.url}` 

### Validation
1. Open a command window in the `<microservice-name>` directory

2. Set all the required **environment variables** for the *property placeholder*s in`application.yml`

   ```shell
   set ORACLE_DB_URL=jdbc:oracle:thin:@<host>:<port>:<schema>
   set ORACLE_SCHEMA=schema-name
   set ORACLE_USERNAME=username
   set ORACLE_PASSWORD=encrypted password
   ```

3. Validate the new microservice ...
   - can be built locally: `gradlew bootJar`
   - runs locally: `gradlew bootRun`
 
  
4. Verify microservice health in the browser

   - `http://localhost:8080/actuator/info`

   - `http://localhost:8080/actuator/health`  
      the _datasource details_ and _uptime status_ should be displayed     in the browser.
     
   - `http://localhost:8080/actuator/beans`  
     search for _HikariDataSource_ in the browser.
     
## NOTES
- [database connection pool **anti-patterns**](https://github.com/pbelathur/spring-boot-performance-analysis)