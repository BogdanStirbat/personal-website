---
layout: post
title:  "Exchanges, bindings and queues in RabbitMQ"
date:   2022-03-31 12:16:18 +0300
categories: jekyll update
---

### Introduction

[RabbitMQ](https://www.rabbitmq.com) is one of the most used open source message broker. In this blog post I will make 
a quick briefing of its architecture.

### The AMQP 0-9-1 model

The [AMQP 0-9-1](https://www.rabbitmq.com/tutorials/amqp-concepts.html) protocol is one of the protocols used by RabbitMQ. 
It defines the following items:
 - exchanges; an exchange is an entity where messages are sent to
 - bindings; a binding is an entity that routes a message to a queue
 - queues; queues store messages that are consumed by applications (consumers)

There different type of exchanges:
 - direct exchanges; a direct exchange routes messages to a queue based on a routing key; ideally, direct exchanges are 
   used when we want a message to be propagated to only one queue
 - fanout exchanges; a fanout exchange routes a message to all the queues bound to it, regardless of the routing key
 - topic exganges; a topic exchange routes a message to one or more queues, based on the matching between the message 
   routing key and the pattern that was used to bind a queue to the exchange
 - header exchanges; a header exchange routes a message on multiple attributes that can be expressed as headers
   
### Testing these concepts

To test these concepts, we can very easily start an RabbitMQ broker and play with what we've learned so far.

We can start a RabbitMQ instance by running:

```
docker run --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.9-management
```

Next, we can open, in browser, a RabbitMQ console by accessing [http://localhost:15672](http://localhost:15672) with
username guest and password guest.

We should be able to see a menu similar to this one:

![Project structure]({{ "/assets/2022-03-31-exchanges-bindings-queues-rmq/1_RabbitMQ_menu.png" | absolute_url }})

We can see, in the top menu, Exchanges and Queues. 

First, let's create the following queues, leaving all other arguments the default ones:
 - test.queue.1
 - test.queue.2
 - test.queue.3

![Project structure]({{ "/assets/2022-03-31-exchanges-bindings-queues-rmq/2_queues.png" | absolute_url }})

Next, let's create the following exchanges:
 - test.exchange.direct, a direct exchange
 - test.exchange.fanout, a fanout exchange
 - test.exchange.topic, a topic exchange

![Project structure]({{ "/assets/2022-03-31-exchanges-bindings-queues-rmq/3_exchanges.png" | absolute_url }})

Now, for each exchange, we will have to define bindings to queues. For each exchange, after bindings are created, we will 
create some messages.

#### The topic exchange

Let's create the following bindings:

![Project structure]({{ "/assets/2022-03-31-exchanges-bindings-queues-rmq/4_topic_exchange.png" | absolute_url }})

The important thing to notice is that each routing key is a pattern. If we send a message with routing key "a1.a1.a" or 
"a2.a2.a", the message ends up in test.queue.1 and test.queue.2; if we send a message with routing key "a1.a2.b", it ends up
in test.queue.3 .

#### The fanout exchange

Let's create the following bindings:

![Project structure]({{ "/assets/2022-03-31-exchanges-bindings-queues-rmq/5_fanout_exchange.png" | absolute_url }})

We can see that, no matter what routing key we use to publish a message, the message ends up in all the queues

#### The direct exchange

![Project structure]({{ "/assets/2022-03-31-exchanges-bindings-queues-rmq/6_direct_exchange.png" | absolute_url }})

Now, if we publish a message with the routing key "a" it ends up in test.queue.1 and test.queue.2; if we publish a message 
with the routing key "b", it ends up in test.queue.3

### Conclusions

Depending on the type of exchange, we can either send a message to one single queue, or to multiple queues, or to 
broadcast messages. We can use a fanout exchange for broadcasting, a topic exchange to route to multiple queues depending on 
a routing key pattern, or a direct exchange to send a message to a single queue. Indeed, we can use a direct exchange
for sending a message to multiple queues, but it's better to use a topic exchange for this scenario.

### References

https://www.rabbitmq.com/tutorials/amqp-concepts.html

https://www.baeldung.com/spring-amqp

