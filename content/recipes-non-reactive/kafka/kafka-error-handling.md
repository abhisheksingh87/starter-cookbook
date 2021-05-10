+++
categories = ["recipes"]
tags = ["kafka","topics", "Error handling", "spring kafka","spring boot"]
summary = "Kafka Error Handling"
title = "Kafka Error Handling"
date = "2021-02-01T14:45:27-06:00"
weight = 4
type="kafka"
+++

## CONTEXT
This recipe walks you through the process of how to configure Kafka consumer for Error Handling.

## Prerequisite:

- **STEP 1:** Create Application using Kafka-Starter is completed.
- **STEP 2:** Java 8 or above installed
- **STEP 3:** Kafka broker (Zookeeper, Server) is setup and running.
- **STEP 4:** Sample Kafka topic (wf-consumer) is created with 1 partition 


## Kafka Error Handling

Spring-Kafka provides two types of error handler: 

* The `KafkaListenerErrorHandler` (specified in the **@KafkaListener** annotation) works at the listener level. 
  It is wired into the listener adapter that invokes the @KafkaListener annotation and thus only has access to the current record.

* The second error handler (configured on the listener container) works at the container level and thus has access to the remaining records. 
  The `SeekToCurrentErrorHandler` is a container-level error handler.
  
## Configure KafkaListenerErrorHandler 

1. Spring-Kafka provides `ErrorHandler` Interface, which instance can be passed to `KafkaListenerContainerFactory`.
   Create a new package `errorhandling` and add new class `CustomerConsumerErrorHandler`like below
   
    ```java
    @Service
    @Slf4j
    public class CustomerConsumerErrorHandler implements ConsumerAwareListenerErrorHandler {
    
        @Override
        public Object handleError(Message<?> message, ListenerExecutionFailedException exception, Consumer<?, ?> consumer) {
            log.warn("Error Message: {}, because: {}", message.getPayload(), exception.getMessage());
        }
    }
   ```
   
   
1. Add `CustomerConsumerErrorHandler` handler to consumer

    ```java
    @KafkaListener(topics = "wf-consumer", groupId = "wf-customer", errorHandler = "customerConsumerErrorHandler")
    public void consume(Customer customer) {
        if (customer.getId().equals("23")) {
            throw new RuntimeException("Invalid Message");
        }
        log.info("Message: {}", customer);
    }
    ```
   
1. If there is **exception** while processing record you will see log message like below:

    ```shell script
    Error Message: Customer(id=23, firstName=alex, lastName=smith, phoneNumber=424645290), because: Listener method 'public void com.wellsfargo.cto.eai.kafkastarter.consumer.CustomerErrorConsumer.consume(com.wellsfargo.cto.eai.kafkastarter.model.Customer)' threw exception; nested exception is java.lang.RuntimeException: Invalid Message
    ```   
   
1. If there are multiple Kafka listeners you may opt to create `GlobalErrorHandler` which is configured in the container factory:
   
    ```java
    @Bean
    public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<Integer, String>>
            kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<Integer, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.getContainerProperties().setErrorHandler(new GlobalErrorHandler());
        return factory;
    }
    ```      

## Configure SeekToCurrentErrorHandler  

1. The `SeekToCurrentErrorHandler` discards remaining records from the `poll()` and performs seek operations on the consumer to reset the 
    offsets so that the discarded records are fetched again on the next poll. 
    By default, the error handler tracks the failed record, gives up after **10** delivery attempts and logs the failed record. 
    However, we can also send the failed message to another topic called as **Dead Letter Topic**. Spring-Kafka provides `DeadLetterPublishingRecoverer` class to handler dead letter
    publishing.

1. Add `SeekToCurrentErrorHandler` to `KafkaListenerContainerFactory` like below in `KafkaConfiguration` class:

    ```java
    @Bean
    public ConcurrentKafkaListenerContainerFactory<Object, Object> kafkaListenerContainerFactory(
                    ConcurrentKafkaListenerContainerFactoryConfigurer concurrentKafkaListenerContainerFactoryConfigurer) {
       ConcurrentKafkaListenerContainerFactory<Object, Object> containerFactory = new ConcurrentKafkaListenerContainerFactory<>();
       concurrentKafkaListenerContainerFactoryConfigurer.configure(containerFactory, consumerFactory());
        
       var deadLetterPublishingRecoverer = new DeadLetterPublishingRecoverer(kafkaOperations(), ((consumerRecord, e) ->
             new TopicPartition("wf-consumer-dlt", consumerRecord.partition())));
        
       var errorHandler = new SeekToCurrentErrorHandler(deadLetterPublishingRecoverer, new FixedBackOff(5L, 2L));
             containerFactory.setErrorHandler(errorHandler);
       return containerFactory;
    }     
    ```
    * The dead letter record will be sent to the same partition number as original partition number. {{% notice note %}} Dead Letter Topics must have same partitions as Original {{% /notice%}}

## De-Serialization Errors

1. There is possibility a record has been produced to a Kafka topic and always fails when consumed, no matter how many times it is attempted. You can read more about it 
    [Poison Pill](https://www.confluent.io/blog/spring-kafka-can-your-kafka-consumers-handle-a-poison-pill/). To solve this problem, Spring has introduced `ErrorHandlingDeserializer`
    
1. This deserializer delegates to a real deserializer(key or value). If the delegate fails to deserialize the record content, the ErrorHandlingDeserializer returns a null value 
    and a `DeserializationException` in a header that contains the cause and the raw bytes. When you use a record-level MessageListener, 
    if the ConsumerRecord contains a DeserializationException header for either the key or value, the container’s ErrorHandler is called with the failed ConsumerRecord. 
    The record is not passed to the listener. The consumer offset moves forward so that the consumer can continue consuming the next record

1. Configure `ErrorHandlingDeSerializer` in application.yml like below:

    ```yaml
    spring:
      kafka:
        bootstrap-servers: localhost:9092
        consumer:
          # Configures the Spring Kafka ErrorHandlingDeserializer that delegates to the 'real' deserializers
          key-deserializer: org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
          value-deserializer: org.springframework.kafka.support.serializer.ErrorHandlingDeserializer
        properties:
          # Delegate deserializers
          spring.deserializer.key.delegate.class: org.apache.kafka.common.serialization.StringDeserializer
          spring.deserializer.value.delegate.class: org.springframework.kafka.support.serializer.JsonDeserializer
    ```    
    
    The source code can be found in [WF Github](http://hop.hosting.wellsfargo.com/kafka-starter)
