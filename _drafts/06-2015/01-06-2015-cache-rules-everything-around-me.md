---
layout: post
category : drupal
tagline: "Drupal and Docker, working together..."
tags : [ docker containers, bash, drush, cache, drupal]
---
{% include JB/setup %}
### The situation.

Running drupal on a stand-alone Docker Container with a second Docker container running the database. Dockerized setup running on a 512mb VPS. At $5 per month.

### The Problem.

Drupal caches get too big, when cron runs it clears some caches, if these are bloated the database container stops - this is because the server has run out of RAM as the cron process handles the bloated cache tables. Furthermore cron does not clear *all* the cache tables, therefore in time the load on the server increases.

### The solution(s)

1. Upgrade the server. Pros - Fast solution, Cons - costs more money.

2. Write a bashscript to periodically check if the Database container has stopped. 

```

```




