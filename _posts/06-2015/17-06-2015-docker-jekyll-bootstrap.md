---
layout: post
category : docker
tagline: "Docker, Jekyll and a Bootstrap theme"
tags : [ docker, jekyll, bootstrap, themes]
---
{% include JB/setup %}

##Introducing the 'Dockyllstrap'

This project brings together two existing projects:

1. [Docker & Jekyll](https://github.com/grahamc/docker-jekyll) is an existing docker buildfile for the Jekyll blogging platform.
2. [Jekyll & Bootstrap](https://github.com/plusjade/jekyll-bootstrap/) "The quickest way to start and publish your Jekyll powered blog. 100% compatible with GitHub pages."

###Why
Jekyll has a bunch of great features for blogging, especially if you are a developer, as a static site generator it's fast and it can be hosted for free with github pages. That said it can be difficult to theme, especially if you want to depart from its defaults. Bootstrap meanwhile is a flexible approach to content layout using html and css, there is a lot of good documentation available.
> "One of the hardest things about Jekyll is understanding and mastering the Liquid templating language. 
JB ships with pre-built pages that would make you cry T_T to have to code manually in Liquid."

###How
When using Docker, it's always a good idea to search the [Dockerhub](https://registry.hub.docker.com/) for existing buildfiles. Sure enough someone has already written a [buildfile for all of your Jekyll pre-requisites](https://registry.hub.docker.com/u/grahamc/jekyll/).

Let's start by pulling that base:
```
docker pull grahamc/jekyll
```
Now let's run that container:
```
docker run -d -p 4000:8080 grahamc/jekyll serve
```
Ok, now that it's up and running let's start poking around inside to get the bootstrap theme going. To find the <imagehexnumber> you can run.
```
docker ps
```
Great, now you can see your running jekyll container, let's have a look inside: 
```
docker exec -it <imagehexnumber> bash
```
`rm -rf _site`
```
git clone https://github.com/plusjade/jekyll-bootstrap.git .
```
-Misc tidying up.
`apt-get -qq update`, `apt-get -qq install vim` 
-Add the following to _config.yml.
```
# Serving
detach:  false
port:    8080
host:    0.0.0.0
baseurl: "" # does not include hostname
```
Exit the container
```
CTRL+D
```
Run the Jekyll process
```
docker exec -d <imagehexnumber> jekyll serve
```
Voila Jekyll blog with bootstrap theme is up at http://yourhostname:4000

#Roadmap

Add github pages flavor.
