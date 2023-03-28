---
layout: post
title:  "Reactive Programming in Spring Framework"
date:   2017-04-02 12:25:18 +0300
categories: jekyll update
---

### Introduction

Reactive programming is, according to [Wikipedia](https://en.wikipedia.org/wiki/Reactive_programming), an asynchronous programming paradigm oriented around data streams and the propagation of change. It involves two components, a Producer and a Consumer. The Consumer subscribes to the Producer, and the Producer sends a stream of events to Consumer. It resembles the Observable design pattern. When a client makes a request to a server, usually client calls the server, and waits the server to send data. In Reactive paradigm, the client subscribes to the server, then continues it's work; when data arrives, it's notified by the server.


### Motivation

In recent years, multi core CPU systems become the norm, and microservice architectures becomes more and more common. In this context, programmers have to deal with multithreading programming challenges more often. Multithreading programming is also notorious hard, and Reactive Programming makes it simpler.

In this context, Reactive Programming, with is not something new, caught attention. Many libraries, frameworks, were developed to support this paradigm. A notable initiative is [Reactive Streams](http://www.reactive-streams.org/). Quoting the project's authors,
> The scope of Reactive Streams is to find a minimal set of interfaces, methods and protocols that will describe the necessary operations and entities to achieve the goal—asynchronous streams of data with non-blocking back pressure.

We see a new term, back pressure. When a server sends data to a client, with a rare high enough that the client is overwhelmed and can't handle this load (this pressure), we want the server to slow or even stop emitting new events. This is what back pressure is.
So, Reactive Streams is a specification (a [interface](https://github.com/reactive-streams/reactive-streams-jvm)). Thus, if a client and server communicate in a Reactive way, and we want to have back pressure, the minimum requirement is that both client and server use a reactive library compatible with Reactive Streams specification - but not the same!

An other motivation comes from the need of vertical scaling. Suppose we have a web server, hosted on a modern computer, having an internet connection of 10 Gb/s, 64 GB RAM, many core CPU. The question is, how many concurrent clients can this server handle? According to [some sources](http://stackoverflow.com/questions/1575453/how-many-socket-connections-can-a-web-server-handle), the server should be able to handle 1 to 10 million clients; and this number should not be a surprise, given [WhatsApp announcement](https://blog.whatsapp.com/196/1-million-is-so-2011?), in 2011, that they supported 1 million concurrent TCP connections. In Java EE, Servlet specification added an one thread per request model. This model simplifies web server programming, as they are written as one client is accessing the server. Unfortunately, an operating system cannot run more than 10000-20000 threads, not to mention context switching overhead (think about the cost of a context switch, when 10000 threads are running) and memory overhead.
Thus, number of concurrent client reduced, from the range 100000-1000000, to no more than 10000-20000. That's a huge drop.

In 2009, in the world of multi-core CPUs, the creation of Node.js framework, single threaded, was a surprise. Whenever a blocking operation was necessary, like a database call, application code registered a callback; when data was available, application code was notified without the need of an blocking operation. Node.js framework scaled surprisingly well.

This framework influenced a movement towards async, non-blocking frameworks, in many programming languages including Java. And Reactive programming gained a lot of popularity.


### In real world

In Java world, and in many other JVM and not-JVM specific language, a lot of Reactive libraries were developed. Reactive Streams specification made such a great impact, that libraries implementing and not implementing this specification were in a different classes. David Karnok, a contributor of Reactive Streams and other project, made a classification for these libraries, a classification that becomed an de-facto standard. The classification can be found in this [blog post](https://akarnokd.blogspot.ro/2016/03/operator-fusion-part-1.html), referred in many places across the internet. For understanding the current Reactive landscape, I strongly recommend reading this [blog post](https://akarnokd.blogspot.ro/2016/03/operator-fusion-part-1.html).

[Reactive Streams](http://www.reactive-streams.org/) made such a big impact, that Reactive Streams interfaces were incorporated even in [Java 9](http://download.java.net/java/jdk9/docs/api/java/util/concurrent/Flow.html)! This is a strong sign, that Reactive is here to stay. Spring authors also reacted to this change; from Spring 5, [Project Reactor](https://projectreactor.io/), a library compatible with Reactive Streams specification, was incorporated into [Spring core framework](https://spring.io/blog/2016/07/28/reactive-programming-with-spring-5-0-m1). In an [DEVOXX presentation](https://www.youtube.com/watch?v=Cj4foJzPF80), Brian Clozel states that adding a dependency to Spring Framework happens not very often.

Spring team made an excellent job in documenting the new Reactive features. Spring's new WebFlux framework is documented not only in [official documentation](http://docs.spring.io/spring/docs/5.0.0.M5/spring-framework-reference/htmlsingle/#web-reactive). Also, on Spring's blog, [here (Part 1)](https://spring.io/blog/2016/06/07/notes-on-reactive-programming-part-i-the-reactive-landscape), [here (Part 2)](https://spring.io/blog/2016/06/13/notes-on-reactive-programming-part-ii-writing-some-code) and [here (Part 3)](https://spring.io/blog/2016/07/20/notes-on-reactive-programming-part-iii-a-simple-http-server-application) Reactive Programming, and Spring adaption of Reactive Programming, are clear, buzzword-free, with advantages, disadvantages, and use cases, explained. I strongly recommend reading this series. Apart from crystal-clear explanations, many general misconceptions, confusions, regarding Reactive Programming are clarified. They even made an [API Hands On workshop](https://github.com/reactor/lite-rx-api-hands-on).

I expect the visitors to read the specified articles. It would be impractical to repeat the same information here.


### Experiments

Here, I will present some experiments with this framework. In this [GitHub repository](https://github.com/BogdanStirbat/reactive-demo) there are 4 projects:
* demo-server
* reactiveweb-app
* servlet-app
* benchmark-runner

In this setup, we have a database, consisting of 100 messages (each with it's own id, generated from 1 to 100). demo-server exposes an API that accepts an message id and returns the associated message.
For this server, there are 2 other servers, each consuming the demo-server's API. One server is running in an "Reactive container" (reactiveweb-app), the other one is running in an traditional servlet container (servlet-app). Each of these 2 servers accept an message id, call demo-server to retrieve the corresponding message, and returns the message back to client.

The purpose of this experiment was to test 2 different web applications, one Reactive and one non-Reactive. So, an client is also included - benchmark-runner. This client runs in the command line. Opens a number of requests, to one of the 2 servers. Both parameters - number of requests, with server - are given as command line arguments.

In the main thread, benchmark-runner creates an ExecutorService, and for each request, supplies an async task using Java 8's CompletableFuture.
Code:

{% highlight java linenos %}
    ExecutorService es = Executors.newFixedThreadPool(numberOfRequests);
    List<CompletableFuture> completableFutures = new ArrayList<>();

    for(int i = 1; i <= numberOfRequests; i++) {
        final int messageId = i;
        CompletableFuture<Message> completableFuture = CompletableFuture.supplyAsync(() -> loadMessage(strings[0], selectedUri, messageId));
        completableFutures.add(completableFuture);
    }

    CompletableFuture<Void> combined = CompletableFuture.allOf(completableFutures.toArray(new CompletableFuture[completableFutures.size()]));
    combined.get();
{% endhighlight %}

I wanted to observe how each of the 4 components behave under test. For this, logs were added, in all 4 projects. Each controller's method execution (demo-server, reactiveweb-app, servlet-app) is surrounded in following code block:

{% highlight java linenos %}
LOGGER.info("Received request for message id: {}", id);
...
LOGGER.info("Finished request for message id: {}", id);
{% endhighlight %}

To simulate a long processing by demo-server, 15 seconds are waited before returning the result from DB.

First, I have tested servlet-app component. Then, the reactiveweb-app component.

Analyzing logs, I've observed that reactiveweb-app's controller returns immediately. The client thread, from benchmark-runner makes a call towards reactiveweb-app's controller, and waits the response for 15 seconds. reactiveweb-app's controller finishes immediately the call, but the client still waits. Of course, demo-server processes the request in 15 seconds.
servlet-app's controller, instead, does not returns immediately. It waits, like the benchmark-runner client, 15 seconds.

I've also observed that no more than 3 benchmark-runner threads were running, at every moment. 3 threads were making the call, then waiting 15 seconds, and after each thread completion another thread made the request.

I've added the console output here.

{% highlight bash %}
spring-reactive/benchmark-runner$ java -jar target/benchmark-runner-0.0.1-SNAPSHOT.jar servlet 10
{% endhighlight %}

Console outputs:
benchmark-runner
{% highlight bash %}
Prepare to send 10 requests to servlet
Supply async: 1
Supply async: 2
Supply async: 3
Supply async: 4
Supply async: 5
Supply async: 6
Supply async: 7
Supply async: 8
Supply async: 9
Supply async: 10
2017-04-03 20:53:56.135  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 3
2017-04-03 20:53:56.134  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 1
2017-04-03 20:53:56.135  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 2
2017-04-03 20:54:12.736  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 3
2017-04-03 20:54:12.736  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 4
2017-04-03 20:54:12.737  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 1
2017-04-03 20:54:12.737  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 5
2017-04-03 20:54:12.736  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 2
2017-04-03 20:54:12.743  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 6
2017-04-03 20:54:27.780  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 4
2017-04-03 20:54:27.780  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 7
2017-04-03 20:54:27.797  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 6
2017-04-03 20:54:27.797  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 8
2017-04-03 20:54:27.798  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 5
2017-04-03 20:54:27.798  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 9
2017-04-03 20:54:42.824  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 7
2017-04-03 20:54:42.824  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service servlet for id 10
2017-04-03 20:54:42.840  INFO 16442 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 8
2017-04-03 20:54:42.848  INFO 16442 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 9
2017-04-03 20:54:57.852  INFO 16442 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service servlet for id 10
{% endhighlight %}

servlet-app
{% highlight bash %}
2017-04-03 20:53:56.757  INFO 15669 --- [nio-8082-exec-1] c.b.r.d.s.controller.MessageController   : Received request for message id: 2
2017-04-03 20:53:56.760  INFO 15669 --- [nio-8082-exec-2] c.b.r.d.s.controller.MessageController   : Received request for message id: 1
2017-04-03 20:53:56.761  INFO 15669 --- [nio-8082-exec-3] c.b.r.d.s.controller.MessageController   : Received request for message id: 3
2017-04-03 20:54:12.653  INFO 15669 --- [nio-8082-exec-1] c.b.r.d.s.controller.MessageController   : Finished request for message id: 2
2017-04-03 20:54:12.653  INFO 15669 --- [nio-8082-exec-2] c.b.r.d.s.controller.MessageController   : Finished request for message id: 1
2017-04-03 20:54:12.658  INFO 15669 --- [nio-8082-exec-3] c.b.r.d.s.controller.MessageController   : Finished request for message id: 3
2017-04-03 20:54:12.739  INFO 15669 --- [nio-8082-exec-4] c.b.r.d.s.controller.MessageController   : Received request for message id: 4
2017-04-03 20:54:12.743  INFO 15669 --- [nio-8082-exec-5] c.b.r.d.s.controller.MessageController   : Received request for message id: 5
2017-04-03 20:54:12.751  INFO 15669 --- [nio-8082-exec-6] c.b.r.d.s.controller.MessageController   : Received request for message id: 6
2017-04-03 20:54:27.778  INFO 15669 --- [nio-8082-exec-4] c.b.r.d.s.controller.MessageController   : Finished request for message id: 4
2017-04-03 20:54:27.784  INFO 15669 --- [nio-8082-exec-5] c.b.r.d.s.controller.MessageController   : Finished request for message id: 5
2017-04-03 20:54:27.790  INFO 15669 --- [nio-8082-exec-6] c.b.r.d.s.controller.MessageController   : Finished request for message id: 6
2017-04-03 20:54:27.795  INFO 15669 --- [nio-8082-exec-7] c.b.r.d.s.controller.MessageController   : Received request for message id: 7
2017-04-03 20:54:27.802  INFO 15669 --- [nio-8082-exec-8] c.b.r.d.s.controller.MessageController   : Received request for message id: 8
2017-04-03 20:54:27.803  INFO 15669 --- [nio-8082-exec-9] c.b.r.d.s.controller.MessageController   : Received request for message id: 9
2017-04-03 20:54:42.819  INFO 15669 --- [nio-8082-exec-7] c.b.r.d.s.controller.MessageController   : Finished request for message id: 7
2017-04-03 20:54:42.831  INFO 15669 --- [nio-8082-exec-8] c.b.r.d.s.controller.MessageController   : Finished request for message id: 8
2017-04-03 20:54:42.835  INFO 15669 --- [io-8082-exec-10] c.b.r.d.s.controller.MessageController   : Received request for message id: 10
2017-04-03 20:54:42.846  INFO 15669 --- [nio-8082-exec-9] c.b.r.d.s.controller.MessageController   : Finished request for message id: 9
2017-04-03 20:54:57.850  INFO 15669 --- [io-8082-exec-10] c.b.r.d.s.controller.MessageController   : Finished request for message id: 10
{% endhighlight %}

demo-server
{% highlight bash %}
2017-04-03 20:53:57.128  INFO 15801 --- [nio-8080-exec-3] c.b.r.d.s.controller.MessageController   : Received request for message id: 3
2017-04-03 20:53:57.128  INFO 15801 --- [nio-8080-exec-1] c.b.r.d.s.controller.MessageController   : Received request for message id: 1
2017-04-03 20:53:57.128  INFO 15801 --- [nio-8080-exec-2] c.b.r.d.s.controller.MessageController   : Received request for message id: 2
2017-04-03 20:54:12.128  INFO 15801 --- [nio-8080-exec-3] c.b.r.d.s.controller.MessageController   : Finished request for message id: 3
2017-04-03 20:54:12.129  INFO 15801 --- [nio-8080-exec-1] c.b.r.d.s.controller.MessageController   : Finished request for message id: 1
2017-04-03 20:54:12.131  INFO 15801 --- [nio-8080-exec-2] c.b.r.d.s.controller.MessageController   : Finished request for message id: 2
2017-04-03 20:54:12.750  INFO 15801 --- [nio-8080-exec-4] c.b.r.d.s.controller.MessageController   : Received request for message id: 5
2017-04-03 20:54:12.750  INFO 15801 --- [nio-8080-exec-5] c.b.r.d.s.controller.MessageController   : Received request for message id: 4
2017-04-03 20:54:12.757  INFO 15801 --- [nio-8080-exec-6] c.b.r.d.s.controller.MessageController   : Received request for message id: 6
2017-04-03 20:54:27.751  INFO 15801 --- [nio-8080-exec-4] c.b.r.d.s.controller.MessageController   : Finished request for message id: 5
2017-04-03 20:54:27.755  INFO 15801 --- [nio-8080-exec-5] c.b.r.d.s.controller.MessageController   : Finished request for message id: 4
2017-04-03 20:54:27.759  INFO 15801 --- [nio-8080-exec-6] c.b.r.d.s.controller.MessageController   : Finished request for message id: 6
2017-04-03 20:54:27.799  INFO 15801 --- [nio-8080-exec-7] c.b.r.d.s.controller.MessageController   : Received request for message id: 7
2017-04-03 20:54:27.806  INFO 15801 --- [nio-8080-exec-8] c.b.r.d.s.controller.MessageController   : Received request for message id: 8
2017-04-03 20:54:27.809  INFO 15801 --- [nio-8080-exec-9] c.b.r.d.s.controller.MessageController   : Received request for message id: 9
2017-04-03 20:54:42.799  INFO 15801 --- [nio-8080-exec-7] c.b.r.d.s.controller.MessageController   : Finished request for message id: 7
2017-04-03 20:54:42.806  INFO 15801 --- [nio-8080-exec-8] c.b.r.d.s.controller.MessageController   : Finished request for message id: 8
2017-04-03 20:54:42.810  INFO 15801 --- [nio-8080-exec-9] c.b.r.d.s.controller.MessageController   : Finished request for message id: 9
2017-04-03 20:54:42.841  INFO 15801 --- [io-8080-exec-10] c.b.r.d.s.controller.MessageController   : Received request for message id: 10
2017-04-03 20:54:57.842  INFO 15801 --- [io-8080-exec-10] c.b.r.d.s.controller.MessageController   : Finished request for message id: 10
{% endhighlight %}

Interesting things are observed when testing the reactiveweb-app.

Code:
{% highlight bash %}
spring-reactive/benchmark-runner$ java -jar target/benchmark-runner-0.0.1-SNAPSHOT.jar reactive 10
{% endhighlight %}

Console outputs:
benchmark-runner
{% highlight bash %}
Supply async: 1
Supply async: 2
Supply async: 3
Supply async: 4
Supply async: 5
Supply async: 6
Supply async: 7
Supply async: 8
Supply async: 9
Supply async: 10
2017-04-03 20:58:22.927  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 1
2017-04-03 20:58:22.930  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 3
2017-04-03 20:58:22.928  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 2
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 1
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 2
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 4
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 3
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 6
2017-04-03 20:58:39.049  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 5
2017-04-03 20:58:54.095  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 6
2017-04-03 20:58:54.095  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 7
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 4
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 5
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 8
2017-04-03 20:58:54.100  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 9
2017-04-03 20:59:09.147  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 9
2017-04-03 20:59:09.148  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Prepare to call service reactive for id 10
2017-04-03 20:59:09.156  INFO 16554 --- [onPool-worker-1] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 8
2017-04-03 20:59:09.160  INFO 16554 --- [onPool-worker-3] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 7
2017-04-03 20:59:24.187  INFO 16554 --- [onPool-worker-2] c.b.r.b.BenchmarkRunnerApplication       : Finished call to service reactive for id 10
{% endhighlight %}

reactiveweb-app console
{% highlight bash %}
2017-04-03 20:58:23.439  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Received request for message id: 1
2017-04-03 20:58:23.439  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Received request for message id: 3
2017-04-03 20:58:23.439  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Received request for message id: 2
2017-04-03 20:58:23.482  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Finished request for message id: 1
2017-04-03 20:58:23.484  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Finished request for message id: 2
2017-04-03 20:58:23.484  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Finished request for message id: 3
2017-04-03 20:58:39.051  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Received request for message id: 6
2017-04-03 20:58:39.051  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Received request for message id: 4
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Received request for message id: 5
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Finished request for message id: 5
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Finished request for message id: 6
2017-04-03 20:58:39.052  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Finished request for message id: 4
2017-04-03 20:58:54.101  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Received request for message id: 7
2017-04-03 20:58:54.101  INFO 15631 --- [-server-epoll-7] c.b.r.d.r.controller.MessageController   : Finished request for message id: 7
2017-04-03 20:58:54.102  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Received request for message id: 8
2017-04-03 20:58:54.106  INFO 15631 --- [-server-epoll-5] c.b.r.d.r.controller.MessageController   : Received request for message id: 9
2017-04-03 20:58:54.107  INFO 15631 --- [-server-epoll-5] c.b.r.d.r.controller.MessageController   : Finished request for message id: 9
2017-04-03 20:58:54.108  INFO 15631 --- [-server-epoll-8] c.b.r.d.r.controller.MessageController   : Finished request for message id: 8
2017-04-03 20:59:09.164  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Received request for message id: 10
2017-04-03 20:59:09.165  INFO 15631 --- [-server-epoll-6] c.b.r.d.r.controller.MessageController   : Finished request for message id: 10
{% endhighlight %}

We can see, the request arrives, and immediately returns (observe times for 'Received request' and 'Finished request'; compare the times with previous example). The message is retrieved via a Mono, using a WebClient. Thus, the server does not has to wait. Behind the scenes, framework will subscribe to this Mono. So, the server handles requests fast, is always prepared to handle more requests. But the client is blocked. Logs for the demo-server are similar as before, so adding them here would be pointless.

I was surprised, that only 3 threads were running every time. I wanted to run multiple request, from the same machine, at the same time.

Remember the initial scalability problem? While Reactive's solution was to limit the number of threads, other libraries had a different approach. [Quasar](http://www.paralleluniverse.co/quasar/) adds lightweight threads to the JVM. A lightweight thread is a thread scheduled by the application, and not by the operating system. Thus, we can still maintain the simple to program one thread per request programming model, without hurting scalability. They even invented a sort of servlet container with lightweight threads: [Comsat](http://www.paralleluniverse.co/comsat/). A small HTTP load testing app was implemented in this model: [Photon](https://github.com/puniverse/photon).

I've run 100 concurrent connections to reactiveweb-app, and I've monitored number of threads created by demo-server and reactiveweb-app:

{% highlight bash %}
java -jar photon.jar -rate 20 -duration 5 -timeout 12000 -stats http://localhost:8081/message/28
{% endhighlight %}

Not surprisingly, I saw one thread procesing requests in reactiveweb-app, and about 100 in demo-server.


This observations leads us straight to conclusions.


### Conclusions
1. To address vertical scalability issues, one thread per request model had to be left behind.
2. Reactive programming simplifies concurrent programming. But, to fully benefit from this mode, reactive components must be used all way down, including at database level. And, currently, a Reactive driver exists only for MongoDB.
3. Reactive programs are harder to debug.
4. Reactive is not the only solution to the vertical scalability problem. Other models promise similar results, keeping the simplicity of one thread per request model.