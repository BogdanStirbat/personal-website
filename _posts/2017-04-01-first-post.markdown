---
layout: post
title:  "First blog article (using Jekyll and GitHub pages)"
date:   2017-04-01 15:39:18 +0300
categories: jekyll update
---
I want to share with you my entusiasm in writing my first blog post.

As blogging solution, I wanted something reliable, secure (SSL is becoming mandatory these days), performant, and last but not least, cheap (pay only for the custom domain, if possible).

One clever solution, outlined in this [blog post](https://www.ctheu.com/2017/01/09/comparing-blog-hosting-services-for-developers), was to host a static website on a GitHub repository with GitHub pages enabled. The repository's issues are used as blog posts, and the static website integrates with GitHub API to retrieve the issues content on demand. You can find out more about this solution [here](https://github.com/casualjavascript/blog). It's a clever solution, easy to implement. I wasn't satisfied with the blog posts load times, and I continued searching.

A blog is, in the end, a collection of static web pages, images, css files, and so on. So, using a CMS like Wordpress or Joomla, to generate web pages on demand, would be a waste of database calls and CPU cycles. So, best idea would be to host some static web pages, with a provider, for free. Regarding SSL, [Let's Encrypt](https://letsencrypt.org/) is free.

After some research, I found appealing 2 options. One would be to use [Google App Engine](https://cloud.google.com/). The [App Engine standard environment](https://cloud.google.com/appengine/docs/standard/) offers resources, up to some quotas, for free. Also, having a private space, means I can configure Let's Encrypt, since nobody can access the private key. In this [post](https://boinkor.net/2016/01/hosting-my-blog-on-google-app-engine-with-letsencrypt/) it is described a little Go program, hosted in App Engine, that responds to requests and serves static web pages. The other option was hosting on [GitHub pages](https://pages.github.com/) with [Jekyll](https://jekyllrb.com) website generator. Github offers free SSL certificates for [GitHub domains](https://github.com/blog/2186-https-for-github-pages), but setting a SSL certificate for a custom domain is not possible at this [moment](https://github.com/isaacs/github/issues/156). A workaround exist, via [CloudFlare CDN](https://www.cloudflare.com/). For free, CloudFlare offers an SSL certificate, and CDN. Of course, there is a drawback: traffic is encrypted between blog reader's personal computer and CloudFlare servers, but not between CloudFlare and GitHub.

So, in the end, I had to choose between hosting on App Engine or on GitHub (with CloudFlare). App Engine with Let's Encrypt offered the advantage of full encryption between final user and server. The drawback was performance, hosting on App Engine for free (or any other PAAS provider, like Heroku) would mean slow performance; also, if the blog is not visited in a while, the machine instance could be closed, and next person visiting could experience a long start time. Using GitHub pages with CloudFlare means unencrypted traffic between CloudFlare CDN and GitHub servers. But no sensitive information is transmitted via this blog. Apart form the free SSL, CloudFlare also offers CDN, with translated to decreased load times. And all of this for free!

So, in the end, I used GitHub pages with Jekyll. After playing a little with [Jekyll](https://jekyllrb.com), I discovered what a wonderful solution it is. I simply couldn't believe it was so awesome!

I bought the domain from [Namecheap](https://www.namecheap.com/). Contrary to what the name might suggest, they offer high quality services: the platform is fast, intuitive. Also, the domain was cheap, about 20 dollars (near 95 RON) for a 2 year registration.

After setting up a GitHub pages repository with a Jekyll blog, all I had to do was to transfer nameservers from NameCheap to CloudFlare, to set up the free SSL certificate from CloudFlare, and to configure the GitHub repository to use a custom domain. With excellent help articles from [GitHub](https://help.github.com/articles/using-a-custom-domain-with-github-pages/) and [CloudFlare](https://blog.cloudflare.com/secure-and-fast-github-pages-with-cloudflare/), and generosity of other bloggers describing their [experience](http://davidensinger.com/2014/04/transferring-the-dns-from-namecheap-to-cloudflare-for-github-pages/), everything went nice and smooth.

Edit (2018.04.15): To be GDPR compliant, I removed Disqus comments plugin. For now, if you have comments and suggestions, you can join the discussion [here](https://github.com/BogdanStirbat/BogdanStirbat.github.io/issues).