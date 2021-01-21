+++
categories = ["recipes"]
tags = ["persistence","mongodb","transactions","anti patterns"]
summary = "Spring Data MongoDB Transactions"
title = "6. MongoDB Transactions"
date = 2020-12-09T14:02:27-05:00
weight = 2
+++

## CONTEXT
**MongoDB** supports _multi-document_ ACID transactions from **4.0** release onwards.
This recipe deals with **MongoDB** [Transactions](https://docs.mongodb.com/manual/core/transactions/) and how to use it. 

### Prerequisite

1. **MongoDB 4.0** and above instance is running on the environment.
1. CREATE APPLICATION USING THE STARTER is completed.

## Solution
1. Add below dependency to build.gradle.

     ```groovy
        implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
     ```
1. Register `MongoTransactionManager` in Java Configuration to enable _native_ MongoDB transactions

    ```java
    @Configuration
    @EnableMongoRepositories(basePackages = "com.baeldung.repository")
    public class MongoConfig extends AbstractMongoClientConfiguration {
    
        @Bean
        MongoTransactionManager transactionManager(MongoDatabaseFactory dbFactory) {
            return new MongoTransactionManager(dbFactory);
        }
    }
    ```
1. Add _@Transactional_ annotation to the method below. 

    ```java
    @Test
    @Transactional
    @Order(2)
    public void testMongoDBTransactions() {
    
            Customer customer1 = Customer.builder().firstName("mark")
                                                .lastName("smith")
                                                .phoneNumber("7589756789")
                                                .build();
    
            Customer customer2 = Customer.builder().firstName("ron")
                                                    .lastName("vanderburg")
                                                    .phoneNumber("7589756789")
                                                    .build();
    
            customerRepository.save(customer1);
            customerRepository.save(customer2);
            Query query = new Query().addCriteria(Criteria.where("firstName").is("mark"));
            List<Customer> customers = mongoTemplate.find(query, Customer.class);
            Assertions.assertThat(customers.size()).isEqualTo(1);
    }
    ```
   
1. _listCollections_ and _listIndexes_ are restricted Operations in a Multi-Document Transactions:

    ```java
    @Test
    @Transactional
    @Order(3)
    public void testListCollectionMongoDBTransaction() {
            Customer customer1 = Customer.builder().firstName("mark")
            .lastName("smith")
            .phoneNumber("7589756789")
            .build();
    
            Customer customer2 = Customer.builder().firstName("ron")
            .lastName("smith")
            .phoneNumber("7589756789")
            .build();
    
            assertThrows(MongoTransactionException.class, ()-> {
            if (mongoTemplate.collectionExists(Customer.class)) {
            customerRepository.save(customer1);
            customerRepository.save(customer2);
            }
            });
    }
    ```
1. Spring Data also supports non-native transactions using _TransactionTemplate_.
   we need to set _SessionSynchronization_ to ALWAYS to use non-native Spring Data transactions.

    ```java
        @Test
        public void testTransactionTemplate() {
            mongoTemplate.setSessionSynchronization(SessionSynchronization.ALWAYS);
    
            Customer customer1 = Customer.builder().firstName("mark")
                    .lastName("smith")
                    .phoneNumber("7589756789")
                    .build();
    
            Customer customer2 = Customer.builder().firstName("ron")
                    .lastName("smith")
                    .phoneNumber("7589756789")
                    .build();
            TransactionTemplate transactionTemplate = new TransactionTemplate(mongoTransactionManager);
            transactionTemplate.execute(new TransactionCallbackWithoutResult() {
                @Override
                protected void doInTransactionWithoutResult(TransactionStatus status) {
                    mongoTemplate.insert(customer1);
                    mongoTemplate.insert(customer2);
                }
    
                ;
            });
    
            Query query = new Query().addCriteria(Criteria.where("firstName").is("mark"));
            List<Customer> customers = mongoTemplate.find(query, Customer.class);
    
            Assertions.assertThat(customers.size()).isEqualTo(1);
    }
    ```
## Best Practices:

* Avoid creating long-running Transactions.
* Transaction Run-time limit - Default value is **60** seconds. Transactions should be broken into smaller parts.
* Choosing _Write Concern_ in a distributed multi-sharded cluster. MongoDB allows you to specify the WriteConcern to define the level of durability guarantee when writing to the database.
