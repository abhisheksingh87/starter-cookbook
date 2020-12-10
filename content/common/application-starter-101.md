+++
categories = ["recipes","common"]
tags = ["starter","spring-boot-app","microservice"]
summary = "How to create a microservice using the greenfield-app-starter"
title = "Step 1: Create application using the starter"
date = 2020-12-09T14:02:27-05:00

+++

## Contexts
This is the first recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
The microservice uses Oracle for persistence, Redis for caching, Kafka for messaging and Prometheus for metrics. 

### Prerequisite

- JDK 8 or greater
- Access to **greenfield-app-starter** Github repo

## Solution

1. Clone the **greenfield-app-starter** from Gitlab repo `git clone <repo url>`

1. Update the following in `build.gradle`

   ```
   description = "<description>"
   group = "<package name>"
   version = "<major.minor.patch>"
   sourceCompatibility = "<jdk version in 1.X format>"
   ```
   **NOTE:** `sourceCompatibility` should be JDK 8 or higher ie. `1.8`

1. Update the following in `settings.gradle`

   ```
   rootProject.name = "<application-name>"
   ```
   
1. Refactor java package name from `com.wellsfargo.cto.eai.starter.greenfield` to `<your package name>`

1. Change the main application name from `com.wellsfargo.cto.eai.starter.greenfield.GreenfieldApplication` to `com.wellsfargo.<lob>.<group>.<app-name>`

   - `com.wellsfargo.<lob>.<group>.<app-name>`
   - **TODO** preferred standard package structure
   - **TODO** testing and TDD strategy

1. Open a command window and navigate to the `greenfield-app-starter` directory
   
1. Validate the starter runs locally: `gradlew bootRun`
   
1. Verify app health and info in the browser 
   
   - `http://localhost:8080/actuator/health`  
   - `http://localhost:8080/actuator/info`  

## Next Step
Configure the Persistence

## Notes and References

