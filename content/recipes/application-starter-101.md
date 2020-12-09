+++
categories = ["recipes"]
tags = ["starter","spring-boot-app","microservice"]
summary = "How to create a microservice using the greenfield-app-starter"
title = "Step 1: Create application using the starter"
date = 2020-12-09T14:02:27-05:00

+++

## Context
This is the first recipe in the series, for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**.  
The microservice uses Oracle for persistence, Redis for caching, Kafka for messaging and Prometheus for metrics. 

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
Configure the Persistence

## Notes and References

