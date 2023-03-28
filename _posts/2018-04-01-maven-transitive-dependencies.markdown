---
layout: post
title:  "Maven transitive dependencies with conflicting versions"
date:   2018-04-01 18:25:18 +0300
categories: jekyll update
---

### Introduction

As a Java developer, I mainly use [Maven](https://maven.apache.org/) as a build tool, at work and at home, and for good reasons. Released in [2004](https://en.wikipedia.org/wiki/Apache_Maven), Maven has become the de facto standard build tool for Java projects.

Maven's popularity speaks for itself about this tool's innovations. Why Maven had such a great success among Java developers? If you ask the typical Java developer what is Maven, he will tell you that Maven is a build tool. But Maven is more than a build tool. It also [runs reports, generates sites, and so on](http://books.sonatype.com/mvnref-book/reference/introduction-sect-whatIsMaven.html). Maven's responsibility is not limited to compiling source code and packaging it. Maven brings a concept called "Project Object Model". It is a standard, so it is invariable among Java projects. This is good news: if you switch to a new project, you already know how to build it, what libraries it depends on and so on. The file describing a Maven project is called "pom.xml". Here, the project's groupId, artifactId, version are defined; the 3 values uniquely refer a project. Dependencies are also listed, with the corresponding version. In this file, plugins can be configured. Maven uses a "convention over configuration" philosophy. If the default options do not satisfy a project's requirements, a Maven build can be configured with plugins. Plugins are declared and configured in the pom.xml file. There are plugins for almost any scenario. In fact, Maven can be seen as a plugin engine. Every task Maven does, is done via a [plugin](https://maven.apache.org/plugins/index.html): [cleaning a build](https://maven.apache.org/plugins/maven-clean-plugin/), [compiling the source code](https://maven.apache.org/plugins/maven-compiler-plugin/), [running unit tests](https://maven.apache.org/surefire/maven-surefire-plugin/) and so on. Artifacts such as libraries and plugins are kept in the Maven repository. It's a big plus, the plugins and libraries are managed in the same way.

If, in my project, I need to use [Spring framework](https://mvnrepository.com/artifact/org.springframework/spring-web), the dependency just needs to be declared in the pom.xml file. Further, Maven will take care of downloading this framework, packaging it in my application.


### Transitive dependencies

There are 2 types of Maven dependencies: direct and transitive. If, a pom.xml includes the following:
{% highlight xml %}
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.4.RELEASE</version>
</dependency>
{% endhighlight %}
then spring-web becomes a direct dependency. As we can [see](https://mvnrepository.com/artifact/org.springframework/spring-web/5.0.4.RELEASE), spring-web also depends on [jackson-databind](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind/2.9.4). Since spring-web declared this as a dependency, Maven will automatically download and include in our application jackson-databind as well; this is a transitive dependency. So, direct dependencies are directly included by the end applications, and transitive dependencies are direct dependency's dependencies. [jackson-databind](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind/2.9.4) depends on [jackson-core](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core/2.9.4). So, jackson-core is a transitive dependency for our application as well. As we can see, we have a graph of dependencies, of arbitrary length.


### Transitive dependencies with conflicting versions

In this [Github repository](https://github.com/BogdanStirbat/transitive-dependency-study) I created a Maven project with the following structure:

![Project structure]({{ "/assets/2018-04-01-maven-transitive-dependencies/transitive-dependency-study.png" | absolute_url }})

Above image was created with [draw.io](https://www.draw.io/). As can be seen from the picture above, this project is a case study for a conflicting transitive dependency. The repository consists of 4 Maven projects:

[message-library](https://github.com/BogdanStirbat/transitive-dependency-study/tree/master/message-library)

This library exists, in my local Maven repository, in 2 versions. The library has, for demo purposes, the following class:
{% highlight java %}
package com.bstirbat.transitive.dependency.study;

public class MessageLibrary {

    public static String getMessage() {
        return "message-version-1";
    }
}

{% endhighlight %}

Locally, I changed the project's version to `2.0-SNAPSHOT`, then I modified the return value: `message-version-2`. I recompiled the project, thus in my local Maven repository there are 2 versions of this library:
{% highlight bash %}
ls ~/.m2/repository/com/bstirbat/transitive/dependency/study/message-library/
1.0-SNAPSHOT  2.0-SNAPSHOT  maven-metadata-local.xml
{% endhighlight %}

[message-service-v1](https://github.com/BogdanStirbat/transitive-dependency-study/tree/master/message-service-v1)

[message-service-v2](https://github.com/BogdanStirbat/transitive-dependency-study/tree/master/message-service-v2)

Each 'service' project has a dependency to message-library: as you probably guessed, message-service-v1 depends on message-library version 1 and message-service-v2 depends on message-library version 2. Each project has a method like this:
{% highlight java %}

public static String getMessage() {
    return MessageLibrary.getMessage();
}

{% endhighlight %}

[message-web-application](https://github.com/BogdanStirbat/transitive-dependency-study/tree/master/message-web-application)

This is a Spring boot application. It has a controller class, with following content:
{% highlight java %}

import com.bstirbat.transitive.dependency.study.MessageServiceV1;
import com.bstirbat.transitive.dependency.study.MessageServiceV2;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MessageController {

    @GetMapping("/message/v1")
    public String getMessageV1() {
        return MessageServiceV1.getMessage();
    }

    @GetMapping("/message/v2")
    public String getMessageV2() {
        return MessageServiceV2.getMessage();
    }
}

{% endhighlight %}

As you probably assume, calling method `getMessageV1()` will return `message-version-1` (since message-service-v1 depends on message-library version 1), and calling method `getMessageV2()` will return `message-version-2` (since message-service-v2 depends on message-library version 2). If we build this spring application and run it, we will have a surprise: both `getMessageV1()` and `getMessageV2()` return `message-version-1`.


### What happened?
[message-service-v1](https://github.com/BogdanStirbat/transitive-dependency-study/tree/master/message-service-v1) and [message-service-v2](https://github.com/BogdanStirbat/transitive-dependency-study/tree/master/message-service-v2) both depend on [message-library](https://github.com/BogdanStirbat/transitive-dependency-study/tree/master/message-library), but with different versions. JVM can only load a specific version of a library. Maven has no other choice but to include the library only once, with a specific version. This is not a Maven limitation, but a JVM limitation. As a side note, Java 9's module system fixed this problem, but this discussion is beyond the scope of this blog article. Thus, message-library will be included only once, with a version; the other version will be ignored, and projects depending on a specific library version will receive a different library version. This can be a big problem: what if logic changed between versions, or what if methods or classes were deleted, and one of our library calls a non-existing method? Meet a class of exception that occur at runtime: [ClassNotFoundException](https://docs.oracle.com/javase/7/docs/api/java/lang/ClassNotFoundException.html), [MethodNotFoundException](https://docs.oracle.com/javaee/7/api/javax/el/MethodNotFoundException.html). Unfortunately, this problems are not visible at compile time. It could even be possible that the excluded version has some additional methods and classes, missing in the included version. The compiler will not detect any problem. The issues will be detected in integration tests, if we are lucky; else, the frustrated user will experience them.

How does Maven picks a specific version? The algorithm that describes which version wins can be found [here](http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#Transitive_Dependencies).

In real-life, we can have projects with hundreds of library dependencies, so losing track of them can be quite easy. How can we easily detect such conflicts?


### Solution
Meet [Maven Enforcer Plugin - The Loving Iron Fist of Maven](http://maven.apache.org/enforcer/maven-enforcer-plugin/index.html). To include it in our web application, we have to add following lines to pom.xml file:
{% highlight xml %}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>1.4.1</version>
    <configuration>
        <rules><dependencyConvergence/></rules>
    </configuration>
</plugin>
{% endhighlight %}

Now, if we run:
{% highlight bash %}
mvn enforcer:enforce
{% endhighlight %}

we will get a pretty informative message:
{% highlight bash %}
[INFO] --- maven-enforcer-plugin:1.4.1:enforce (default-cli) @ message-web-application ---
[WARNING]
Dependency convergence error for com.bstirbat.transitive.dependency.study:message-library:1.0-SNAPSHOT paths to dependency are:
+-com.bstirbat.transitive.dependency.study:message-web-application:0.0.1-SNAPSHOT
  +-com.bstirbat.transitive.dependency.study:message-service-v1:1.0-SNAPSHOT
    +-com.bstirbat.transitive.dependency.study:message-library:1.0-SNAPSHOT
and
+-com.bstirbat.transitive.dependency.study:message-web-application:0.0.1-SNAPSHOT
  +-com.bstirbat.transitive.dependency.study:message-service-v2:1.0-SNAPSHOT
    +-com.bstirbat.transitive.dependency.study:message-library:2.0-SNAPSHOT

[WARNING] Rule 0: org.apache.maven.plugins.enforcer.DependencyConvergence failed with message:
Failed while enforcing releasability the error(s) are [
Dependency convergence error for com.bstirbat.transitive.dependency.study:message-library:1.0-SNAPSHOT paths to dependency are:
+-com.bstirbat.transitive.dependency.study:message-web-application:0.0.1-SNAPSHOT
  +-com.bstirbat.transitive.dependency.study:message-service-v1:1.0-SNAPSHOT
    +-com.bstirbat.transitive.dependency.study:message-library:1.0-SNAPSHOT
and
+-com.bstirbat.transitive.dependency.study:message-web-application:0.0.1-SNAPSHOT
  +-com.bstirbat.transitive.dependency.study:message-service-v2:1.0-SNAPSHOT
    +-com.bstirbat.transitive.dependency.study:message-library:2.0-SNAPSHOT
]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
{% endhighlight %}

Now, the nasty problems are detected while building the project. The displayed messages are very informative, we can see what libraries have conflicting versions, and the whole dependency graph.


Now, we have to solve the problem. There are many solutions, some strategies for solving such problems can be found [here](http://books.sonatype.com/mvnref-book/reference/pom-relationships-sect-project-dependencies.html#pom-relationships-sect-conflict).


<br>
<br>

### Some useful links

https://www.ricston.com/blog/solving-dependency-conflicts-maven/

https://bryantsai.com/how-to-resolve-dependency-conflict-out-of-your-control-e75ace79e54f

https://blog.sonatype.com/2009/12/maven-dependency-resolution-a-repository-perspective/

https://blog.mafr.de/2014/08/30/maven-discovering-dependency-conflicts/

https://www.davidjhay.com/maven-dependency-management/

http://techidiocy.com/maven-dependency-version-conflict-problem-and-resolution/

https://labs.spotify.com/2015/09/01/java-linking/

http://blog.florian-hopf.de/2014/01/analyze-your-maven-project-dependencies.html

https://carlosbecker.com/posts/maven-dependency-hell/

http://immutables.pl/2015/03/30/Resolving-dependency-conflicts-in-maven/

https://maven.apache.org/plugins/maven-dependency-plugin/examples/resolving-conflicts-using-the-dependency-tree.html

https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html

http://books.sonatype.com/mvnref-book/reference/index.html