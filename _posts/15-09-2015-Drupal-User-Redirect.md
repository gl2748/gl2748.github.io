---
layout: post
category : drupal
tagline: "Redirecting users based on their role"
tags : [drupal, php]
---

In this code snippet we check the user role and redirect them to a different page depending on their user role.
Usefully, this approach can be used to redirect different users to different homepages upon login. This is done by setting the global site homepage to our hook_menu path, in this case /frontdirector.

```php
<?php

/**
 * Helper function
 * If user has any role return TRUE.
 */

function _query_user_has_role($user, $role){
  if (in_array($role, $user->roles)) {
    return TRUE;
   }
  return FALSE;
}

/**
 * Implements hook_menu()
 */

function psilabs_frontdirector_menu() {
  $items['frontdirector'] = array(
    'page callback' => '_psilabs_frontdirector_page',
    'access callback' => TRUE,
    'menu_name' => 'hidden_menu',
  );
  return $items;
}

/**
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
```
