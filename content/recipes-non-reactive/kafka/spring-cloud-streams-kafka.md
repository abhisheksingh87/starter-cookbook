+++
categories = ["recipes"]
tags = ["kafka","topics", "spring cloud stream", "spring kafka","spring boot"]
summary = "Spring Cloud Streams Kafka"
title = "Spring Cloud Streams Kafka"
date = "2021-02-04T09:45:27-06:00"
weight = 101
type="kafka"
+++

## CONTEXT
This recipe walks you through the process of how to configure spring cloud streams with Kafka

## Prerequisites


- **STEP 1:** Create Application using Spring-Cloud-Stream-Starter is completed.
- **STEP 2:** Java 8 or above installed
- **STEP 3:** Kafka broker (Zookeeper, Server) is setup and running.
- **STEP 4:** Sample Kafka topic (wf-customer) is created with 1 partition


## Spring Cloud Stream
Spring Cloud Stream provides a number of abstractions and primitives that simplify the writing of message-driven microservice applications. 
It provides messaging tier specific **Binder** implementation which abstracts away all connectivity details to your middleware. 
Deployers can dynamically choose at runtime, the destination to which the channels connect. 
![](/images/scs.png)


### Define the Kafka streams

In order for our application to be able to communicate with Kafka, we'll need to define an outbound stream to write 
messages to a Kafka topic, and an inbound stream to read messages from a Kafka topic.

Spring Cloud provides a convenient way to do this by simply creating an interface that defines a separate method 
for each stream.

```java
public interface CustomerStreams {

    String INPUT = "customer-in";
    String OUTPUT = "customer-out";

    @Input(INPUT)
    SubscribableChannel inboundCustomer();

    @Output(OUTPUT)
    MessageChannel outboundCustomer();
    
}
```

The ```inboundCustomer()``` method defines the inbound stream to read from Kafka and ```outboundCustomer()``` method defines 
the outbound stream to write to Kafka.

During runtime Spring will create a java proxy based implementation of the ```CustomerStreams``` interface that 
can be injected as a Spring Bean anywhere in the code to access our two streams. 

### Configure Spring Cloud Stream

Our next step is to configure Spring Cloud Stream to bind to our streams in the ```CustomerStreams``` interface. 

This can be done by creating a ```@Configuration``` class ```com.wellsfargo.cto.eai.scs.config.StreamsConfig``` with 
below code:

```java
@EnableBinding(CustomerStreams.class)
public class StreamsConfig {
}
```

Binding the streams is done using the ```@EnableBinding``` annotation where the ```CustomerService``` interface is passed to.

### Configuration properties for Kafka

By default, the configuration properties are stored in the ```src/main/resources/application.yaml``` file.

```yaml
spring:
  cloud:
    stream:
      kafka:
        binder:
          brokers: ${broker}
          auto-create-topics: false
          configuration:
            auto.offset.reset: latest
      bindings:
        customer-in:
          destination: wf-customer
          group: customer-in-group
          contentType: application/json
        customer-out:
          destination: wf-customer
          contentType: application/json
```          
{{% notice note %}}
The above configuration properties configure the address of the Kafka server to connect to, and the Kafka topic we use 
for both the inbound and outbound streams in our code. They both must use the same Kafka topic!{{% /notice %}}

The ```contentType``` properties tell Spring Cloud Stream to send/receive our message objects as ```json```s in the streams.

## Create Model:

Create new package _model_. Add new class `Customer` like below:
```java
@Builder
@ToString
@AllArgsConstructor
@NoArgsConstructor
@Getter
public class Customer {
    
   private UUID customerId;
   private String firstName;
   private String lastName;
   private String social;
   private String income;
}    
```

### Create service layer 

Create ```CustomerService``` class with like below:

```java
@Service
@Slf4j
public class CustomerService {

    @Autowired
    private CustomerStreams customerStreams;

    public void sendCustomer(final Customer customer) {
        log.info("Sending customer {}", customer);

        MessageChannel messageChannel = customerStreams.outboundCustomer();
        boolean sent = messageChannel.send(MessageBuilder
                .withPayload(customer)
                .setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.APPLICATION_JSON)
                .build());

        log.info("Sent {} customer {}", sent, customer);
    }

}
```

The ```@Slf4j``` annotation will generate an SLF4J logger field that we can use for logging.

In the ```sendCustomer()``` method we use the injected ```CustomerStream``` object to send a message represented by the ```Customer``` object.

### Create Customer Consumer

Create a  class ```CustomerListener``` class that will listen to messages on the ```wf-customer``` Kafka topic and log them on the console:

```java
@Component
@Slf4j
public class CustomerListener {

    @StreamListener(CustomerStreams.INPUT)
    public void handleCustomer(@Payload Customer customer, @Headers Map<String, Object> headers) {
        log.info("Received customer: {}. Partition: {}. Offset: {}", customer,
                headers.get(KafkaHeaders.RECEIVED_PARTITION_ID), headers.get(KafkaHeaders.OFFSET));
    }

}
```

{{%notice note%}}
```CustomerListener``` has a single method, ```handleCustomer()``` that will be invoked by Spring Cloud Stream with 
every new ```Customer``` message object on the ```wf-customer``` Kafka topic. This is thanks to the ```@StreamListener``` annotation 
configured for the ```handleCustomer()``` method. {{%/notice%}}

```@Headers``` annotation inject Kafka record headers from the Kafka Topic. This map includes additional information from
the record as: partition id, offset etc.

### Create REST API

1. Create a REST API endpoint that will trigger sending a message to Kafka using the ```CustomerService``` Spring Bean:

    ```java
    @RestController
    public class CustomerController {
    
        @Autowired
        private CustomerService customerService;
    
        @GetMapping("/customer")
        @ResponseStatus(HttpStatus.ACCEPTED)
        public ResponseEntity<Customer> customer(@RequestBody Customer Customer) {
            customerService.sendCustomer(customer);
            return ResponseEntity.ok(customer);
        }
    
    }
    ```

### Running Application

1. Use a rest client to send _Customer_ object like below:

    ```json
    {
        "firstName": "alex",
        "lastName": "smith",
        "social": "180-00-000",
        "income": "100000"
    }
    ```

1. You should see below console output:

    ```shell script
    [ctor-http-nio-3] c.w.c.e.s.c.s.service.CustomerService    : Sent true customer Customer(customerId=267b68c9-f0fc-49e7-826f-f2bc5e36d0f2, firstName=alex, lastName=smith, social=180-00-000, income=100000)
    [container-0-C-1] c.w.c.e.s.c.s.service.CustomerListener   : Received customer: Customer(customerId=267b68c9-f0fc-49e7-826f-f2bc5e36d0f2, firstName=alex, lastName=smith, social=180-00-000, income=100000). Partition: 0. Offset: 2
    ```

The source code is available at: [WF Github](http://hop.hosting.wellsfargo.com/spring-cloud-stream-kafka)
