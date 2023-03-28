---
layout: post
title:  "High performance I/O bound applications in Java"
date:   2017-10-26 22:25:18 +0300
categories: jekyll update
---


In this blog post, I will discuss about I/O bound applications, and their performance.
What are I/O bound applications, and what makes them different from CPU bound applications? According to Wikipedia, [I/O bound](https://en.wikipedia.org/wiki/I/O_bound) could go faster if period waiting for data is shortened. As you probably guess, [CPU bound](https://en.wikipedia.org/wiki/CPU-bound) applications go faster if the central processor makes computations faster.

Examples of CPU bound applications: training a neural network, running a physics simulation; in general, data is readily available or inexpensive to retrieve, and intensive computations are required.
For I/O bound applications, the opposite is true: we don't need too expensive computations, but we wait for data quite a lot. Examples of I/O bound applications include: web applications, when we usually wait for the database to return some results.

If we need to make a web application faster, a trivial solution is to add new computers to a cluster, so more computational power will be used (horizontal scaling). This is not a bad solution. To search for a better solution, let's understand what a machine does to process a request. A typical model, made popular in the 90's with [Apache web server](https://httpd.apache.org/), one of the most popular HTTP web server, is the thread per request model. Each new request will be served in a separate thread. This thread will wait for database, possible other HTTP web services, then will give the result to the caller. Each new thread brings overhead: we add pressure on the kernel, as context switch is more difficult, each new thread requires memory for the call stack (on a 64-Bit machine, about 1MB). In general, we cannot have more than about 10000, maybe 20000 threads running on a system. So, if we deploy a web server and it's becomes very successful, many of our users will experience timeouts. And, in all of this time, we have a powerful CPU, but is sits idle most of the time waiting for data.

There is nothing wrong with the thread for request model. It was a successful model, and many successful applications are using it. It's a simple, intuitive, model. But it has a problem, it doesn't scales well. For a certain class of applications, like Facebook, Twitter, chat applications, this is the wrong model to use. To move forward, two solutions emerged: event loops, and M:N threading.

### Event loops

The event loop pattern emerged in the 2000's. In 2004, the [Nginx](https://www.nginx.com/) web server saw it's initial release. As described [by the team](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/), this web server was designed having performance in mind. In 2009, [Node.js](https://nodejs.org/en/), a web server that made history, was released. Node.js uses an event-driven, non-blocking I/O model that makes it highly scalable.
In an Node.js application, data access is no longer blocking, but asynchronous. When an external source of data is required, the user will call a function, and will specify (among other parameters) a callback. When the external source of data produced the desired information, the callback method will be called. Thus, the current thread of execution will not block. This model works; and it's very performant, as can be seen [in various places on the internet](http://blog.caustik.com/2012/04/10/node-js-w250k-concurrent-connections/).
Problems emerge, if, based on newly arrived data, we will have to call further external sources of data. Thus, we will add callbacks in callbacks, and this will make the code hard to reason about. This situation is best described as callback hell. In Node.js, there is a single running thread, but in Java there could be many concurrent threads running simultaneously. To make the code run correctly in a multithreading context, it will be complicated even further.

### Reactive Programming

To solve the callback hell problem by keeping the advantages of asynchronous code, a new pattern emerged, Reactive programming. Many things had been said about reactive programming. In short, Reactive Programming is about asynchronous, event-driven, non-blocking applications. Since about 2011, in Java world, many reactive libraries emerged. For interoperability, and backpressure, a standard emerged, by collaboration of several actors across the industry: [Reactive Streams](http://www.reactive-streams.org/). This standard introduces 4 interfaces and a TCK. The set of 4 interfaces are: [Publisher](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.1/api/src/main/java/org/reactivestreams/Publisher.java), [Subscriber](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.1/api/src/main/java/org/reactivestreams/Subscriber.java), [Subscription](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.1/api/src/main/java/org/reactivestreams/Subscription.java), [Processor](https://github.com/reactive-streams/reactive-streams-jvm/blob/v1.0.1/api/src/main/java/org/reactivestreams/Processor.java). The Publisher produces data, and the Subscriber consumes it. The Publisher exposes a method, allowing a consumer to subscribe. The Subscriber exposes 4 callbacks. Once the subscriber subscribed, it will receive a Subscription. Via this Subscription, the consumer can request more data, via the request method (with number of elements to return as parameter), or can unsubscribe (via the cancel method). When the Publisher has more data, will call the Subscriber's onNext callback. When there is no more data to be produced, the 'onComplete' method will be called. If an error occurred, the onError callback will be called.
[Here](https://github.com/reactive-streams/reactive-streams-jvm) are more information about this specification. Also, for a better understanding, [this](https://spring.io/blog/2016/06/07/notes-on-reactive-programming-part-i-the-reactive-landscape) article is a good one.

A notable thing to say, is that Reactive Streams specification is now part of Java 9, in the package java.util.concurrent.Flow; more can be found [here](http://openjdk.java.net/jeps/266). As reactive libraries, two are notable: [RxJava](http://reactivex.io/), and [Project Reactor](https://projectreactor.io/) . Since version 5, Spring framework also adds Reactive support. [Here](https://github.com/BogdanStirbat/message-app-different-frameworks/blob/master/reactive/src/test/java/com/bstirbat/message/reactive/ReactiveStreamsTests.java), I've played a bit with Project Reactor.

### M:N threading

By using Reactive design pattern, one can write highly-scalable applications. But, moving away from the thread per request model, one has something to loose: the familiar, linear, flow of executions. In the Reactive world, one has to think in terms of callbacks, and reasoning about programs is harder. To overcome this issue, the authors of [Quasar library](http://docs.paralleluniverse.co/quasar/) had an different approach: lightweight threads. In the M:N threading model, there are user-level threads, running on top of kernel-level threads. A user-level thread consumes less memory than a system-level one, and it's easier to schedule. Thus, we can write performant applications, while keeping the code simple.

Of course, for this model to work, all framework used by an application must be ported. Meet [Comsat](http://docs.paralleluniverse.co/comsat/), a coolection of open source libraries integrating Quasar with usefull web and database technologies. If an framework method requests some data and blocks the kernel thread, then there is not much we can do, the kernel thread and the fibers on top of it will be blocked. Fortunately, many new technologies now have an async version. And the authors of Quasar library added a mechanism for easily converting and async method into a fiber-blocking method: by extending the FiberAsync class. [Here](https://github.com/BogdanStirbat/message-app-different-frameworks/blob/master/quasar/src/main/java/com/bstirbat/message/dw/repository/MongoDbAsyncMessageRepositoryImpl.java) I've added an example of such conversion, for the MongoDB async driver.

This model exists not only in Java, but was actually borrowed from other languages. Briefly I will mention the actors model in Erlang and Scala, Go goroutines, and Kotlin coroutines. I can't wait to play with them!

### Tests
In this [Github repository](https://github.com/BogdanStirbat/message-app-different-frameworks), I made a comparison between the 3 models: thread per request, reactive, and lightweight threads. I compared the programming style, and the performance of an application written in a particular model. To do this, I implemented a simple application that handles messages. The messages are stored in a MongoDB application, because there exists a truly reactive and async driver for accessing this database. I saw that the linear execution model, in [servlet](https://github.com/BogdanStirbat/message-app-different-frameworks/tree/master/servlet) application, was maintained in the [quasar](https://github.com/BogdanStirbat/message-app-different-frameworks/tree/master/quasar) application, as I switched to lightweight threads. Of course, for the [reactive](https://github.com/BogdanStirbat/message-app-different-frameworks/tree/master/reactive) application, I had to think in terms of callbacks; but, as Reactive library offered a fluent API, it was easy.

For performance testing, I used [wrk](https://github.com/wg/wrk). This tool allows to send many concurrent requests from the same machine, by using a combination of multithreading and epoll event listening. For each application, I run 5 tests, and I've averaged the number of requests/second the application handled, and number of requests that were timeout. The application using thread per request model, [servlet](https://github.com/BogdanStirbat/message-app-different-frameworks/tree/master/servlet), created about 200 new threads during tests. About 2800 req/sec were processed, and 528 were timeout. The application using Reactive pattern, [reactive](https://github.com/BogdanStirbat/message-app-different-frameworks/tree/master/reactive), created about 20 new threads under load. It also started with a low number of threads. It processed 4182 req/sec, and 345 requests were timeout. So, the performance was better. The big surprise was the application written with lightweight threads, [quasar](https://github.com/BogdanStirbat/message-app-different-frameworks/tree/master/quasar). It processed 8505 requests per second, and no response was timeout. Also, a small number of new threads was created, about 15 (the application started with a big number of threads, about 200). Even if the performance was surprising, we have to remember that Quasar is still in an experimental phase.

### Conclusions
1. The thread per request model is good, but it might not answer all needs.
2. To address scalability issues, several solutions were implemented. In Java world, currently, the most mature and adopted is the Reactive pattern.
3. The transiction should be made only if it makes sense. For a CPU bound application, it would make no sense to use Reactive pattern or lightweight threads. Also, by making the transiction, we gain something, but we loose something else. Engineering is about trade-offs.

### Bibliography
https://bytearcher.com/articles/io-vs-cpu-bound/

https://blog.whatsapp.com/196/1-million-is-so-2011

http://highscalability.com/blog/2013/5/13/the-secret-to-10-million-concurrent-connections-the-kernel-i.html

https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/

https://nodejs.org/en/

http://blog.caustik.com/2012/04/10/node-js-w250k-concurrent-connections/

https://github.com/reactive-streams/reactive-streams-jvm

https://spring.io/blog/2016/06/07/notes-on-reactive-programming-part-i-the-reactive-landscape

http://reactivex.io/

https://projectreactor.io/

http://openjdk.java.net/jeps/266

http://docs.paralleluniverse.co/quasar/

http://docs.paralleluniverse.co/comsat/