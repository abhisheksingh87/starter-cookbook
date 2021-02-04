+++
categories = ["recipes"]
tags = ["practices", "spring boot", "kafka", "application development", "console", "commands"]
title = "Kafka Console Commands"
description = "Kafka Console Commands"
date = "2021-02-02T22:20:27-06:00"
weight = 6
draft = false
authors = ["Rohan Mukesh"]
+++

## CONTEXT
This recipe walks you through Kafka _console_ commands.

## Topics

1. _create_ a new Kafka topic named `wf-customer-topic` as follows:

    ```shell script
    kafka-topics --create --zookeeper localhost:2181 --replication-factor 1 --partitions 3 --topic wf-customer-topic
    ```

1. _verify_ that the `wf-customer-topic` topic was successfully created by listing all available topics:

    ```shell script
    kafka-topics --list --zookeeper localhost:2181
    ```

1. _add_ more partitions as follows:

    ```shell script
    kafka-topics --zookeeper localhost:2181 --alter --topic wf-customer-topic --partitions 16
    ```

1. _delete_ a topic named `wf-customer-topic` as follows:

    ```shell script
    kafka-topics --zookeeper localhost:2181 --delete --topic wf-customer-topic
    ```

1. _find_ more details about a topic named `wf-customer-topic` as follows:

    ```shell script
    kafka-topics --describe --zookeeper localhost:2181 --topic wf-customer-topic
    ```

1. _view_ _under-replicated_ partitions for all topics as follows:

    ```shell script
    kafka-topics --zookeeper localhost:2181/kafka-cluster --describe --under-replicated-partitions
    ```

## Producers

1. _produce_ messages from standard input as follows:

    ```shell script
    kafka-console-producer --broker-list localhost:9092 --topic wf-customer-topic
    ```

1. _produce_ new messages from an existing file named `messages.txt` as follows:

    ```shell script
    kafka-console-producer --broker-list localhost:9092 --topic test < messages.txt
    ```

1. _produce_ Avro messages as follows:

    ```shell script
    kafka-avro-console-producer --broker-list localhost:9092 --topic wf-customer-topic --property value.schema='{"type":"record","name":"customer","fields":[{"name":"alex","type":"string"}]}' --property schema.registry.url=http://localhost:8081
    ```

## Consumers

## Consume messages

1. _begin_ a consumer from the _beginning_ of the log as follows:

    ```shell script
    kafka-console-consumer --bootstrap-server localhost:9092 --topic wf-customer-topic --from-beginning
    ```

1. _consume_ a _single_ message as follows:

    ```shell script
    kafka-console-consumer --bootstrap-server localhost:9092 --topic wf-customer-topic  --max-messages 1
    ```

1. _consume_ a single message from `__consumer_offsets` as follows:
    * kafka version 0.9.x.x ~ 0.10.x.x*
    ```shell script
    kafka-console-consumer --bootstrap-server localhost:9092 --topic __consumer_offsets --formatter 'kafka.coordinator.GroupMetadataManager$OffsetsMessageFormatter' --max-messages 1
    ```
    * kafka version 0.11.x.x+ *
    ```shell script
    kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic __consumer_offsets --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --max-messages 1
    ```

1. _consume_ and _specify_ a consumer group as follows:

    ```shell script
    kafka-console-consumer --topic wf-customer-topic --new-consumer --bootstrap-server localhost:9092 --consumer-property group.id=wf-customer-group
    ```

## Consume Avro messages

1. _consume 10_ Avro messages from a topic named `wf-customer` as follows:

    ```shell script
    kafka-avro-console-consumer --topic wf-customer --new-consumer --bootstrap-server localhost:9092 --from-beginning --property schema.registry.url=localhost:8081 --max-messages 10
    ```

1. _consume all_ existing Avro messages from a topic named `wf-customer` as follows:

    ```shell script
    kafka-avro-console-consumer --topic wf-customer --new-consumer --bootstrap-server localhost:9092 --from-beginning --property schema.registry.url=localhost:8081
    ```

## Consumers admin operations

1. _list all_ groups as follows:

    ```shell script
    kafka-consumer-groups --new-consumer --list --bootstrap-server localhost:9092
    ```

1. _describe_ a Group named `testgroup` as follows:

    ```shell script
    kafka-consumer-groups --bootstrap-server localhost:9092 --describe --group testgroup
    ```

## Config

1. _set_ the retention for a topic as follows:

    ```shell script
    kafka-configs --zookeeper localhost:2181 --alter --entity-type topics --entity-name wf-cust-topic --add-config retention.ms=3600000
    ``` 

1. _print all_ configuration overrides for a topic named `wf-customer-topic` as follows:
    
    ```shell script
    kafka-configs --zookeeper localhost:2181 --describe --entity-type topics --entity-name wf-customer-topic
    ```

1. _delete_ a configuration override for `retention.ms` for a topic named `wf-customer-topic` as follows:

    ```shell script
    kafka-configs --zookeeper localhost:2181 --alter --entity-type topics --entity-name wf-customer-topic --delete-config retention.ms 
    ```

## ACLs

1. _add_ a new *consumer* ACL to an existing topic as follows:

    ```shell script
    kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:Bob --consumer --topic topicA --group groupA
    ```

1. _add_ a new *producer* ACL to an existing topic as follows:

    ```shell script
    kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --add --allow-principal User:Bob --producer --topic topicA
    ```

1. _list_ the ACLs of a topic named `topicA` as follows:

    ```shell script
    kafka-acls --authorizer-properties zookeeper.connect=localhost:2181 --list --topic topicA
    ```

## Zookeeper 

1. _enter_ the zookeeper shell as follows:

    ```shell script
    zookeeper-shell localhost:2182 ls 
    ```
