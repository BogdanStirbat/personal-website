---
layout: post
title:  "Introduction to Docker for Java Developers"
date:   2018-01-20 12:25:18 +0300
categories: jekyll update
---

### Introduction

Let's talk about Docker, a technology that was born in 2013, made a huge impact in the software industry, and saw a rapid adoption.
First of all, what is Docker? From the [official documentation](https://www.docker.com/what-docker), it can be seen that Docker has different meanings in different contexts. For now, let's limit ourselves to the following description: Docker is an engine that orchestrates containers. A container is a sort of VM. A sort of, because a container isolates different processes on a machine, rather that virtualizing a new environment. In the Linux kernel, there are 2 features: [cgroups](https://en.wikipedia.org/wiki/Cgroups) and [namespaces](https://en.wikipedia.org/wiki/Linux_namespaces) that allow processes to be isolated. Applications running in different containers are isolated from each other (to a degree: can share ports, if running on same physical machine of course an CPU-intensive app from a container can impact an app running in a different container). This is a reason why we can say a container is a sort of virtual machine. More explanations about containers can be found [here](https://jaxenter.com/nobody-puts-java-container-139373.html).

Now, that we understand what's a container, we can understand why Docker gained so much popularity: because the way it can build, ship, and run applications. The [official documentation](https://docs.docker.com/get-started) describes an image like an executable file. An container is the running instance of an image. Just like more than one process can run the same executable image, more than one container can run a image. A image describes how to create a container. A Docker image is like a source file. One can create a Docker image by adding to an existing Docker image. From a Docker image that runs the Java runtime, we can create a new image that adds a .jar file to that initial image running the Java runtime only. Now, comes the question, how do we share the images? The answer is given by Docker hub, a place where Docker images can be found. A Docker hub is like a Maven repository. The oficial Docker hub can be found [here](https://hub.docker.com/). Everybody can add an image to the Docker hub. Companies, or other parties interested, can add a private image to this hub. [Here](http://blog.codepipes.com/containers/docker-for-java-big-picture.html) can be found a list of Docker-specific terms explained in Java-specific terms.

Now, we understand how Docker builds, ships, and runs applications: an application is build and encapsulated in an image; then, the image is put on Docker Hub; someone else can download the image from the hub and run it (start an container).

<br>

### Running Java applications

Now, that we have a grasp of what Docker is, let's run some containers. The first example will be easy, just to get us started. Assuming you already [installed Docker](https://docs.docker.com/engine/installation/), you can create somewhere on your disk a new folder, create there a file named `Dockerfile`, and add the following content:

{% highlight bash %}
FROM ubuntu:latest

CMD ["/bin/echo", "hello world"]
{% endhighlight %}

The file named `Dockerfile` describes a Docker image. To actually create the Docker image, with the name `helloworld`, run:
{% highlight bash %}
docker image build . -t helloworld
{% endhighlight %}



On the first run, just like Maven, a Docker image with name `ubuntu` and version tag `latest` (just like in Maven, where you can alter the Java source code to get a slightly different jar file, and specify a version for the resulted executable, in Docker you can modify a little Dockerfile and obtain a different image; it's more or less a convention, most recent image of an application has the tag latest) will be downloaded locally. To see the locally available images, run `docker image ls`. You can now run the newly created `helloworld` image, by running:
{% highlight bash %}
docker container run helloworld
{% endhighlight %}

Now, let's play with the tags. Modify the file named Dockerfile, we no longer use `ubuntu` as a base image, but `busybox`. Now, run
{% highlight bash %}
docker image build . -t helloworld:2
{% endhighlight %}

Inspect the output of `docker image ls`. Now, run this version 2 of helloworld image:
{% highlight bash %}
docker container run helloworld:2
{% endhighlight %}

More samples can be found [here](https://docs.docker.com/samples/).

Now, let's run a Java application. Let's create a simple Spring Boot application, and run it in a Docker container. For this, go to [start.spring.io](start.spring.io), from the list of dependencies select Web, give a meaningful Group and Artifact, and generate the project. Open the project in your favorite IDE, and add a controller like this:
{% highlight java %}
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @RequestMapping("/hello")
    public String hello() {
        return "hello";
    }
}

{% endhighlight %}

Now, let's build the project:
{% highlight bash %}
mvn clean install
{% endhighlight %}
To run the project in Docker, first we need to create an image; so, let's add a file named Dockerfile, describing how to create an image running our sample web application. So, in the root folder (where pom.xml is) add a file named Dockerfile with the following content:
{% highlight bash %}
FROM openjdk:latest

COPY target/hello-0.0.1-SNAPSHOT.jar /usr/src/hello-0.0.1-SNAPSHOT.jar

CMD java -jar /usr/src/hello-0.0.1-SNAPSHOT.jar
{% endhighlight %}
We inherit from an image named [openjdk](https://hub.docker.com/_/openjdk/), the official image running Java. We copy the generated jar file into the image, and run the file. Notice a pattern: we compile the project on local machine, and run it in Docker. We don't compile the project in Docker, because we would need a Docker image containing JDK as well, not just JRE. Our image size would increase, unnecessary. Now, let's create the image:
{% highlight bash %}
docker image build -t docker-sample-hello:latest .
{% endhighlight %}

then run it:
{% highlight bash %}
docker run -it -p 8080:8080 docker-sample-hello:latest
{% endhighlight %}

In this `docker run` command, we see 2 additional parameters:
 - `-it`, we need the container to run in interactive mode
 - `-p 8080:8080`, we need to map the container's 8080 port to the local machine's 8080 port.

Now, just go to [http://localhost:8080/hello](http://localhost:8080/hello), and see that everything is running as planned.
[Here](https://github.com/BogdanStirbat/docker-sample/tree/master/hello) you can see a Github repo with this application.

<br>

### An multiple dependencies app

This was a rather simple app. Let's now move to an more real-life scenario. We would need at least a database as dependency, not to mention other services like caching, message queues, and so on. For now, suppose we need a database. We could run the database in an container, the application in an container, and everything would work fine. First, let's see how we would run Postgres DB in an container:
{% highlight bash %}
docker run -it --name postgress-name-example -e POSTGRES_PASSWORD=password -e POSTGRES_USER=postgres -e POSTGRES_DB=name -p 5432:5432 postgres
{% endhighlight %}

In last command, we are mapping Postgres 5432 port to local machine's 5432 port. We also define the user (`POSTGRES_USER=postgres`), password(`POSTGRES_PASSWORD=password`), and database name (`POSTGRES_DB=name`) we want Postgres to run. We also give the container a name, `--name postgress-name-example`.
Now, we can extend our simple Spring Boot application, to store names in a database.
First, add the following to pom.xml, under dependencies:
{% highlight xml %}
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
{% endhighlight %}

Then, add the following entity:
{% highlight java %}
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class Name {

    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private Long id;

    private String message;

    public Name() {

    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
{% endhighlight %}

and the following repository:
{% highlight java %}
import org.springframework.data.repository.CrudRepository;

public interface NameRepository extends CrudRepository<Name, Long> {
}

{% endhighlight %}

You can, now, add the following controller:
{% highlight java %}
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class NameController {

    @Autowired
    private NameRepository nameRepository;

    @GetMapping("/name/{id}")
    @ResponseBody
    public Name findById(@PathVariable Long id) {

        return nameRepository.findById(id).orElseThrow(() -> new RuntimeException("Cannot find a name with id: " + id));
    }

    @PostMapping("/name")
    @ResponseBody
    public Name addName(@RequestBody Name name) {

        return nameRepository.save(name);
    }
{% endhighlight %}

And, to connect to Postgres DB runing in the container, add the following to application.properties file:
{% highlight bash %}
spring.datasource.url=jdbc:postgresql://localhost:5432/name
spring.datasource.username=postgres
spring.datasource.password=password

spring.jpa.hibernate.ddl-auto=create
{% endhighlight %}

Now, you can build the Spring Boot application, and run it on local machine while DB is running in a container. But, we want to run the application in an container as well. We want to expose the 5432 port from DB's container to application's container. Also, the DB and the application are related; we want them to start both and stop both, like a unit. How do we achieve this? Meet [services](https://docs.docker.com/get-started/part3/), and a new file called `docker-compose.yml`. In the root folder, add the file docker-compose.yml with the following content:
{% highlight yaml %}
version: '2'
services:
  app:
    image: docker-sample-hello
    ports:
    - "8080:8080"
    depends_on:
    - mypostgres
  mypostgres:
    image: postgres
    ports:
     - "5432:5432"
    environment:
     - POSTGRES_PASSWORD=password
     - POSTGRES_USER=postgres
     - POSTGRES_DB=name
{% endhighlight %}

Next, we also need to change the application.properties file. Change the following line: `spring.datasource.url=jdbc:postgresql://mypostgres:5432/name`. Next, you need to install [docker-compose](https://docs.docker.com/compose/overview/). Rebuild the docker-sample-hello image, now everything is set to run:
{% highlight bash %}
docker-compose up
{% endhighlight %}

Now, we are running the Java application and the DB in a Docker container, each in it's own container. To see the benefits of our approach, suppose we need to record the number of times our CRUD links are accessed. We will need an other service, [Redis](https://redis.io/). Redis is an open-source, in-memory data structure store. We will use this solution to store and update the statistics. To include a Redis container, we only have to add the following lines at the end of docker-compose.yml:
{% highlight yaml %}
  redis:
    image: redis
{% endhighlight %}

Now, we will add to application.properties:
{% highlight bash %}
spring.session.store-type=redis
server.session.timeout=5
spring.redis.host=redis
spring.redis.port=6379
{% endhighlight %}

Also, don't forget about pom.xml:
{% highlight xml %}
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
{% endhighlight %}


Now, Spring Boot magic created an bean of type `RedisTemplate<String, String>`. As side note, a bean of type `RedisTemplate<String, Integer>` will not be magically created (at least, at the time of writing). So, now, we can add a controller, to view the statistics:
{% highlight java %}
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class StatisticsController {

    private static final String GET_KEY = "GET_KEY";
    private static final String POST_KEY = "POST_KEY";

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    @RequestMapping("/statistics")
    public String statistics() {
        String numberOfGets = redisTemplate.opsForValue().get(GET_KEY);
        String numberOfPosts = redisTemplate.opsForValue().get(POST_KEY);

        return  "{" + "GET: " + numberOfGets + ", POST: " + numberOfPosts + "}";
    }
}
{% endhighlight %}

After the controller for saving and retrieving names is also adjusted, we can rebuild the project, the Docker image. After we access several resources, we can access [http://localhost:8080/statistics](http://localhost:8080/statistics) and see something like:
{% highlight json %}
{
    GET: 3,
    POST: 2
}
{% endhighlight %}

A working example can be found [here](https://github.com/BogdanStirbat/docker-sample/tree/master/name).

<br>

### Conclusions

For development, Docker brings a huge advantage: we can test technologies, like Redis, Postgres, MongoDB, without installing anything on our machines. When working in a team, we can create a Docker environment, so every newcomer will no longer have to spend a lot of time installing everything. When it comes to running Docker in production, there is a longer discussion. In a cloud environment, Docker can be an advantage; but, as always, you should look at [the big picture](http://blog.codepipes.com/containers/docker-for-java-big-picture.html).


<br>
<br>

### Some useful links

https://jaxenter.com/nobody-puts-java-container-139373.html

http://blog.codepipes.com/containers/docker-for-java-big-picture.html

https://en.wikipedia.org/wiki/Docker_(software)