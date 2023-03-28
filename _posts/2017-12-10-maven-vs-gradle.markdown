---
layout: post
title:  "Maven and Gradle: a comparison"
date:   2017-12-10 14:23:18 +0300
categories: jekyll update
---


In this blog post, I will talk about 2 build tools in the Java ecosystem: [Maven](https://maven.apache.org/) and [Gradle](https://gradle.org/).

As a Java developer, at my daily job I use Maven. But, while playing with some open source projects, I came in contact with the Gradle build tool. And, I decided to compare the 2 build tools, to figure out with one is better.

First, some words about Maven. Maven was released in [2004](https://en.wikipedia.org/wiki/Apache_Maven), after Apache Ant. It has an convention over configuration style. Developers write XML files in order to configure a Maven build. It also uses a linear, standard, lifecycle. All these characteristics make Maven builds similar across different projects. And this is a good thing, developers spend little time to familiarize with a new project, so the time to learn the tool is well spent. Unfortunately, this also makes Maven rigid, and for more complex cases like a library this might make it hard to work with, or even impossible.

Gradle was released in [2007](https://en.wikipedia.org/wiki/Gradle), 4 years after Maven. Unlike Maven, Gradle uses a programming language (Groovy) for writing build files. Users are specifying builds using a Groovy-based DSL, but they can also write Groovy executable code. This makes Groove easier to adjust, and for complex builds this might be the answer. But, there is a price to pay: developers need time to familiarize with a new build; and what if there is an bug in the build file? Unlike Maven, with uses a fixed, linear set of phases, a Gradle build is a DAG of task dependencies. Once a task is run, it's output can be cached so future builds run faster. Thus, Gradle builds can run faster, but at the expense of more memory.


### Conclusions
In the end, with build tool is better? The answer is, it depends, including on personal preferences. For standard JavaEE projects, where a build's output is one or more war or jar files, Maven can be more than enough. Combined with its simplicity and conventions, it can be the perfect tool. This is why, across industry, Maven is a de facto standard. But, for an open-source project or a complicated build, or because developers don't like to use XML, or want to benefit from Gradle performance improvements, Gradle can be better suited.


### Bibliography

https://guides.gradle.org/migrating-from-maven/

https://gradle.org/maven-vs-gradle/

https://technologyconversations.com/2014/06/18/build-tools/

https://stackify.com/gradle-vs-maven/

http://letstalkdata.com/2017/04/java-build-tools-ant-vs-maven-vs-gradle/

https://www.softwareyoga.com/10-reasons-why-we-chose-maven-over-gradle/

https://developer.jboss.org/wiki/GradleWhy?_sscc=t