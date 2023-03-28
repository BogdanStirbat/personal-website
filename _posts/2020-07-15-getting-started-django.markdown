---
layout: post
title:  "Getting started with Django"
date:   2020-07-15 21:05:18 +0300
categories: jekyll update
---


### Introduction

[Django](https://www.djangoproject.com/) is a web framework for the Python world. Django's motto is (from the website): "The web framework for perfectionists with deadlines.". 

I decided to play a bit with this framework, to satisfy my curiosity. The end result's repo is [here](https://github.com/BogdanStirbat/django-example).

Before explaining how to start a new Django project, let's talk about Python's dependency management. 

### Python virtual environments

To install a new library, in Python, there is available a command that can be used out of the box: `pip`. 
Using this command, as it is, is problematic because all libraries are installed system wised. Suppose that you have 2 projects using Python, on your system. 
One of them is using a specific library, but you need a newer version of the same library for the other project. By upgrading the library to a newer version, system wise, you can break the first project.

To solve this problem, there was invented a new concept: virtual environments. A virtual environment in an environment isolated from the rest of the system. 
Having an isolated environment gives you the freedom to experiment, without the fear of breaking any other project or the system itself. 

Virtual environments can be archived using a special library, [virtualenv](https://virtualenv.pypa.io/en/latest/); many features of this library have been ported into Python [venv](https://docs.python.org/3.4/library/venv.html) module, 
thus available by default without the need of installing anything new. It will be show later, in this blog, how to use venv.

### Django framework

Django is a well known framework for the Python world, pragmatic and easy to work with. More information about the framework can be found [here](https://www.djangoproject.com/) .

In this example, Django will be installed and used in an virtual environment, using venv.

To start a new virtual environment:

 - `python3.8 -m venv venv`
  
Here, on my Ubuntu system, I'm using Python 3.8 to start a new virtual environment called venv.

Next, to activate it:

- `. venv/bin/activate`

You can see that the prompt changed. We are now in a virtual environment. 

Now, we can simply install Django:

 - `pip install Django==3.0.8`
 
Any Django application is part of a Django project. There can be one or many other Django applications for a Django project. So, next, we need to start a new Django project:

- `django-admin startproject mydjangosite`

Here, we are starting a new project named mydjangosite. In the folder mydjangosite, among other things, there is a Python script called manage.py. 

We can use this script to start the server:

- `python manage.py runserver`

or to start a new application:

- `python manage.py startapp django_hello_world`

To exit the virtual environment, you just need to run `deactivate`. 

### Conclusion

Django is a pragmatic, easy to learn, well known and used in practice, web framework.

It's a very good framework for blogs, news sites, management systems and so on. After playing with it, my impression is that indeed Django is a web framework for perfectionists with deadlines.