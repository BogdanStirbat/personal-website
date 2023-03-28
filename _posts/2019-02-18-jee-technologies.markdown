---
layout: post
title:  "JEE technologies"
date:   2019-02-18 21:49:18 +0300
categories: jekyll update
---

### Introduction

JEE is a collection of standards, developed by various actors from industry, in a process called [Java Comunity Process](https://jcp.org/en/home/index) or JCP. 
The standards are implemented by different application servers or libraries. Standards are, just like interfaces, a contract; a solution needs to pass a TCK (a suite of tests) to be certified as implementing a standard.
This standardization process allows application developers to know what services are provided by application servers and libraries they use.
At the time of writing this article, the latest JEE version was JEE 8. Full list of standards can be found [here](https://www.oracle.com/technetwork/java/javaee/tech/index.html).

This article will further enumerate some application servers often used in industry. This discussion will lead to the two options Java developers have, when writing a new JEE application: JEE and Spring. 
Next, a bit of platform history will be presented. Then, a non-complete list of standards will be listed and described briefly.


### Servlets, Servlet containers, Application servers

A concept often encountered when discussing about JEE is the concept of container. In short, a container is an application on top of with end user JEE applications are running. The container offers services and manages resources for a JEE application.
A more detailed definition can be found [here](https://docs.oracle.com/javaee/5/tutorial/doc/bnabo.html).

The first standard we will discuss about is the Servlet. A servlet is, in simple terms, a Java class extending the [HttpServlet](https://javaee.github.io/javaee-spec/javadocs/javax/servlet/http/HttpServlet.html) class.
It has methods like `doGet` and `doPost`; when a web request arrives from a client, the methods are executed and the method's result are sent back to the client. 
Of course, extending this class is not enough if we want to serve web clients. We also need to do some low level networking stuff: opening an internet connection, waiting for web request, calling one of our servlet class's method when a 
request arrives (`doGet`, `doPost` etc, depending on context), and sending the result back to the client. This low level job is done by a program called a servlet container. 
Thus, we also need a servlet container. The servlet class will be handled to the container: the container will then load the class, and start serving requests. 
Of course, we will not handle just a class, but a full application: classes, static resources, libraries used etc are packaged in special java archives called wars. 
We handle the war file to the servlet container, and we say that we deployed the application to some container. 
The servlet specification defines the structure of a war archive, so that servlet containers will know where to look when an application is deployed. The specification evolved, and what was done once with xml files now can be done with Java annotations.

Armed with this knowledge, we can now enumerate the two big classes of application servers: application servers and web servers.

Web servers, also named web containers, servlet containers etc implement only a small part of JEE standard: the servlet standard, and usually two other servlet related standards.
Example of web servers: [Apache Tomcat](http://tomcat.apache.org/) and [Jetty](https://www.eclipse.org/jetty/).

Application servers, on the other hand, implement the full JEE standard. Examples of application servers: 
[GlassFish](https://javaee.github.io/glassfish/) (the reference JEE implementation), [Apache TomEE](https://tomee.apache.org/), [WildFly](http://www.wildfly.org/) and so on. Some are free, for others you have to pay.

As we can see, there are two options for developing JEE applications: we can either use a servlet container (in combination with Spring framework), either use an application server.

### Development options: Servlet containers (with Spring) and application containers

One of the option to be used when developing a new JEE application is to use an application server. All the JEE standards are available, so in theory there is no need for any other additional libraries.

The other is to use a Servlet container with Spring. All the additional libraries should be added, usually via Maven/Gradle, configured and glued together. 
Configuring and gluing libraries together with Spring can be hard, specially if you have not done it before and/or you don't know the Servlet specification.
To address the problem, [Spring Boot](https://spring.io/projects/spring-boot) was created. Spring Boot is an opinionated way of configuring a Spring application; it even has an initializer, [Spring Initializr](https://start.spring.io/), where you can just specify what you need and a starter project is created.

Now, with solution is better? Both have advantages and disadvantages. 
Using a JEE application server gives a final, standard solution, without requiring gluing together different libraries. 
Also, you can buy an application server that offers support, so you know whom to call if something is not working.
Using Spring, on the other hand, gives freedom to use non-JEE standard solutions if the situation requires it. Also, Spring tends to be more innovative, and it's where innovation happens.

[Here](https://zeroturnaround.com/rebellabs/spring-vs-java-ee-survey-results/) you can have some statistics regarding the popularity of JEE vs Spring popularity.
As said [here](https://www.reddit.com/r/java/comments/7kxsbb/is_javaee_or_spring_stack_better_for_the_job/), if you know one platform the other one can be easily learned.
More information about Spring vs Java EE can be found [here](https://www.quora.com/What-are-the-differences-between-Java-EE-and-Spring). An other good article is [this](https://www.baeldung.com/java-enterprise-evolution).

To see how we came to this situation of having two option, we need to know a little history. 


### Spring and JEE history

The first version of JEE (it was called J2EE back then) appeared in [1999](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition). The newly born platform offered a great number of features for developers to build on.
But the platform was very hard to use. Application servers took a lot of time to start, a lot of configuration was required to do even the simplest things. [Here](https://www.quora.com/What-are-the-differences-between-Java-EE-and-Spring) you can find more information about the early difficulties associated with the platform.

A lightweight solution was required. In this context, [Rod Johnson](https://en.wikipedia.org/wiki/Rod_Johnson_(programmer)) wrote a book, in 2002, [Expert One-on-One J2EE Design and Development](https://www.amazon.com/gp/product/0764543857).
The book (still valuable today) addressed the problems, and proposed an elegant solution. The book was also accompanied by 30,000 lines of framework code; in [collaboration](https://spring.io/blog/2006/11/09/spring-framework-the-origins-of-a-project-and-a-name) with Juergen Hoeller and Yann Caroff, this framework became the Spring framework.
The framework's name was choosen to be Spring, because it was a spring from the J2EE winter.

In time JEE evolved as well, and the original difficulties developing JEE applications were eliminated. Of course, Spring evolved as well during this time. It's a pattern of innovation, one platform was influencing the other.
But the two platform are in a sort of competition, you can find out more [here](https://www.coherentsolutions.com/blog/the-on-going-war-between-jee-and-spring-framework/).

Spring tends to be the more innovative solution; and JEE tends to standardise the maturing and established solutions, integrating them in the platform.


### JEE standards

Next, I will briefly present a non-complete list JEE standards. Full list can be found [here](https://www.oracle.com/technetwork/java/javaee/tech/index.html).
Related standards are grouped.


#### Servlet, JSP, JSTL, EL, JSF, WebSocket

- [Java Servlet 4.0, JSR 369](https://jcp.org/en/jsr/detail?id=369)
- [JavaServer Pages 2.3, JSR 245](https://jcp.org/en/jsr/detail?id=245)
- [Standard Tag Library for JavaServer Pages (JSTL) 1.2, JSR 52](https://jcp.org/en/jsr/detail?id=52)
- [Expression Language 3.0, JSR 341](https://jcp.org/en/jsr/detail?id=341)
- [JavaServer Faces 2.3, JSR 372](https://jcp.org/en/jsr/detail?id=372)
- [Java API for WebSocket 1.1, JSR 356](https://jcp.org/en/jsr/detail?id=356)

We discussed previously about servlets: they are basically Java classes. Writing a Java class having methods that return HTML documents as String is difficult and error prone, and for this reason JSP was created.
JSP is, basically, a HTML page allowing inclusion of Java code. A JSP file will be compiled to a servlet, by the web container/application server, at deploy time or when the page is first accessed. More information about JSP can be found [here](https://en.wikipedia.org/wiki/JavaServer_Pages).
JSP allows the insertion of Java code via <% %> blocks. To make JSP pages more readable, JSTL defines tags for common tasks done in Java code.
JSP pages are further enhanced with EL, allowing the insertion of Java expressions using syntax like ${}.

To simplify the process of developing web applications, JSF was created. JSF is a component based web framework; more information can be found [here](https://en.wikipedia.org/wiki/JavaServer_Faces).

The JEE platform also has support for the WebSocket standard; more information can be found [here](https://www.oracle.com/technetwork/articles/java/jsr356-1937161.html).


#### Dependency injection, beans

- [Dependency Injection for Java 1.0, JSR 330](https://jcp.org/en/jsr/detail?id=330)
- [Contexts and Dependency Injection for Java 2.0, JSR 365](https://jcp.org/en/jsr/detail?id=365)
- [Enterprise JavaBeans 3.2, JSR 345](https://jcp.org/en/jsr/detail?id=345)
- [Bean Validation 2.0, JSR 380](https://jcp.org/en/jsr/detail?id=380)
- [Interceptors 1.2, JSR 318](https://jcp.org/en/jsr/detail?id=318)
- [Common Annotations for the Java Platform 1.3, JSR 250](https://jcp.org/en/jsr/detail?id=250)

Spring is an Dependency Injection framework. The idea worked well, and was adopted in JEE as well. JSR 330 defines injection annotations, like `@Inject`, similar to the ones defined by Spring. Spring also supports JSR 330 annotation.

JSR 345 defines Enterprise JavaBeans. An Enterprise Java bean is a special class that encapsulates business logic. And, by using annotation provided by the standard, it's lifecycle is managed by the container.
It's all about the Dependency Injection pattern; to make things clear, some examples can be found [here](https://docs.oracle.com/cd/E13222_01/wls/docs100/ejb30/examples.html).

An standard that emerged from EJB is JSR 318, specifying interceptors. An interceptor is a sort of hook, allowing introduction of code to be executed on method invocations or lifecycle events; more information can be foun [here](https://javaee.github.io/tutorial/interceptors001.html) and [here](https://abhirockzz.wordpress.com/2015/01/03/java-ee-interceptors/).

The standardisation of Dependency Injection and JavaBeans created a lot of annotations. The common annotations were defined by the JSR 250 (Common Annotations for the Java Platform 1.3) standard.


#### Database access, transactions

- [Java Database Connectivity 4.0, JSR 221](https://jcp.org/en/jsr/detail?id=221)
- [Java Persistence 2.2, JSR 338](https://jcp.org/en/jsr/detail?id=338)
- [Java Transaction API (JTA) 1.2, JSR 907](https://jcp.org/en/jsr/detail?id=907)

Transactions are a powerful feature, implemented at the [database level](https://brandur.org/postgres-atomicity), ensuring that a group of operations either completes successfully and the data is persisted, 
either if a operation fails no modifications is saved in the database; you can read more about the topic [here](https://vladmihalcea.com/a-beginners-guide-to-acid-and-database-transactions/).
In enterprise software, transactions are very important; in case of a bug, or a network connection error for example, data is not left in a messy state.

The first API for database access was JDBC; it was release in [1997](https://jcp.org/en/jsr/detail?id=221), and it's part of Java Standard Edition.
The API is [hard to use](https://www.tutorialspoint.com/jdbc/jdbc-sample-code.htm), an frameworks appeared to work around it. One of them is [Spring JDBC](https://www.baeldung.com/spring-jdbc-jdbctemplate). 
An other successfully framework is [Hibernate ORM](http://hibernate.org/orm/), an [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) for the Java language.
This framework was so widely used, that it influenced the design of JPA. Hibernate is a JPA provider.

JTA offers an API for controlling transactions in Java EE applications. Spring Framework also offers strong [transaction support](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/data-access.html#transaction).

#### Web services

- [Java API for RESTful Web Services (JAX-RS) 2.1, JSR 370](https://jcp.org/en/jsr/detail?id=370)
- [Java API for XML-Based Web Services (JAX-WS) 2.2, JSR 224](https://jcp.org/en/jsr/detail?id=224)
- [Implementing Enterprise Web Services 1.3, JSR 109](https://jcp.org/en/jsr/detail?id=109)
- [Web Services Metadata for the Java Platform 2.1, JSR 181](https://jcp.org/en/jsr/detail?id=181)
- [SOAP with Attachments API for Java (SAAJ) Specification 1.3, JSR 67](https://jcp.org/en/jsr/detail?id=67)
- [Java API for XML-Based Web Services (JAX-WS) 2.2, JSR 224](https://jcp.org/en/jsr/detail?id=224)

This set of standards refer to [web services](https://en.wikipedia.org/wiki/Web_service). Simply put, a web service is a service offered by an application to an other application. 
The data exchange is done, usually, via HTTP; data is exchanged in computer readable file formats, most often XML and JSON.

There are two API's for web services: [JAX-RS](https://en.wikipedia.org/wiki/Java_API_for_RESTful_Web_Services) and [JAX-WS](https://en.wikipedia.org/wiki/Java_API_for_XML_Web_Services).


#### JSON processing

- [Java API for JSON Binding 1.0, JSR 367](https://jcp.org/en/jsr/detail?id=367)
- [Java API for JSON Processing 1.1, JSR 374](https://jcp.org/en/jsr/detail?id=374)

This set of standards refer to manipulating JSON objects.
 

#### Security

- [Java EE Security API 1.0, JSR 375](https://jcp.org/en/jsr/detail?id=375)
- [Java Authentication Service Provider Interface for Containers 1.1, JSR 196](https://jcp.org/en/jsr/detail?id=196)
- [Java Authorization Contract for Containers 1.5, JSR 115](https://jcp.org/en/jsr/detail?id=115)

This is a group of standards addressing the need of securing a JEE application.


#### Batch application

- [Batch Applications for the Java Platform 1.0, JSR 352](https://jcp.org/en/jsr/detail?id=352)

This standard allows running of [batch](https://www.oracle.com/technetwork/articles/java/batch-1965499.html) jobs in an enterprise application. 
Batch jobs are tasks non-interactive, long running, possible computational intensive tasks like interest calculation, computing a rapport at the end of the day etc.

Spring offers Batch capabilities too; [here](https://blog.codecentric.de/en/2013/07/spring-batch-and-jsr-352-batch-applications-for-the-java-platform-differences/) is a comparison between the two solutions.


#### Concurrency Utilities

- [Concurrency Utilities for Java EE 1.0, JSR 236](https://jcp.org/en/jsr/detail?id=236)

This standard allows the creation of managed threads. Programmers no longer have to manually create, start new threads; the platform will manage the threads for them. 
Managed threads are safer to use, and JEE services are guaranteed to work inside a managed thread. [Here](https://docs.oracle.com/javaee/7/tutorial/concurrency-utilities.htm) can be found more information.


#### JCA

- [Java EE Connector Architecture 1.7, JSR 322](https://jcp.org/en/jsr/detail?id=322)

This standard was defined for the task of connecting a JEE platform to different Enterprise Information Systems.


#### JMS

- [Java Message Service API 2.0, JSR 343](https://jcp.org/en/jsr/detail?id=343)

This standard defines the capability of two or more JEE application to exchange messages. It allows [loosely coupled, distributed](https://en.wikipedia.org/wiki/Java_Message_Service) communication between different applications.  


#### JavaMail

- [JavaMail 1.6, JSR 919](https://jcp.org/en/jsr/detail?id=919)

This standard allows JEE applications to send emails. More information can be found [here](https://javaee.github.io/javamail/).


#### JMX

- [Java Management Extensions (JMX) 2.0, JSR 3](https://jcp.org/en/jsr/detail?id=3)

[JMX](https://www.oracle.com/technetwork/java/javase/tech/javamanagement-140525.html) provide the tools to monitor Java applications; JMX it's part of Java Standard Edition, so it's not JEE specific only.
More information can be found [here](https://stackify.com/java-management-extensions/).

Examples of such tools are [JConsole](http://openjdk.java.net/tools/svc/jconsole/), part of OpenJDK, and [Java Mission Control](https://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html), part of Oracle JDK.
If you have Java installed, most likely you also have one of these two tools installed. You can just start it, and experiment with it to see what features it provides.


### Conclusion

JEE is a mature platform, offering powerful tolls for enterprise developers. Developers can choose to use either Spring Framework, either an JEE application server. The platform changed a lot over the last 20 years, and it's still evolving today.