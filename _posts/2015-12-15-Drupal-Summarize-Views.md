---
layout: post
category : drupal
tagline: "Summarize views"
tags : [drupal, views, drush]
---

Views in drupal are database queries. They can be difficult to manage because they abstract the tricky task of making complex database queries. Preparing for a big site overhaul I found myself writing summaries for existing views on a site. Here we look at a way to get a neat summary of our views.

The first step is to find away to access our views programmatically. To do this we use *[views_get_all_views](https://api.drupal.org/api/views/views.module/function/views_get_all_views/7)*, described as:
> "Return an array of all views as fully loaded $view objects."
With this in mind we initially want to take a look at how these view objects are structured:
`drush ev "print_r(views_get_all_views());"`

Outputs something like this:

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

This gives us some information on the views objects that we have access to. It is not visible in my snippet, but properties of the views object shift between objects and arrays and objects again, this is is pertinent when we *[get/set property values](http://php.net/manual/en/sdo.sample.getset.php)* deeper inside the object. e.g.
`$view->display['default']->display_options['fields']['php']['php_value'],`

Here is our complete script that outputs a table of some pertinent information for all of our views:

    <?php
    $views = [
      [
        'view machine name',
        'view name',
        'view desc',
        'base table',
        'display machine name',
        'display title',
        'display plugin',
        'php field the first',
      ]
    ];
    foreach (views_get_all_views() as $view) {
      foreach ($view->display as $display => $data) {
        $views[] = [
          // 'view machine name'
          $view->name,
          // 'view name'
          $view->human_name,
          // 'view desc'
          $view->description,
          // 'base table'
          $view->base_table,
          // 'display machine name'
          $display,
          // 'display title'
          $data->display_title,
          // 'display plugin'
          $data->display_plugin,
          // 'php value in default field'
          $view->display['default']->display_options['fields']['php']['php_value'],
        ];
      }
    }
    $fp = fopen('views-summary.csv', 'w');
    foreach ($views as $fields) {
        fputcsv($fp, $fields);
    }
    fclose($fp);

We can run this by saving it as a file named 'views-summarize.php' and
`drush php-script views-summarize.php`
