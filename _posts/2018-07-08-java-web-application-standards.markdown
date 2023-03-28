---
layout: post
title:  "Java web applications using Tomcat, Spring, Hibernate and JEE standards"
date:   2018-07-08 12:25:18 +0300
categories: jekyll update
---

### Introduction

In this blog post, I want to discuss about typical technologies behind Java EE web applications. I will focus on a typical 'standard' stack: Tomcat, Spring, Hibernate, and how all there technologies interplay in the making of a wonderful new Java web application.

### Frameworks, standards, and Java EE.
[Java EE](http://www.oracle.com/technetwork/java/javaee/overview/index.html) is a collection of standards. These standards come to life in a process named [JCP](https://www.jcp.org/en/home/index). In short, JCP is a process that allows individuals, organization to collaborate and define new standards. The process is open, everybody can bring new ideas on the table.

Each standard is, essentially, a contract. So, if a technology implements a contract, you know how to use it and what services it offers. For example, Hibernate implements [JPA](http://www.oracle.com/technetwork/java/javaee/tech/persistence-jsp-140049.html) standard; it is often said that Hibernate is a JPA provider; an other JPA provider is [EclipseLink](http://www.eclipse.org/eclipselink/). So, by having standards, the gain is huge: if you have 2 libraries implementing the same standard, you know how to communicate with the 2 libraries and what services they provide. So, in theory, you can always change the 2 frameworks, they both provide the same functionality. Of course, in practice this does not holds true: for example sometimes vendors implement functionalities out of the standard that you unconciously get to use, and you realize this when it's too difficult to change; or the standards might not cover some very granular details, leaving space for creativity. Think about the standard in an other way: suppose we have logo pieces; if we hold one piece of logo in our hand, we can describe how other logo piece should look like to connect to the piece we are holding: that description is the standard, is the contract that the other piece must adhere to.  If we no longer like the color of a logo piece, we can choose an other. Thinking from a Java point of view, a standard is an interface; and a standard provider is a class implementing that interface.

### Tomcat and JEE

When we develop a web application, we need an application server. The application server is an application, installed on a server computer, that waits clients requests received over the internet, routes the requests to the web application, and sends the result our web application produced back to the client.

One JEE standard is [Java servlet specification](https://javaee.github.io/servlet-spec/). This specification describes the interface of [servlets](https://docs.oracle.com/javaee/5/tutorial/doc/bnafe.html). A servlet is, essentially, a Java class that interacts with clients requests; it implements the [Servlet](https://docs.oracle.com/javaee/6/api/javax/servlet/Servlet.html) interface. As we can see, this interface defines several methods: init, destroy, service. So, the servlet class has a life cycle, and a method [service](https://docs.oracle.com/javaee/6/api/javax/servlet/Servlet.html#service(javax.servlet.ServletRequest,%20javax.servlet.ServletResponse)) that actually computes client's request.

The component that manages a servlet's life cycle, and calls the servlet's service method when a new client requests arrives, is called servlet container or servlet engine. When a client requests arrives, the servlet container instantiates the servlet or creates a new thread, and calls the servlet's service method. There are two types or servlet containers: [Web Servers and Application Servers](http://www.baeldung.com/java-servers). An Web Server is an application that provides a servlet engine. An Application Server is an application that provides implementation for all Java EE standards. So, usually, a Web Server is simpler, contains fewer bits than an Application Server. Examples of Web Servers: [Tomcat](https://tomcat.apache.org/), [Jetty](https://www.eclipse.org/jetty/). Examples of Application Servers: [TomEE](http://tomee.apache.org/) (open-source), [GlassFish](https://javaee.github.io/glassfish/) (open-source), [WildFly](http://wildfly.org/) (open-source), [IBM WebSphere Application Server](https://www.ibm.com/cloud/websphere-application-platform) (paid application server), [Oracle WebLogic](http://www.oracle.com/technetwork/middleware/weblogic/overview/index.html). Because a Web Server implements only a subset of the JEE standard, it is usually more lightweight than an Application Server. Sometimes, a Web Server is so lightweight, it can be embedded in a web application itself. The web application with embedded server no longer requires the existence of a servlet container installed on the server; it can be deployed as a simple executable jar file. Jetty is often used as an embedded web server. Sometimes Tomcat is also used as an embedded web server; an example can be found [here](http://blog.sortedset.com/embedded-tomcat-jersey/).

In conclusion, Tomcat is a Web Server, is a lightweight servlet engine. And, by the way, [it's one of the most popular Java application server](https://plumbr.io/blog/java/most-popular-java-application-servers-2017-edition).

### More about servlet engines, standards and so on

Before we move the discussion further, let's talk more about standards. First of all, [here](https://www.mkyong.com/servlet/a-simple-servlet-example-write-deploy-run/) is an example servlet. When a client request arrives, the result will be computed by the doGet method:
{% highlight java %}
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        PrintWriter out = response.getWriter();
        String name = getName();
        out.println("<html>");
        out.println("<body>");
        out.println("<h1>Hello " + name + "!</h1>");
        out.println("</body>");
        out.println("</html>");
    }
{% endhighlight %}
In this example, the result is a simple HTML page. It works, but image we will have to send as response a complex HTML page like this article for example. Imagine how hard it will be to implement and maintain the Java class associated with the servlet. To solve this issue, a new standard emerged, [JSP](http://www.oracle.com/technetwork/java/index-jsp-138231.html). [JSP files](https://en.wikipedia.org/wiki/JavaServer_Pages) are (sort of) HTML files where you can include Java fragments of code (or scriptlet); the scriptlets allows the developer to generate content dynamically. JSP files are translated to servlets at runtime, so in a way are [servlets](http://www.informit.com/articles/article.aspx?p=130980&seqNum=3).

The same HTML page can be rendered by the following JSP code:
{% highlight html %}
<html>
<body>
  <% String message=getMessage() %>
  <h1>${message}</h1>
</body>
</html>>
{% endhighlight %}
As you can see, the code is now simpler, more elegant, with less room for error. As a side note, it's not a good practice to mess view code with business logic. There are mechanisms to avoid such bad practice, like [JavaBeans](https://en.wikipedia.org/wiki/JavaBeans).

So, now that we becomed familiar with some JavaEE standards, we coded a simple application, using several servlets, JSP files. We need to deploy this application to a servlet engine, so we can test how it works. But how do we package the servlets when we want to send them to the application server? The answer is: [WAR file](https://spring.io/understanding/WAR). A WAR (Web application Archive) file is, essentially, an [archive file](https://en.wikipedia.org/wiki/Archive_file) having a predefined structure, documented [here](https://docs.oracle.com/javaee/5/tutorial/doc/bnadx.html).

Here is the structure of a war file:
![Project structure]({{ "/assets/2018-07-08-java-web-application-standards/web_module_structure.png" | absolute_url }})

 As we can see, in the root folder we have HTML files, JSP files, and other static content (javascript and css files, images etc). In the root folder we also have a different folder, WEB-INF. In the WEB-INF folder there is the web.xml file, the web application deployment descriptor. Also, there is the lib folder, where libraries used by the application are put. There is an other folder as well, classes; here, we can find the servlet compiled .class files.

 The [web application deployment descriptor](https://docs.oracle.com/cd/E19226-01/820-7627/bncbj/index.html) is like a map of the application. It defines what servlets are used, what servlets should be called depending on the url the user accessed the file, parameters and so on. For example, if you need to integrate Spring framework, you will introduce in web.xml a configuration similar to this one:
 {% highlight xml %}
 <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
 {% endhighlight %}

 Spring's [ContextLoaderListener](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/context/ContextLoaderListener.html) is a class implementing the JEE interface [ServletContextListener](http://docs.jboss.org/jbossas/javadoc/7.1.2.Final/javax/servlet/ServletContextListener.html); ServletContextListener is an interface that allows a third party to be notified by deployment events. So, when the application is deployed, the application server will instantiate and call lifecycle methods on each class implementing ServletContextListener interface and registered in web.xml file. In the case of Spring, the methods of ContextLoaderListener called at deployment are an entry point for the framework's logic. Spring framework has other entry points as well, depending on the context it's used. More informations about Spring will come soon.

 Before moving the discussion further, I must say something about the servlet standard. Java EE is always moving forward. As an example, Servlet 3.0 specification improved Servlet 2.5 specification in several ways. The the rigid directory structure that we previosly reviewed is law for Servlet 2.5, but is no longer mandatory for Servlet 3.0 specification. The web.xml file, where listeners, filters, url mappings and other variables were defined, is no longer mandatory; thus, it's easier to integrate the application with other libraries and frameworks like Spring. Also, in a big application, the web.xml file was quite huge and complex; maintaining this file is no longer required. [Here](http://www.oracle.com/technetwork/server-storage/ts-5415-159162.pdf) is a more detailed list of improvements.

### Spring and Dependency Injection

Large applications, that sometimes are mission critical for the business they were developped for, are quite complex and difficult to write, modify and maintain. To handle the increasing complexity, the business adapted some [design patterns](https://en.wikipedia.org/wiki/Software_design_pattern), one of the most often used is [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection). [Spring Framework](https://spring.io/) is one of the most used Dependency Injection framework in the Java world. It started as a Dependency Injection framework, but in time more modules were added; nowadays Spring has a lot of modules, you can see the [documentation](https://spring.io/). To understand how Spring become so popular, we need a little history. When JEE was released (it was named J2EE at first) around 1999, it was very complex. A lot of boilerplate, error prone configuration was needed for the most basic tasks. Also, application servers tooked a lot of time to start, so the develop - deploy - test cycle was slow. A more complete list of complains can be found [here](https://thenewstack.io/spring-rod-johnson-enterprise-java/); JEE (J2EE back then) was so difficult to work with, it was nicknamed [Java Evil Edition](https://www.coherentsolutions.com/blog/the-on-going-war-between-jee-and-spring-framework/). To improve the development process of enterprise Java applications, programmer [Rod Johnson](https://en.wikipedia.org/wiki/Rod_Johnson_(programmer)) wrote an excellent book, [Expert One on One J2EE Development without EJB](http://www.wrox.com/WileyCDA/WroxTitle/Expert-One-on-One-J2EE-Development-without-EJB.productCd-0764558315.html); the ideas were translated in a new framework, Spring Framework, first released in 2002. Developers moved from the heavy JEE platform to the lightweight Spring framework. Of course, this created a conflict between JEE and Spring framework; Spring was always one step ahead, and once a new technology was often used in practice (a de-jure standard) JEE committee would standardise it and integrate it in Java (thus it becomed a de-jure standard). For example, [JSR 330](https://www.jcp.org/en/jsr/detail?id=330) was influenced by Spring's Dependency Injection capabilities. Once the standard was released, Spring framework supported that standard too. More informations about history on Spring, the conflict between Spring and JEE, and a comparison between the 2 technologies can be found [here](https://www.coherentsolutions.com/blog/the-on-going-war-between-jee-and-spring-framework/).

### Hibernate and database access

Now that we discussed about Spring, let's talk about an other framework, [Hibernate](https://en.wikipedia.org/wiki/Hibernate_(framework)), first released in 2001 as an alternative to the complexity of J2EE. As it gained popularity in the Java world, it influenced a new JEE standard, [JPA](https://en.wikipedia.org/wiki/Java_Persistence_API). After the release of JPA standard, Hibernate become a JPA provider, like EclipseLink. Many web application need to access an SQL database to fulfill their business logic; and Java web applications are not an exception from this rule. Hibernate facilitates the database access. It is an [ORM](https://en.wikipedia.org/wiki/Object-relational_mapping) framework; in short, the framework translates sql tables into Java classes. The framework does a non-trivial task, so it is quite complex; is is a reason why it gained a lot of critics. You can find more information about this framework [here](http://hibernate.org/orm/). An other reason the framework gained a lot of criticism is it's poor performance. But this performance is due, at least in part, to programmers not familiar with SQL believing that this framework relieves them from thinking about SQL and database access in general. An other critic is that this framework tends to hide the database access entirerly behind an layer of abstraction, and some users are unconfortable with this. If you have to access a SQL database from a Java application, you have to use [JDBC](https://en.wikipedia.org/wiki/Java_Database_Connectivity), there is no way around it. JDBC is a difficult framework to work with; all other frameworks, including Hibernate, use JDBC behind the scenes thus abstracting this complexity for the user. But, for simple tasks, maybe JDBC is a good alternative to Hibernate. Other alternatives include: [Spring JDBC](http://www.baeldung.com/spring-jdbc-jdbctemplate), whitch hides the complexity of JDBC, while the user still works with the database directly and writes SQL queries. An other alternative is [jOOQ](https://www.jooq.org/), a framework that offers a lot of functionalities without hiding the database behind some layers of abstractions. An other alternative worth mentioning is [Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/2.1.0.M3/reference/html/), facilitating the usage of JPA. As in the case of Spring framework, there are many ways to initialize Hibernate framework in a web application; this can be done either by the [Spring Framework](https://stackoverflow.com/questions/42749716/web-xml-settings-with-spring-and-hibernate), either with functionalities provided by [Hibernate alone](http://docs.jboss.org/hibernate/orm/5.3/userguide/html_single/Hibernate_User_Guide.html#bootstrap).

### Conclusions

We discussed about implementing web applications in Java, and about some of the most used technologies. We also observed how things changed fast, how different problems created the need of innovation, and how the flow of innovation created the need for more innovation. Things move fast; when often used frameworks like Hibernate and Spring become de facto standard, actors from industry transforms them into de jure standards; in this time, open sorce frameworks continued to innovate. Things move fast, and recently [Java EE become Jakarta EE](https://www.zdnet.com/article/good-bye-jee-hello-jakarta-ee/). In August 2017 [Oracle open sourced Java EE](https://blogs.oracle.com/theaquarium/opening-up-java-ee), in September 2017 [Oracle announced that the JEE ownership will be transfered to Eclipse Foundation](https://blogs.oracle.com/theaquarium/opening-up-ee-update), and since February 2018 [JEE becomed Jakarta EE](https://www.tomitribe.com/blog/2018/02/java-ee-to-jakarta-ee/); the process of ownership transfer from Oracle to Eclipse Foundation is, at the time this article is written, still ongoing. And many people still refer to this technology as J2EE!

In the end, I want to mention that by no means I covered all the framework and libraries used in developing web applications in Java! I did not mentioned for example anything about [microservice frameworks](https://www.gajotres.net/best-available-java-restful-micro-frameworks/), a trend that caught shape several years ago.


### Reference

https://www.jcp.org/en/home/index

https://plumbr.io/blog/java/most-popular-java-application-servers-2017-edition

https://thenewstack.io/spring-rod-johnson-enterprise-java/

https://www.coherentsolutions.com/blog/the-on-going-war-between-jee-and-spring-framework/

https://www.zdnet.com/article/good-bye-jee-hello-jakarta-ee/

http://www.oracle.com/technetwork/java/javaee/overview/index.html

https://javaee.github.io/servlet-spec/

https://tomcat.apache.org/

https://spring.io/

http://hibernate.org/orm/

https://www.gajotres.net/best-available-java-restful-micro-frameworks/