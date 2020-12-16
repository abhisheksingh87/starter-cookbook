+++
categories = ["recipes"]
tags = ["persistence","mongodb","spring boot"]
summary = "Configure mongodb datasource in microservice"
title = "Configure mongodb"
date = 2020-12-09T14:02:27-05:00
weight = 1

+++

## Context
This is the second recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
This recipe deals with configuring persistence in the microservice.  

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

## Solution

1. Determine and record the following **mongodb database connection details** 

   | Property        | Details  |
      | :---          |    :----   | 
   | host  |  `[host]`  # Mongo server host |
   | port | `[port]` #Mongo server port   |
   | database | localhost #database  name  |
   | username | Login user of the mongo server - WellsFargo standard 
   | password | Login password of the mongo server - WellsFargo standard
   | uri | mondogb://localhost/test    #Mongo database URI. when set, host and port are ignored.
 
1. Navigate to the `<microservice>` directory
   
1. Update the _database connection_ and _connection pool requirements_ in `src/main/resources/application.yml`

   ```yml
   application:
           name:
           description:
           id: <wells fargo distributed-id>
           persistence:
              mongodb:
                  host:
                  port:
                  database:
                  username:
                  password:
    ```

### Validation

1. Open a command window in the `<microservice-name>` directory


1. Validate the new microservice can be built locally: `gradlew bootJar`


1. Validate the new microservice runs locally: `gradlew bootRun`


1. Verify microservice health and info in the browser

   - `http://localhost:8080/actuator/info`
     
   - `http://localhost:8080/actuator/health`  
      the _datasource details_ and _uptime status_ should be displayed in the browser.
   
   - `http://localhost:8080/actuator/beans`  
     search for _mongo_ in the browser and you should see below
     ```json
      "mongo": {
         "status": "UP",
         "details": {
         "version": "4.2.0"
         }
      },    
     ```

## Notes and References

  you can also use properties provided by **Spring** to configure mongodb
  ```yaml
     spring:
        mongodb:
          host:  
          port:    
          uri:    
          database:   
          username:
          password:
  ```

### Anti Patterns

