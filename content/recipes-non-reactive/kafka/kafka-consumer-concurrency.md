+++
categories = ["recipes"]
tags = ["kafka","producer", "consumer", "concurrent", "spring kafka","spring boot"]
summary = "Configure Multiple Consumers For Topic"
title = "Configure Multiple Consumers For Topic"
date = "2021-01-29T14:45:27-06:00"
weight = 2
type="kafka"
+++

## CONTEXT

Multiple consumers in kafka depends on how many `Partitions` a topic has. Let us assume a Topic has 3 Partitions.
If a consumer takes more than 2 seconds to process each message, it is possible that big chunk of message will remain
on the topic. This may slow down the process as well. A solution for this is to have multiple concurrent
consumers for the topic.

This recipe walks you through the process of how to create multiple consumers for a topic.

## Prerequisite:

- **STEP 1:** Create Application using Starter is completed.
- **STEP 2:** Java 8 or above installed
- **STEP 3:** Kafka broker (Zookeeper, Server) is setup and running.
- **STEP 4:** Sample Kafka topic (wf-customer) is created with 1 partition 

## Create Model:

1. Create new package _model_. Add new class `Customer` like below:
    ```java
    @Builder
    @ToString
    @AllArgsConstructor
    @NoArgsConstructor
    @Getter
    public class Customer {
    
        private String firstName;
        private String lastName;
        private String phoneNumbers;
    }
   ```
## Configure Multiple Kafka Consumers For a Topic

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

    **group.id:** defines the Consumer Group this process is consuming on behalf of.  
    **auto-offset-reset:** If there are 3 partitions and 3 consumers in the group, each consumer will be assigned to a partition. Consumer has to determine if it needs to start consuming the messages from the beginning or starts consuming the new messages.

1. Create new package **consumer** and Add new class `ConcurrentCustomerConsumer` class like below:

    ```java
    import com.wellsfargo.cto.eai.kafkastarter.model.Customer;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.kafka.annotation.KafkaListener;
    import org.springframework.stereotype.Service;
    
    import java.util.concurrent.CountDownLatch;
    
    @Service
    @Slf4j
    @Getter
    public class ConcurrentCustomerConsumer {
    
        private CountDownLatch latch = new CountDownLatch(1);
    
        private Customer payload;
    
        @KafkaListener(topics = "wf-consumer", concurrency = "3")
        public void consume(ConsumerRecord<String, Object> consumerRecord) {
            setPayload(consumerRecord.value());
            log.info("Partition: {}, Offset: {}, Message: {}", consumerRecord.partition(), consumerRecord.offset(), consumerRecord.value());
        }
    
        private void setPayload(Customer customer) {
            this.payload = customer;
        }
        
    }
    ```
    * **@KafkaListener:** annotation provides a mechanism for simple POJO listeners like `CustomerConsumer`. 
    * **concurrency:** # Number of threads to run in the listener containers. 
    * **ConsumerRecord** is Java representation of kafka message. We can get several properties by using class `ConsumerRecord`, including `key`, `partition`, 
        and the data itself.
    
## Test Producer and Consumer:

1. Create below test case to test `ConcurrentCustomerConsumer`.

    ```java
    package com.wellsfargo.cto.eai.kafkastarter.consumer;

    import com.wellsfargo.cto.eai.kafkastarter.KafkaStarterApplication;
    import com.wellsfargo.cto.eai.kafkastarter.model.Customer;
    import com.wellsfargo.cto.eai.kafkastarter.producer.CustomerProducer;
    import org.assertj.core.api.Assertions;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    
    import java.util.concurrent.TimeUnit;

    @SpringBootTest(classes = KafkaStarterApplication.class)
    public class ConcurrentCustomerConsumerTest {
    
        @Autowired
        private CustomerProducer customerProducer;
    
        @Autowired
        private ConcurrentCustomerConsumer concurrentCustomerConsumer;
    
        @Test
        public void testMultipleConsumers() throws InterruptedException {
    
            //given
            Customer customer1 = Customer.builder()
                    .id("1")
                    .firstName("alex")
                    .lastName("smith")
                    .phoneNumber("424645290").build();
    
            Customer customer2 = Customer.builder()
                    .id("2")
                    .firstName("ron")
                    .lastName("stewart")
                    .phoneNumber("424645290").build();
            customerProducer.sendData(customer1);
            customerProducer.sendData(customer2);
            concurrentCustomerConsumer.getLatch().await(10000, TimeUnit.MILLISECONDS);
    
    
            //then
            Assertions.assertThat(concurrentCustomerConsumer.getPartitions().size()).isEqualTo(2);
            Assertions.assertThat(concurrentCustomerConsumer.getPartitions()).contains(0, 1);
        }
    
    }
    ```
    The source code can be found in [WF Github](http://hop.hosting.wellsfargo.com/kafka-starter)
