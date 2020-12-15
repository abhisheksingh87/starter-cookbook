+++
categories = ["recipes"]
tags = ["starter","microservice", "barebone microservice"]
summary = "How to create a microservice using the greenfield-app-starter"
title = "Create microservice using the starter"
date = 2020-12-09
weight = 10
draft = false
+++

## Context
This recipe is part of a cookbook for developing a modern _cloud ready_ microservice using the **greenfield-app-starter**. Upon completion of this recipe you will have a _working_ spring boot microservice with health, info and  metrics endpoints enabled.

### Prerequisite

- JDK 8 or greater
- IntelliJ or Eclipse IDE
- GIT client
- Access to **greenfield-app-starter** Github repo

## Solution

1. Collect or record the following details for the new microservice

   | Property        | Description | Example |
   | :---          |    :----   |  :----   |
   | lob  | line of business | **consumer** |
   | business-group | business group within the lob | **lending** |
   | application-group  | application category or grouping | **loan** |
   | microservice-name      | camelCased name  | **AutoLoanCalculator**
   | microservice-version    | in `major.minor.patch` format; start with `0.1.0` | **0.1.0**
   | description    | short phrase describing the purpose of the microservice | **consumer auto loan calculator for period less than 36 months**
   | JDK-version  |Java 8 or above; one of `1.8`, `1.11`, `1.12`, `1.13` or `1.14`| **1.12**
   | project-group  | `com.wellsfargo.<lob>.<business-group>.<application-group>` |  **`com.wellsfargo.consumer.lending.loan`**

1. Clone the **greenfield-app-starter** from Gitlab repo `git clone <repo url>`
   
![starter app folder structure](/images/resized.jpg "microservice folder structure")
1. Rename folder: `greenfield-app-starter` to `<microservice-name>`  
   (example: **AutoLoanCalculator**)

1. Update the microservice name in `settings.gradle`

   ```gradle
   rootProject.name = "<microservice-name>"
   ```

   ```gradle
   # EXAMPLE
   
   rootProject.name = "AutoLoanCalculator"
   ```   


1. Update project details in `build.gradle`
   
   ```gradle
   description = "<description>"
   group = "<project-group>"
   version = "<microservice-version>"
   sourceCompatibility = "<JDK-version>"
   ```

    ```gradle
   # EXAMPLE
   
   description = "consumer auto loan calculator for period less than 36 months"
   group = "com.wellsfargo.consumer.lending.loan"
   version = "0.1.0"
   sourceCompatibility = "1.12"
   ```
   
1. Rename _package_ from `com.wellsfargo.cto.eai.starter` to `<project-group>`  
   (example: **`com.wellsfargo.consumer.lending.loan`**)

1. Rename _main application classname_ from `GreenfieldMicroservice` to `<microservice-name>`  
   (example: **`AutoLoanCalculator`**)

1. As an example, the _barebone_ microservice will have the following:
   
   * folder: `AutoLoanCalculator` containing
      * `com.wellsfargo.consumer.lending.loan.AutoLoanCalculator.java`
      * `src/main/resources/application.yml`
      * `build.gradle`
      * `settings.gradle`
   

1. Create the codebase folder structure based on the\<recipe\> in ***Best Practices***

### Validation

1. Open a command window in the `<microservice-name>` directory

1. Validate the new microservice can be built locally: `gradlew bootJar`

1. Validate the new microservice runs locally: `gradlew bootRun`

1. Verify microservice health and info in the browser

    - `http://localhost:8080/actuator/health`
    - `http://localhost:8080/actuator/info`

## Next Step
Follow the **Reactive** or **Non-Reactive** recipes depending on your microservice needs. 

## Notes and References