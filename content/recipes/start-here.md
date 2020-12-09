+++
categories = ["recipes"]
tags = ["starter","spring-boot-app","microservice"]
summary = "How to use the greenfield starter to create a microservice"
title = "How to use the greenfield starter"
date = 2020-12-09T14:02:27-05:00

+++

## Context
This is the first recipe in the series, that is used to develop a modern microservice deployable in the cloud using the **greenfield-app-starter**.

### Prerequisite

- JDK 8 or greater
- Access to **greenfield-app-starter** Github repo

## Solution

1. Clone the **greenfield-app-starter** from Gitlab repo `git clone <repo url>`
   
1. Open a command window and navigate to the `greenfield-app-starter` directory
   
1. Validate the starter runs locally: `gradlew bootRun`
   
1. Verify app health and info in the browser 
   
   - `http://localhost:8080/actuator/health`  
   - `http://localhost:8080/actuator/info`  

## Next Step
Configure the Persistence (database) module

## Notes and References

