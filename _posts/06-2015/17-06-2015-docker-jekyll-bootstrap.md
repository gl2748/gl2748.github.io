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

#How
```
docker pull grahamc/jekyll
```
```
docker run -d -p 4000:8080 grahamc/jekyll serve
```
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
