---
layout: post
title:  "Writing a WordPress theme"
date:   2020-09-13 12:16:18 +0300
categories: jekyll update
---


After playing a little bit with WordPress, I've decided to write a simple WordPress theme for getting familiar with the process.

Since I'm running this blog, I've decided to use the [Jekyll minima](https://github.com/jekyll/minima) as a source of inspiration. 
This exercise's result is the [minimajekyll](https://github.com/BogdanStirbat/minimajekyll) WordPress theme.

The first step im developing the theme was to write a static website, so that I will know what CSS and HTML structure the theme will be using.

Then, the actual coding started. I used, as starting theme, the [Twenty Twenty](https://wordpress.org/themes/twentytwenty/) WordPress theme. 
I've downloaded the Twenty Twenty theme, extracted it, and renamed the folder to minimajekyll since that's the name of the new theme I wanted to create.

Next, I've moved the minimajekyll folder to the folder `/var/www/html/wordpress/wp-content/themes` on my computer, so that I could test the result of my changes
as I made them, on my local WordPress installation.

After creating, on my local WordPress, several posts similar to the posts I regularly publish, I was ready to make the actual code changes.

The first change was to replace all `twentytwenty` occurrences (with different caps) to `minimajekyll`. 
After testing that everything still works ok, I've updated the `style.css` file in order to update the theme's look and feel. 
Then, I've updated the `header.php`, `footer.php`, `index.php` and several other files, in order to have the desired appearance. 
This process involved reading the existing code, updating it, testing changes; it was a trial and error process.
Last step involved some cleanup: there were several files that I didn't used, so I deleted them.

In conclusion, writing a new WordPress theme is a fun process. A prerequisite is having a static site, so that the layout is known before theme development starts.