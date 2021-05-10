+++
categories = ["recipes"]
tags = ["application development", "kafka","topics", "spring kafka","spring boot"]
summary = "Configure Kafka Java Configuration"
title = "Configure Kafka Using Java Configuration"
date = "2021-01-31T14:45:27-06:00"
weight = 3
type="kafka"
+++

## CONTEXT
This recipe walks you through the process of how to configure Kafka Topics, Producer, Consumer using Java Configuration.

## Prerequisite:

- **STEP 1:** Create Application using Kafka-Starter is completed.
- **STEP 2:** Java 8 or above installed
- **STEP 3:** Kafka broker (Zookeeper, Server) is setup and running.

## Configure Topics

1. Spring-Kafka provides `KafkaAdmin` class which can automatically add topics to the broker.
    {{% notice note%}} If you are using Spring Boot, a `KafkaAdmin` class is automatically registered {{% /notice %}}
   
   Create a class `KafkaConfiguration` like below 
    ```java
    @Configuration
    public class KafkaConfiguration {
       
       @Value("${bootstrapServers")
       private String bootStrapServers;
   
        @Bean
        public KafkaAdmin admin() {
            Map<String, Object> configs = new HashMap<>();
            configs.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, bootStrapServers);
            return new KafkaAdmin(configs);
        }
    }
   ```
1. Spring-Kafka provides class `TopicBuilder` and `NewTopic` which can be used to create Topics dynamically like below:

    ```java
    @Value("${partitions}")
    private String partitions;
       
    @Value("${replicas}")
    private String replicas;
       
    @Bean
    public NewTopic wf-customer() {
        return TopicBuilder.name("wf-customer")
                .partitions(partitions)
                .replicas(replicas)
                .compact()
                .build();
    }
    
    @Bean
    public NewTopic wf-account() {
        return TopicBuilder.name("wf-account")
                .partitions(partitions)
                .replicas(replicas)
                .build();
    }
    ```   
    {{%notice note%}} By Default, if the broker is not available, a message is logged, but the context will continue to load.
    You can programmatically, invoke the admin's `initialize()` method to try again later.{{% /notice %}}

## Configure Kafka Producer 


1. Add new method `producerConfigs()` like below in the class `KafkaConfiguration`:

    ```java
    @Value("${wf.kafka.bootstrap-servers}")
    private String bootstrapServers;
    
    @Bean
    public Map<String, Object> producerConfigs() {
      Map<String, Object> props = new HashMap<>();
      props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG,
        bootstrapServers);
      props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG,
        StringSerializer.class);
      props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG,
        JsonSerializer.class);
      return props;
    }
       
    @Bean
    public ProducerFactory<String, String> producerFactory() {
      return new DefaultKafkaProducerFactory<>(producerConfigs());
    }
       
    @Bean
    public KafkaTemplate<String, String> kafkaTemplate() {
      return new KafkaTemplate<>(producerFactory());
    }
    ```
    The above example shows how to configure the Kafka producer to send messages. 
    * **ProducerFactory:** class is responsible for creating Kafka Producer instances.
    * **KafkaTemplate:** is used to send messages to their respective topic.
    * In `producerConfigs()` method we configured below properties:
    
        * **BOOTSTRAP_SERVERS_CONFIG** - Host and port on which Kafka is running.
        * **KEY_SERIALIZER_CLASS_CONFIG** - Serializer class to be used for the key.
        * **VALUE_SERIALIZER_CLASS_CONFIG** - Serializer class to be used for the value. We are using StringSerializer for both keys and values.
        * Additional Properties for the producer to be configured can be found in [ProducerConfig](https://kafka.apache.org/26/javadoc/org/apache/kafka/clients/producer/ProducerConfig.html)
## Configure Kafka Consumer

1. Add new method `consumerConfigs()` in `KafkaConfiguration` class like below:

    ```java
    @Value("${wf.kafka.bootstrap-servers}")
      private String bootstrapServers;
    
      @Bean
      public Map<String, Object> consumerConfigs() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG,
          bootstrapServers);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG,
          StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG,
                 JsonDeserializer.class);
        return props;
      }
    
      @Bean
      public ConsumerFactory<String, String> consumerFactory() {
        return new DefaultKafkaConsumerFactory<>(consumerConfigs());
      }
    
      @Bean
      public KafkaListenerContainerFactory<ConcurrentMessageListenerContainer<String, String>> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
          new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
      }
    ```
    * **KafkaMessageListenerContainer** receives all messages from all topics on a single thread.
    * **ConcurrentKafkaListenerContainerFactory:**  is used to create containers for methods annotated with **@KafkaListener**. 
        The KafkaListenerContainer receives all the messages from all topics or partitions on a single thread. 
    * In `consumerConfigs()` method we configured below properties:
        
         * **BOOTSTRAP_SERVERS_CONFIG** - Host and port on which Kafka is running.
         * **KEY_DESERIALIZER_CLASS_CONFIG** - DeSerializer class to be used for the key.
         * **VALUE_DESERIALIZER_CLASS_CONFIG** - DeSerializer class to be used for the value. We are using `JsonDeSerializer` values.
         * Additional Properties for the consumer to be configured can be found in [ConsumerConfig](https://kafka.apache.org/26/javadoc/org/apache/kafka/clients/consumer/ConsumerConfig.html)

    The source code can be found in [WF Github](http://hop.hosting.wellsfargo.com/kafka-starter)
    

