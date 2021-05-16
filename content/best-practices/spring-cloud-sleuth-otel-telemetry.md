+++
date = "2021-05-13T14:02:27-05:00"
title = "Spring Cloud Sleuth Open Telemetry"
summary = "Spring cloud Sleuth Otel Telemetry."
tags = ["application development", "logging", "open telemetry", "sleuth", "spring-cloud"]
weight = 7
+++

## Context
Spring Cloud Sleuth provides Spring Boot auto-configuration for distributed tracing.
It introduces unique IDs to logging which are consistent between microservice calls which makes it possible to find how a single request travels from one microservice to the next.

Spring Cloud Sleuth adds two types of IDs to your logging, one called a trace ID and the other called a span ID. The span ID represents a basic unit of work, for example sending an HTTP request. The trace ID contains a set of span IDs, forming a tree-like structure. 
The trace ID will remain the same as one microservice calls the next

## Setup Spring Cloud Sleuth With Open Telemetry:

Add `Sleuth Starter`, exclude `Brave` and add `spring-cloud-sleuth-otel-autoconfigure` dependency to build.gradle. 
As OpenTelemetry is not **GA** yet we `spring-cloud-sleuth-otel` is not **GA**. We will have to rely on **milestone**
repositories.   

```groovy
ext {
    set('springCloudVersion', "2020.0.2")
    set('springCloudSleuthOtelVersion', "1.0.0-M7")
}

dependencies {
    implementation('org.springframework.cloud:spring-cloud-starter-sleuth') {
        exclude group: 'org.springframework.cloud', module: 'spring-cloud-sleuth-brave'
    }
    implementation 'org.springframework.cloud:spring-cloud-sleuth-otel-autoconfigure'
}

dependencyManagement {
  imports {
    mavenBom "org.springframework.cloud:spring-cloud-sleuth-otel-dependencies:${springCloudSleuthOtelVersion}"
   }
}
repositories {
    maven {
        url "https://repo.spring.io/milestone"
    }
}
```

## Create Rest Controller like below

```java
@RestController
public class CustomerController {
    private static Logger log = LoggerFactory.getLogger(CustomerController.class);

    @RequestMapping("/customers")
    public String getAllCustomers() {
        log.info("retrieve all customers!");
        return Arrays.asList(Customer.builder()
                .customerId(id)
                .firstName("alex")
                .lastName("smith")
                .build());
    }
}
```

## Running Application
Use a rest client to send **GET** request `/customers`
```shell
2020-10-21 12:01:16.285  INFO [customer-service,0b6aaf642574edd3,0b6aaf642574edd3,true] 289589 --- [nio-9000-exec-1] CustomerController : retrieve all customers!
```
The portion of the log statement that Sleuth adds is [customer-service,0b6aaf642574edd3,0b6aaf642574edd3,false].
* The first part is the application name (spring.application.name=customer-service in **bootstrap.yml**). 
* The second value is the trace id. 
* The third value is the span id. 

### How Much Data is Enough?
Which requests should be traced? Ideally, you’ll want enough data to see trends reflective of live, operational traffic. You don’t want to overwhelm your logging and analysis infrastructure, though. Some organizations may only keep requests for every thousand requests, or every ten, or every million!
By default, the threshold is **10%* *, or **.1**, though you may override it by specifying a sampling probability:

```properties
spring.sleuth.sampler.probability=1.0
```

Alternatively, you may register your own `Sampler` bean definition and programmatically make the decision 
which requests should be sampled. You can make the choice about which requests to trace, for example, 
by ignoring successful requests, perhaps checking whether some component is in an error state, or really anything else.
