+++
categories = ["recipes"]
tags = ["kafka","topics", "Error handling", "spring kafka","spring boot"]
summary = "Kafka Consumer Retry"
title = "Kafka Consumer Retry"
date = "2021-02-03T11:18:27-06:00"
weight = 4
+++

## CONTEXT
This recipe walks you through the process of how to configure kafka consumer retry

## Prerequisite:

- **STEP 1:** Create Application using Kafka-Starter is completed.
- **STEP 2:** Java 8 or above installed
- **STEP 3:** Kafka broker (Zookeeper, Server) is setup and running.
- **STEP 4:** Sample Kafka topic (wf-customer) is created with 1 partition 


## Kafka Consumer Retry

Spring-Kafka provides `AbstractKafkaListenerContainerFactory` to configure the **Spring-Retry** template. 
Using the Spring-Retry template you can set the number of _retries_ and _backoff time_ (after how many ms next retry should be started).

## Configure Retry Template 

1. Configure `RetryTemplate` in `KafkaConfiguration` class like below
   
    ```java
    public RetryTemplate retryTemplate() {
        var retryTemplate = new RetryTemplate();
        var retryPolicy = new SimpleRetryPolicy(retryAttemps);
        retryTemplate.setRetryPolicy(retryPolicy);
        FixedBackOffPolicy fixedBackOffPolicy = new FixedBackOffPolicy();
        fixedBackOffPolicy.setBackOffPeriod(backOffPeriod);
        retryTemplate.setBackOffPolicy(fixedBackOffPolicy);
        return retryTemplate;
    }
   ```
1. Add retryPolicy and backOffPeriod in `src/main/resources/application.yml`

    ```yaml
    retryAttempts: 3
    backOffPeriod: 5000
    ```
   
   In the above configuration:
   * **retry** will be executed **3** times in span of **5000ms**.
   
## Configure KafkaListenerContainerFactory:

1. Add `Bean` method to configure `KafkaListenerContainerFactory` to be used by **@KafkaListener** annotation

    ```java
    @Bean
    public ConcurrentKafkaListenerContainerFactory<Object, Object> retryContainerFactory(
                ConcurrentKafkaListenerContainerFactoryConfigurer concurrentKafkaListenerContainerFactoryConfigurer) {        ConcurrentKafkaListenerContainerFactory<Object, Object> containerFactory = new ConcurrentKafkaListenerContainerFactory<>();
         concurrentKafkaListenerContainerFactoryConfigurer.configure(containerFactory, consumerFactory());
         containerFactory.setRetryTemplate(createRetryTemplate());
         return containerFactory;
    }
    ```
## Add ContainerFactory to Listener:

   ```java
    @KafkaListener(topics = "wf-customer", groupId = "retry-customer", containerFactory = "retryContainerFactory", errorHandler = "customerConsumerErrorHandler")
    public void consume(Customer customer) {
            log.info("Message: {}", customer);
    }
   ```
    
   The source code can be found in [WF Github]()
