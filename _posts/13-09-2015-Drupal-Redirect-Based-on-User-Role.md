---
layout: post
category : drupal
tagline: "Redirecting users  based on their role"
tags : [drupal, php]
---

{% highlight php %}
<?php

/* Helper function
 * If user has any role return TRUE.
 */

function _query_user_has_role($user, $role){
  if (in_array($role, $user->roles)) {
    return TRUE;
   }
  return FALSE;
}

/*
 * Implementation of hook_menu()
 */

function psilabs_frontdirector_menu() {
  $items['frontdirector'] = array(
    'page callback' => '_psilabs_frontdirector_page',
    'access callback' => TRUE,
    //because we do not want this menu item displaying in the 'navigation' menu we assign it an arbitrary non-existant menu... 
    'menu_name' => 'hidden_menu',
  );
  return $items;
}

/*
 * If user has role Client_User redirect to manage-my-samples
 * If user has role Data Manager or Data Viewer redirect to manage samples.
 * If user has no role redirect to html 403 access denied.
 */

function _psilabs_frontdirector_page() {
  global $user;

  //client user
  if (_query_user_has_role($user, 'client_user')) {
    drupal_goto('manage-my-samples');
    return;
  }
  //data manager user
  if (_query_user_has_role($user, 'data manager')) {
    drupal_goto('manage-samples');
    return;
  }
  //data viewer user
  if (_query_user_has_role($user, 'data viewer')) {
    drupal_goto('manage-samples');
    return;
  }
  else {
    drupal_access_denied();
  }
}
{% endhighlight %}
