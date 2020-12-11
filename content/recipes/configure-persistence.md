+++
categories = ["recipes"]
tags = ["persistence","hikari","database connection pool","anti patterns"]
summary = "Configure persistence in microservice"
title = "Step 2: Configure Persistence in microservice"
date = 2020-12-09T14:02:27-05:00

+++

## Context
This is the second recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
This recipe deals with configuring persistence in the microservice.  
The `HikariDatasource` bean with preconfigured connection pool is used to fulfill persistence needs in the microservice.

### Prerequisite

- **STEP 1: CREATE APPLICATION USING THE STARTER** is completed.

## Solution

1. Determine and record the following **oracle database connection details** for the new microservice

   | Property        | Details  |
      | :---          |    :----   | 
   | jdbc url  |  `jdbc:oracle:thin:@<host>:<port>:<schema>` OR `jdbc:oracle:oci:@<host>:<port>:<schema>` |
   | driver classname | `oracle.jdbc.OracleDriver`    |
   | schema     | database schema name  | 
   | dialect    | Hibernate dialect based on the Oracle database version `org.hibernate.dialect.Oracle12cDialect` | 
   | username | schema username | 
   | password | schema password | 

1. Determine and record the **database connection pool requirements** for the new microservice

   | Property        | Details  |
         | :---          |    :----   | 
   | jdbc url  |  `jdbc:oracle:thin:@<host>:<port>:<schema>` OR `jdbc:oracle:oci:@<host>:<port>:<schema>` |
   | driver classname | `oracle.jdbc.OracleDriver`    |
   | schema     | database schema name  | 
   | dialect    | Hibernate dialect based on the Oracle database version `org.hibernate.dialect.Oracle12cDialect` | 
   | username | schema username | 
   | password | schema password | 


 
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

### Validation

1. Step 1

1. Step 2

## Next Step
Configure the Cache


## Notes and References

### Anti Patterns
1. Oversizing the `maximumPoolSize`
1. Having too many _idle_ connections
1. Setting high `maxLifetime`

#### Hikari Connection Pool configuration
