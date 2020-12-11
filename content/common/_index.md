+++
categories = ["recipes"]
tags = ["starter","spring-boot-app","microservice", "common"]
summary = "How to create a microservice using the greenfield-app-starter"
title = "Create barebone microservice using the starter"
date = 2020-12-09
weight = 10
draft = false
+++

## Context
This is the first recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
The microservice uses Oracle for persistence, Redis for caching, Kafka for messaging and Prometheus for metrics.

Upon completion of this recipe you will have a working spring boot microservice with health, info and  metrics endpoints enabled.

### Prerequisite

- JDK 8 or greater
- IntelliJ or Eclipse IDE
- GIT client
- Access to **greenfield-app-starter** Github repo

## Solution

1. Collect or record the following details for the new microservice

   | Syntax        | Description | Example |
   | :---          |    :----   |  :----   |
   | lob  | line of business | **cto** |
   | business-group | business group within the lob | **eai** |
   | application-group  | application category or grouping | **starter** |
   | application-name      | kasbah or camel cased application name  | **greenfield-starter**
   | application-version    | application version in `major.minor.patch` format; start with `0.1.0` | **0.1.0**
   | description    | short phrase describing the purpose of the application | **_cloud ready_ microservice starter using spring boot 2.3.6**
   | JDK-version  |Java 8 or above; one of `1.8`, `1.11`, `1.12`, `1.13` or `1.14`| **1.12**
   | project-group  | `com.wellsfargo.<lob>.<business-group>.<application-group>` |  **`com.wellsfargo.cto.eai.starter`**

1. Clone the **greenfield-app-starter** from Gitlab repo `git clone <repo url>`

1. Rename folder: `greenfield-app-starter` to `<application-name>`

1. Update the application name in `settings.gradle`

   ```gradle
   rootProject.name = "<application-name>"
   ```

1. Update project details in `build.gradle`
   
   ```gradle
   description = "<description>"
   group = "<project-group>"
   version = "<application-version>"
   sourceCompatibility = "<JDK-version>"
   ```
   
1. Rename package from `com.wellsfargo.cto.eai.starter.greenfield` to `<project-group>.<application-name>`

1. Rename main application classname from `GreenfieldApplication` to `<application-name>Application`

### Validation

1. Open a command window in the `<application-name>` directory

1. Validate the new microservice can be built locally: `gradlew bootJar`

1. Validate the new microservice runs locally: `gradlew bootRun`

1. Verify app health and info in the browser

    - `http://localhost:8080/actuator/health`
    - `http://localhost:8080/actuator/info`

## Next Step
Follow the **Reactive** or **Non-Reactive** recipes depending on your microservice needs. 

## Notes and References