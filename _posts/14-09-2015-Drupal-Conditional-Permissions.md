---
layout: post
category : drupal
tagline: "Giving user permissions based on their role"
tags : [ drupal, php, publishcontent, API]
---
The [PublishContent Module](http://cgit.drupalcode.org/publishcontent/tree/publishcontent.api.php) provides an API hook for publish content permissions. This is a powerful feature when we want to expand publish permissions beyond efaults.

In this example we will grant a logged-in-user publish/unpublish controls over content that they are indirectly associated with. By default publish/unpublish controls attach to a node author, and there can only ever be one node author attached directly to a particular etity. Therefore you can see how this approach is useful whenever we want to grant author-like power to more than one user at more than one stage removedd.

In our example each node has an entity reference field of users associated with that node. For example we could have nodes for different companies with a user reference field listing employees who can log into the drupal site and edit the node for their company.

In our example we have a slightly more convoluted senario. Here we have samples that are associated with a client and users associated with that client. We want to give permissions to these users to publish/unpublish the content associated with the client that they themselves are associated with.

```
<?php

/**
 * Psuedo-code sketching out what this does.
 * If the current user's role = 'user client' continue...
 * Get the user's UID.
 * Get the client entity referenced in field field_sample_client_id
 * Get the UID of the user entity referenced in field field_associated_user_id of the client entity
 * If UIDs match grant publish/unpublish permission.

 * @param node $node
 *   A node object being checked
 * @param user $user
 *   The user wanting to publish the node.
 *
 * @return bool|NULL
 *   PUBLISHCONTENT_ACCESS_ALLOW - if the account can publish the node
 *   PUBLISHCONTENT_ACCESS_DENY - if the user definetley can not publish
 *   PUBLISHCONTENT_ACCESS_IGNORE - This module wan't change the outcome.
 *   It is typically better to return IGNORE than DENY. If no module returns
 *   ALLOW then the account will be denied publish access. If one module
 *   returns DENY then the user will denied even if another module returns
 *   ALLOW.
 */

/**
 * Check to see if the current user has been assigned the client_user role, and get the uid
 * of the current user.
 *
 * @param $role
 *   The name of the role you're trying to find.
 * @param $user
 *   The user object for the user you're checking; defaults to the current user.
 * @return
 *   TRUE if the user object has the role, FALSE if it does not.
 */
function _query_user_role($user){
   // check if the current user has the 'client_user' role
  if (in_array("client_user", $user->roles)) {
    return TRUE;
   }
  return FALSE;
}

/**
 * Get an array of user IDs associated with entity of type:node bundle:sample
 * 
 * @param $node
 *   The nodes object being checked
 * @return
 *  An array of user ID's associated with the nodes being checked.
 */
function _get_referenced_users($node){
  //get the client referenced by the sample.
  $node_wrapper = entity_metadata_wrapper('node', $node);
  $referenced_client = $node_wrapper->field_sample_client_id->value();
  //Declare an array that we will use to hold the results of the foreach loop.
  $referenced_user_ids = array();
  //Get the user(s) referenced by the referenced_client https://www.drupal.org/documentation/entity-metadata-wrappers
  //Store each item from foreach loop in array http://stackoverflow.com/questions/3045619/need-to-store-values-from-foreach-loop-into-array
  $client_wrapper = entity_metadata_wrapper('node', $referenced_client);
    foreach ($client_wrapper->field_associated_user_id->getIterator() as $delta => $referenced_user_wrapper) {
      // $user_wrapper may now be accessed as a user term wrapper.
      $referenced_user_ids [] = $referenced_user_wrapper->value()->uid;
    }
  return $referenced_user_ids;
}

/** 
 * Modify access to the publish controls.
 *
 * Implements hook publishcontent_publish_access, if current user is referenced on a sample give them the ability to publish that sample.
 *
 * @param node $node
 *   A node object being checked
 * @param user $user
 *   The user wanting to publish the node.
 *
 * @return bool|NULL
 *   PUBLISHCONTENT_ACCESS_ALLOW - if the account can publish the node
 *   PUBLISHCONTENT_ACCESS_DENY - if the user definetley can not publish
 *   PUBLISHCONTENT_ACCESS_IGNORE - This module won't change the outcome.
 *   It is typically better to return IGNORE than DENY. If no module returns
 *   ALLOW then the account will be denied publish access. If one module
 *   returns DENY then the user will denied even if another module returns
 *   ALLOW.
 */
function psilabs_str_publishcontent_publish_access($node, $user) {
  if (_query_user_role($user)) {
    //get the currently logged-in user's id.
    $current_user_id = $user->uid;
    //get the array of users referenced on the node
    $referenced_user_ids = _get_referenced_users($node);
    //if current_user_id is referenced on a node, give them unpublish access to that node;
    if (in_array($current_user_id, $referenced_user_ids)) {
      return PUBLISHCONTENT_ACCESS_ALLOW;
    }
  }
  return PUBLISHCONTENT_ACCESS_IGNORE;
}

/**
 * Modify access to the publish controls.
 *
 * Implements hook publishcontent_unpublish_access, if current user is referenced on a sample give them the ability to publish that sample.
 *
 * @param node $node
 *   A node object being checked
 * @param user $account
 *   The user wanting to unpublish the node.
 * @return bool|NULL
 *   PUBLISHCONTENT_ACCESS_ALLOW - if the account can publish the node
 *   PUBLISHCONTENT_ACCESS_DENY - if the user definetley can not publish
 *   PUBLISHCONTENT_ACCESS_IGNORE - This module won't change the outcome.
 *   It is typically better to return IGNORE than DENY. If no module returns
 *   ALLOW then the account will be denied publish access. If one module
 *   returns DENY then the user will denied even if another module returns
 *   ALLOW.
 */
function psilabs_str_publishcontent_unpublish_access($node, $user) {
  if (_query_user_role($user)) {
    //get the currently logged-in user's id.
    $current_user_id = $user->uid;
    //get the array of users referenced on the node
    $referenced_user_ids = _get_referenced_users($node);
    //if current_user_id is referenced on a node, give them unpublish access to that node;
    if (in_array($current_user_id, $referenced_user_ids)) {
      return PUBLISHCONTENT_ACCESS_ALLOW;
    }
  }
  return PUBLISHCONTENT_ACCESS_IGNORE;
}
```
