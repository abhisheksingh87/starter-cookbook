+++
categories = ["recipes"]
tags = ["kafka","producer", "consumer", "spring kafka","spring boot"]
summary = "Simple kafka producer and consumer"
title = "Kafka Producer And Consumer Application"
date = "2021-01-30T14:45:27-06:00"
weight = 1
type="kafka"
+++

## CONTEXT
This recipe walks you through the process of how to create kafka producer and consumer application.

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
   
## Configure Kafka Producer 
1. Add below configuration in `src/main/resources/application.yml`

    ```yaml
    spring:
      kafka:
        bootstrap-servers:
          - localhost:9092
        template:
          default-topic: wf-customer
        producer:
          key-serializer: org.apache.kafka.common.serialization.StringSerializer
          value-serializer: org.springframework.kafka.support.serializer.JsonDeserializer
    ```
    * **bootstrap-servers:** are the kafka brokers.  {{% notice note %}}  It is good practice to list all brokers in the cluster. {{% /notice %}}
                                                     
    * **spring.kafka.template.default-topic:** defines the topic to produce messages.     
    * **spring.kafka.producer._key-serializer_:** defines the key serializer for the message. Key is optional but it is recommended to use keys.  
    * **spring.kafka.producer._value-serializer_:** defines the value serializer for the message. The value serializer depends on the type of message 
                                                    you are going to publish to the topic.   
                                                
1. Create new package _producer_. Create new class `CustomerProducer` like below under package _producer_:

    ```java
    import com.wellsfargo.cto.eai.kafka.kafkastarter.model.Customer;
    import lombok.AllArgsConstructor;
    import org.springframework.kafka.core.KafkaTemplate;
    import org.springframework.stereotype.Component;
   
    @Component
    @AllArgsConstructor
    public class CustomerProducer {
    
        private final KafkaTemplate<String, Object> kafkaTemplate;
    
        public void sendData(Customer customer) {
            this.kafkaTemplate.sendDefault(customer.getId(), customer);
        }
    
    }
    ```
   * **KafkaTemplate:** wraps a producer and provides convenience methods to send data to Kafka topics.
   * **sendDefault:** Api requires that a default topic has been provided to the template.
  
## Configure KafkaConsumer

1. Add below configuration in `src/main/resources/application.yml`

    ```yaml
    spring:
      kafka:
        consumer:
          group-id: customer-consumer
          auto-offset-reset: earliest
          key-serializer: org.apache.kafka.common.serialization.StringDeserializer
          value-serializer: org.springframework.kafka.support.serializer.JsonDeserializer
          properties:
             spring.json.trusted.packages: "com.wellsfargo.cto.eai.model"
    ```

    **group.id:** defines the Consumer Group this process is consuming on behalf of.  
    **auto-offset-reset:** If there are 3 partitions and 3 consumers in the group, each consumer will be assigned to a partition. Consumer has to determine if it needs to start consuming the messages from the beginning or starts consuming the new messages.

1. Create new package **consumer** and Add new class `CustomerConsumer` class like below:

    ```java
    import com.wellsfargo.cto.eai.kafkastarter.model.Customer;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.kafka.annotation.KafkaListener;
    import org.springframework.stereotype.Service;
    
    import java.util.concurrent.CountDownLatch;
    
    @Service
    @Slf4j
    @Getter
    public class CustomerConsumer {
    
        private CountDownLatch latch = new CountDownLatch(1);
    
        private Customer payload;
    
        @KafkaListener(topics = "wf-customer")
        public void consume(Customer customer) {
            setPayload(customer);
            log.info("Message: {}", customer);
        }
    
        private void setPayload(Customer customer) {
            this.payload = customer;
        }
        
    }
    ```
    * **@KafkaListener:** annotation provides a mechanism for simple POJO listeners like `CustomerConsumer`.  
    * **Spring Kafka uses** the `@KafkaListener` annotation to listen for consumer messages. 
        Message listening can be divided into two types: single data consumption and batch consumption. The difference is the number of messages that the listener obtains at one time.

## Test Producer and Consumer:

1. Create below test case to test `CustomerConsumer` and `CustomerProducer`

    ```java
    import com.wellsfargo.cto.eai.kafkastarter.KafkaStarterApplication;
    import com.wellsfargo.cto.eai.kafkastarter.model.Customer;
    import com.wellsfargo.cto.eai.kafkastarter.producer.CustomerProducer;
    import org.assertj.core.api.Assertions;
    import org.junit.jupiter.api.Test;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    
    import java.util.concurrent.TimeUnit;
    
    @SpringBootTest(classes = KafkaStarterApplication.class)
    public class CustomerConsumerTest {
    
        @Autowired
        private CustomerProducer customerProducer;
    
        @Autowired
        private CustomerConsumer customerConsumer;
    
        @Test
        public void testKafkaProducerAndConsumer() throws InterruptedException {
    
            //given
            Customer customer = Customer.builder()
                    .firstName("alex")
                    .lastName("smith")
                    .phoneNumber("424645290").build();
            customerProducer.sendData(customer);
            customerConsumer.getLatch().await(10000, TimeUnit.MILLISECONDS);
    
    
            //then
            Assertions.assertThat(customerConsumer.getLatch().getCount()).isEqualTo(1L);
            Assertions.assertThat(customerConsumer.getPayload()).usingRecursiveComparison().isEqualTo(customer);
        }
    }
    ```
    The source code can be found in [WF Github](http://hop.hosting.wellsfargo.com/kafka-starter)
