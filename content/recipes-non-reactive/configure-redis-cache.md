+++
categories = ["recipes"]
tags = ["starter","spring-boot-app","microservice"]
summary = "Configure Redis Cache in microservice"
title = "Configure Redis"
date = 2020-12-09T14:02:27-05:00

+++                     

## Context
This is the third recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
This recipe deals with configuring cache in the microservice.
### Prerequisite

- **STEP 2: CONFIGURE PERSISTENCE IN MICROSERVICE** is completed.

## Solution
 
1. Navigate to the `<microservice>` directory
   
1. Update `src/main/resources/application.yml` with Redis connection, credentials and other configuration related details.

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
               password:
               schema:
               dialect: <hibernate oracle dialect>
               connection-pool:
                 name: <<user friendly pool name>
                 maximumPoolSize: <less than 10>
                 connectionTimeout: <in millisecs>
                 idleTimeout: <in millisecs>
                 maxLifetime: <in millisecs>
   ```
   
1. Database connection properties
   
   - `url`
   - `dialect`
   - `schema`
   
1. Connection pool properties

   - `url`
   - `dialect`
   - `schema`

## Next Step
Configure the Messaging

## Notes and References

