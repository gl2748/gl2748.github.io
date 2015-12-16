---
layout: post
category : drupal
tagline: "Summarize views"
tags : [drupal, views, drush]
---

Views in drupal are database queries. Because they simplify the difficult task of making mysql joins etc... they can themselves be tricky to manage. Preparing for a big site overhaul I found myself writing summaries for existing views on a site. Here we look at a way to get a neat summary of our views. 

The first step is to find away to access our views programmatically. To do this we use [views_get_all_views](https://api.drupal.org/api/views/views.module/function/views_get_all_views/7), described as:
> Return an array of all views as fully loaded $view objects.
With this in mind we initially want to take a look at how these view objects are structured:
```php
drush ev "print_r(views_get_all_views());"
```
Outputs something like this:
```php
Array
(
    [nodequeue_2] => view Object
        (
            [db_table] => views_view
            [base_table] => node
            [base_field] => nid
            [name] => nodequeue_2
            [vid] => 1
            [description] => Display a list of all nodes in queue 'Home Left Column'
            [tag] => nodequeue
            [human_name] => Home Left Column
            [core] => 0
            [api_version] => 
            [disabled] => 1
            [built] => 
            [executed] => 
            [editing] => 
            [args] => Array
            ...
```

