+++
categories = ["recipes"]
tags = ["kafka","producer", "consumer", "spring kafka","spring boot"]
summary = "create kafka producer and consumer"
title = "Create Kafka Producer And Consumer"
date = "2021-01-29T14:45:27-06:00"
weight = 1
+++

## CONTEXT
This recipe walks you through the process of how to create kafka producer and consumer application.

## Prerequisite:

- **STEP 1:** Create Application using Starter is completed.
- **STEP 2:** Java 8 or above installed
- **STEP 3:** Kafka broker (Zookeeper, Server) is setup and running.
- **STEP 4:** Sample Kafka topic (wf-consumer) is created with 1 partition 

## Configure Kafka Producer 

1. Add below configuration in `src/main/resources/application.yml`

    ```yaml
    spring:
      kafka:
        bootstrap-servers:
          - localhost:9092
        template:
          default-topic: wf-consumer
        producer:
          key-serializer: org.apache.kafka.common.serialization.StringSerializer
          value-serializer: org.apache.kafka.common.serialization.JsonSerializer
    ```
**bootstrap-servers:** are the kafka brokers. 
        {{% note  %}}
          It is good practice to list all brokers in the cluster.
        {{% /note %}}
**spring.kafka.template.default-topic:** defines the topic to produce messages. 
**spring.kafka.producer._key-serializer_:** defines the key serializer for the message. Key is optional but it is recommended to use keys.
**spring.kafka.producer._value-serializer_:** defines the value serializer for the message. The value serializer depends on the type of message 
you are going to publish to the topic.

1. Create CustomerProducer class like below:

    ```java
    @Component
    @AllArgsConstructor
    public class CustomerProducer {
    
        private final KafkaTemplate<String, String> kafkaTemplate;
    
        public void sendData(Customer customer) {
            this.kafkaTemplate.sendDefault(customer.getId(), customer);
        }
    
    }
    ``` 

## Configure KafkaConsumer

1. Add below configuration in `src/main/resources/application.yml`

    ```yaml
    spring:
      kafka:
        consumer:
          group-id: customer-consumer
          auto-offset-reset: earliest
          key-serializer: org.apache.kafka.common.serialization.StringDeserializer
          value-serializer: org.apache.kafka.common.serialization.JsonDeserializer
          properties:
             spring.json.trusted.packages: "com.wellsfargo.cto.eai.model"
    ```

 **group.id** defines the Consumer Group this process is consuming on behalf of.
 **auto-offset-reset** If there are 3 partitions and 3 consumers in the group, each consumer will be assigned to a partition. 
 Consumer has to determine if it needs to start consuming the messages from the beginning or starts consuming the new messages.
   

1. Create CustomerConsumer class like below:

```java
@Component
@Slf4j
public class CustomerConsumer {

    @KafkaListener(topics = "wf-customer")
    public void consume(Customer customer) throws UnknownHostException {
        log.info("Message: {}", customer)
    }

}
```
