---
layout: post
title:  "Writing a SPA application using Spring Boot and React"
date:   2020-05-17 14:49:18 +0300
categories: jekyll update
---


In this post, it will be described a SPA application using Spring Boot and React. The application, 'Supermarket Scheduler', allows users to make appointments
before going to the supermarket, thus reducing crowd. There are 2 types of users:
 - regular users, the clients of the supermarket; they can view supermarkets, available time slots, and make appointments
 - manager users, the administrators; they can add new users (managers or regular), they can register supermarkets and time slots
 
The application consists of 2 smaller applications:
 - the backend application
 - the frontend application
 

### The backend application

The backend application exposes a REST API for fulfilling the business logic. It allows authentication using JWT, authorization, and all functionality is allowed based on user type.

The application uses Spring Boot, Spring Data JPA, and a PostgreSQL database.

### The frontend application

The frontend application uses React. This application was created using the `npx` command. Next, to start a development server, just run `npm start`. 

In the end, this application will consist of a list of static files. In production, there will be an additional server (using node, or Nginx) to deliver the files.

Of course, it would be possible to configure the Spring Boot app to serve the static files too; but I think that each component should do one thing and should do it well: the Spring Boot app offers the API, and some other server offers the static files.

### React intro

React is an a Javascript library for building user interfaces running in the browser. It is a component-based framework. 

In order to use React, you can have a HTML page like this:

{% highlight html linenos %}

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>

{% endhighlight %}

this .html file can be linked with the following code:

{% highlight javascript linenos %}

const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));

{% endhighlight %}

The `element` constant is a component. In React, there are 2 types of components:
 - function components
 - state components
 
An example of a function component:

{% highlight javascript linenos %}

function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

{% endhighlight %}

An example of a class component:

{% highlight javascript linenos %}

class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

{% endhighlight %}

One difference between function components and class components is that class components have state; class components have 
lifecycle methods as well.

The components can be reused, and this is an aspect that gives React elegance:

{% highlight javascript linenos %}

function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}
 
{% endhighlight %}


Components can have state; once the state modifies, the affected parts are re-rendered; but not everything is re-rendered, only the elements affected by the change.
To figure out the parts needed to re-render after a property change, React uses a Virtual DOM.

State can only travel from parent components to child components; but parent components can pass child components methods as well, and a child component can call these methods to notify the parent when state changes.

React uses ES6.

More information about React, including official tutorials and guidelines, can be found [here](https://reactjs.org/) .

### React Router

React is a library that allows the creation of SPA applications. By definition, SPA run in a single web page, so the concept of URL links is irrelevant (at least in the classic sense).

If you want to have links in your SPA application, that users can bookmark, or can click browser's Back button to return from where they left, you need to interact with an API provided by the browser.

A useful library for doing  this is [React Router](https://reacttraining.com/react-router/web/guides/quick-start); using this library, different browser URL's can point to different parts of your React application.

Example:

{% highlight javascript linenos %}

function LinksExample() {
  return (
    <Router>
      <div>
        <ul>
          <li>
            <Link to="/">Home</Link>
          </li>
          <li>
            <Link to="/about">About</Link>
          </li>
          <li>
            <Link to="/dashboard">Dashboard</Link>
          </li>
        </ul>
        <Switch>
          <Route exact path="/">
            <Home />
          </Route>
          <Route path="/about">
            <About />
          </Route>
          <Route path="/dashboard">
            <Dashboard />
          </Route>
        </Switch>
      </div>
    </Router>
  );
}

function Home() {
  return (
    <div>
      <h2>Home</h2>
    </div>
  );
}

function About() {
  return (
    <div>
      <h2>About</h2>
    </div>
  );
}

function Dashboard() {
  return (
    <div>
      <h2>Dashboard</h2>
    </div>
  );
}

{% endhighlight %}


### Screenshots

Home page for an not logged in user:

![Project structure]({{ "/assets/2020-05-17-spa-application-spring-boot-react/front_page_anonim_user.png" | absolute_url }})

Supermarket view for an not logged in user:

![Project structure]({{ "/assets/2020-05-17-spa-application-spring-boot-react/supermarkets_anonim_user.png" | absolute_url }})

Home page for an regular user:

![Project structure]({{ "/assets/2020-05-17-spa-application-spring-boot-react/front_page_regular_user.png" | absolute_url }})

Supermarket view for an regular user:

![Project structure]({{ "/assets/2020-05-17-spa-application-spring-boot-react/supermarkets_regular_user.png" | absolute_url }})

Appointments view for an regular user:

![Project structure]({{ "/assets/2020-05-17-spa-application-spring-boot-react/appointments_regular_user.png" | absolute_url }})

Home page for an manager user:

![Project structure]({{ "/assets/2020-05-17-spa-application-spring-boot-react/front_page_manager_user.png" | absolute_url }})

Supermarket view for an manager user (no time slots created yet for the viewed supermarket):

![Project structure]({{ "/assets/2020-05-17-spa-application-spring-boot-react/supermarkets_manager_user.png" | absolute_url }})


### Conclusion

React is a framework that is fun to work with. It allows easy development of SPA applications. It has a great community behind it, with many libraries and contributors.