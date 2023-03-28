---
layout: post
title:  "Topics, partitions, offsets in Apache Kafka"
date:   2022-04-06 12:16:18 +0300
categories: jekyll update
---

### Introduction

[Apache Kafka](https://kafka.apache.org/) is an open source event streaming platform widely used in the industry.
The development of this project started at LinkedIn, in 2011, to address the scalability challenges this company encountered. 
Today, Kafka is a widely used, and this blog post will be an introduction to Kafka.

### Events, Topics, Partitions, Offsets
In Kafka, records, or messages, are called events. Events are written to topics: a topic is similar to a 
folder, and events to files within that folder. A topic is organized into partitions; a topic can have as many 
partitions as needed. 

An event can be written to a Kafka topic with or without a key. If the event has no key, Kafka will automatically 
and non-deterministic pick a partition where to put the event. If the event has a key, the partition will be determined
based on the key. Thus, if two events are written with the same key, both events are written to the same topic.

Kafka guarantees that a consumer will read events from a partition in the same order as they were written. Thus, events 
having the same key will be retrieved in the same order as they were written.

Each message within a partition gets an incremental id, called offset. An offset has meaning only within the context of a 
single partition. Data written to Kafka cannot be altered. A consumer will retrieve data from Kafka offset by offset.

### An example

To illustrate these concepts, let's run some tests. Kafka provides [documentation](https://kafka.apache.org/quickstart) 
on how to install and run Kafka.

First, create a Kafka topic, with 3 partitions:
```
bin/kafka-topics.sh --bootstrap-server 127.0.0.1:9092 --create --topic demo_topic --partitions 3
```

Next, let's write a producer:
```
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.Properties;

public class ProducerDemo {

    private static final Logger log = LoggerFactory.getLogger(ProducerDemo.class);

    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.setProperty(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092");
        properties.setProperty(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.setProperty(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        KafkaProducer<String, String> producer = new KafkaProducer<>(properties);

        for (int i = 0; i < 50; i++) {
            String topic = "demo_topic";
            String value = "Message nr. " + i;
            String key = "id_" + i;

            ProducerRecord<String, String> producerRecord = new ProducerRecord<>(topic, key, value);

            producer.send(producerRecord, ((metadata, exception) -> {
                if (exception == null) {
                    log.info("Callback writing event: \n" +
                            "Topic: " + metadata.topic() + "\n" +
                            "Key: " + producerRecord.key() + "\n" +
                            "Partition: " + metadata.partition() + "\n" +
                            "Offset: " + metadata.offset() + "\n");
                } else {
                    log.error("Error producing message.", exception);
                }
            }));
        }

        producer.flush();
        producer.close();
    }
}
```

and a consumer:
```
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class ConsummerDemo {

    private static final Logger log = LoggerFactory.getLogger(ConsummerDemo.class);

    public static void main(String[] args) {
        log.info("Consumer started.");

        String bootstrapServers = "127.0.0.1:9092";
        String groupId = "my-second-application";
        String topic = "demo_topic";

        Properties properties = new Properties();
        properties.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        properties.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class.getName());
        properties.setProperty(ConsumerConfig.GROUP_ID_CONFIG, groupId);
        properties.setProperty(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");

        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);

        consumer.subscribe(Collections.singleton(topic));

        while (true) {
            log.info("Polling...");
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));

            for (ConsumerRecord<String, String> record: records) {
                log.info("Received new record " +
                        "key: "+ record.key() + "\n" +
                        "Value: " + record.value() + "\n" +
                        "Partition: " + record.partition() + "\n" +
                        "Offset: " + record.offset());
            }
        }

    }
}
```

We can see, by running the producer multiple times, that messages with the same key end up in the same partition.

### References

https://kafka.apache.org/


