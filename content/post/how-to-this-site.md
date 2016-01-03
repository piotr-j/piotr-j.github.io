+++
author = "PiotrJ"
date = "2016-01-03T12:08:42+01:00"
description = "or close?"
tags = ["webdev"]
title = "How to: this website"

+++

# How this site was made

I'm not very experianced web developer, so process of making this website was a bit frustrating. Actually, deployment was the most annoying part.
Lets start with Hugo.

# Hugo

[Hugo](https://gohugo.io/) is a static site generator written in [Go](https://golang.org/) programming language. Hugo is still in early stages, but it has good documentation and active community.
Check [Hugo quickstart](https://gohugo.io/overview/quickstart/) for a short guide how to create your first website in few minutes.

# Github pages

This page is hosted on github pages, as user page. [Github pages quick guide](https://pages.github.com/).

# Deployment

Deployment is done with [wercker](http://wercker.com/), an automation tool. [This guide](https://gohugo.io/tutorials/automated-deployments/) is a great starting point.
Assuming that you have read the guide, here are some gotchas that I've found important:

- Don't forget to remove the hidden ```.git``` folder from cloned theme, as it will cause the theme to be not available in ```wercker``` environment
    
```git
rm -rf themes/herring-cove/.git
```


- User/organization github site is located in ```master``` branch of your repo, __not__ ```gh-pages``` like project sites. 
This means that you should put your source files in different branch, I've done it in ```source``` branch. Master branch should be empty now, it will be overriden anyway.

```git
git checkout --orphan source
git add .
git commit -m "Added site sources"
git push -u origin source
```


- Below is my wercker.yml files. I use ```golang``` box to save few seconds on build time. Pygments are disabled to save time as well, theme I'm using has ```highlight.js``` built-in.
The theme has a ```-clean``` suffix, as I forgot to remove ```.git``` from it first time. I'm using different ```gh-pages``` script, as I had trouble with the one linked in official guide 
with user page.

```yml
box: golang:1.5.1
build:
  steps:
    - arjen/hugo-build:
        version: "0.15"
        theme: "ghostwriter-clean"
        disable_pygments: true
deploy:
  steps:
    - install-packages:
        packages: git ssh-client
    - uetchy/gh-pages@0.3.1:
        token: $GIT_TOKEN
        domain: piotrjastrzebski.io
        path: public/
```

- on wercker site, in application settings -> options -> ignored branches add ```master``` 

- in application settings -> targets -> your deploy target, change ```master``` to your custom branch, ```source``` in this example
![dns](/img/targets.jpg)


# Custom domain

Last piece of the puzzle, is custom domain support, if you need it. From source side, setting your domain in ```wercker.yml``` is all that is required.

```yml
    - uetchy/gh-pages@0.3.1:
        token: $GIT_TOKEN
        domain: your-domain-here
        path: public/
```

The hard part is, wrestling with your domain registrar to setup dns correctly. Here are the settings I've used:
![dns](/img/dns.jpg)

Read more in [github pages documentation](https://help.github.com/articles/setting-up-a-custom-domain-with-github-pages/)

# Final source

Current source for this website is available [here](https://github.com/piotr-j/piotr-j.github.io).

