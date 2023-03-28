---
layout: post
title:  "Comparison between Angular and React"
date:   2020-06-16 21:05:18 +0300
categories: jekyll update
---


After [this](https://github.com/BogdanStirbat/SupermarketScheduler) experiment, in which I played with React, I did [an other experiment](https://github.com/BogdanStirbat/SupermarketSupplier), this time playing with Angular.

After playing a bit with the 2 different frameworks, I'm making a comparison between the 2 frameworks.

### Similarities

Both frameworks were developed for writing SPA applications. Using any of the 2 frameworks, the HTML code is no longer generate on the server side, but on the 
client side. An observation should be make: client side rendering hurts SEO performance, and for this reason both frameworks offer the capacity to render the 
HTML on the server side, a functionality called server side rendering. 

### Differences

The first noted difference is that Angular is a full, opinionated framework, while React is actually a library. There are several implications of this observation:
 - projects using Angular have a standard structure, while there there is no standard (or recommended) structure for a React project; thus, it's easier to understand a 
 new Angular project than to understand a new React project.
 - Angular offers a lot of functionality in the framework itself, while in React a new dependency might be needed for some extra functionality. Example: routing; while 
 Angular offers routing out of the box, in React there is a different library for the same thing: [React Router](https://reacttraining.com/react-router/web/guides/quick-start).
 - it's easy to learn React, and to start a new project; Angular, on the other side, has a steep learning curve, the developer needs to read more documentation an understand more 
 concepts before he will be able to start a new project.
 - on the other hand, progress in developing a new Angular application is more constant, not only due to it's standard structure, but also due to the fact that, by design, 
 Angular enforces best software engineering practices; while it's easier to get started a new React project, sometimes the developer might need to stop, search for a library, 
 add it as dependency and learn it.
 
An other major difference is that Angular projects are written in Typescript, while React projects in Javascript. [Typescript](https://www.typescriptlang.org/) is a programming 
language invented for addressing some limitations in the Javascript language. Thus, Typescript has some improvements over Javascript, beneficial specially in big projects: it's a compiled
language, it has types thus some problems are detected at compile time, and so on.

One thing to mention is that Angular has special classes for different purposes: components (responsible with UI elements), services (functionality not necessarily associated with an UI element,
but with functionality that can be used in different components). Angular also has built in support for Dependency Injection, thus enforcing software engineering best practices. 
Thus, an Angular application will look familiar to an Java software developer. 

React uses the concept of Virtual DOM. A virtual DOM is a DOM representation that is kept in memory only; it tracks changes, so only the elements of the real DOM that changed are re-render. 
Angular doesn't has this concept; so, at least in theory, a React app has better performance than an Angular app.

### Conclusion

Angular and React are different technologies for writing SPA applications. 
Which one is better? It depends on specific projects. While some are written and maintained by one man or a handful few, some others are developed by multiple teams, or are just more complex. 
React might be a better choice in the first scenario, due to ease of developing, and in the second scenario Angular might be a better one.