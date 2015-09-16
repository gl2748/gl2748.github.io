---
layout: post
category : drupal
tagline: "Giving users permissions based on their role"
tags : [drupal, php, publishcontent, API]
---

The [PublishContent Module](http://cgit.drupalcode.org/publishcontent/tree/publishcontent.api.php) provides an API hook for publish content permissions. This is a powerful feature when we want to expand publish permissions beyond defaults.

In this example we will grant a logged-in-user publish/unpublish controls over content that they are indirectly associated with. By default publish/unpublish controls attach to a node author, and there can only ever be one node author attached directly to a particular etity. Therefore you can see how this approach is useful whenever we want to grant author-like power to more than one user at more than one stage removedd.

In our example we have three entities, samples, clients and users. The samples are related to clients via an entity reference field on the sample entity and the clients are related to users via an entity reference field on the client entity. We want to give permissions to these users to publish/unpublish the content associated with the client that they themselves are associated with.

```php
/**
 * @file
 * Code for the psilabs_str feature.
 *
 * This custom module checks the currently logged in user against
 * users associated(via clients) with samples on the Psilabs site.
 * If a logged in user is associated with a sample they are given
 * Publish/Unpublish controls over those associated nodes.
 */

include_once 'psilabs_str.features.inc';

/**
 * Implements _query_user_role().
 *
 * Check to see if the current user has been assigned
 * the client_user role, and get the uid
 * of the current user.
 *
 * @param user $user
 *   The user object for the user you're checking; defaults to the current user.
 *
 * @return bool
 *   TRUE if the user object has the role, FALSE if it does not.
 */
function _query_user_role($user) {
  if (in_array("client_user", $user->roles)) {
    return TRUE;
  }
  return FALSE;
}

/**
 * Implements _get_referenced_users().
 *
 * Get an array of user IDs associated with entity of type:node bundle:sample.
 *
 * @param node $node
 *   The nodes object being checked.
 *
 * @return array
 *   An array of user ID's associated with the nodes being checked.
 */
function _get_referenced_users($node) {
  // Get the client referenced by the sample.
  $node_wrapper = entity_metadata_wrapper('node', $node);
  $referenced_client = $node_wrapper->field_sample_client_id->value();
  // Declare an array that we will use to hold the results of the foreach loop.
  $referenced_user_ids = array();
  // Get the user(s) referenced by the referenced_client
  // Https://www.drupal.org/documentation/entity-metadata-wrappers
  // Store each item from foreach loop in array
  // http://stackoverflow.com/questions/3045619/need-to-store-values-from-foreach-loop-into-array
  $client_wrapper = entity_metadata_wrapper('node', $referenced_client);
  foreach ($client_wrapper->field_associated_user_id->getIterator() as $delta => $referenced_user_wrapper) {
    // $user_wrapper may now be accessed as a user term wrapper.
    $referenced_user_ids[] = $referenced_user_wrapper->value()->uid;
  }
  return $referenced_user_ids;
}

/**
 * Modify access to the publish controls.
 *
 * Implements hook_publishcontent_publish_access,
 * if current user is referenced on a sample give them
 * the ability to publish that sample.
 *
 * @param node $node
 *   A node object being checked.
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
    // Get the currently logged-in user's id.
    $current_user_id = $user->uid;
    // Get the array of users referenced on the node.
    $referenced_user_ids = _get_referenced_users($node);
    // If current_user_id is referenced on a node.
    // Give them unpublish access to that node.
    if (in_array($current_user_id, $referenced_user_ids)) {
      return PUBLISHCONTENT_ACCESS_ALLOW;
    }
  }
  return PUBLISHCONTENT_ACCESS_IGNORE;
}

/**
 * Modify access to the publish controls.
 *
 * Implements hook publishcontent_publish_access,
 * if current user is referenced on a sample give them
 * the ability to publish that sample.
 *
 * @param node $node
 *   A node object being checked.
 * @param user $user
 *   The user wanting to unpublish the node.
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
function psilabs_str_publishcontent_unpublish_access($node, $user) {
  if (_query_user_role($user)) {
    // Get the currently logged-in user's id.
    $current_user_id = $user->uid;
    // Get the array of users referenced on the node.
    $referenced_user_ids = _get_referenced_users($node);
    // If current_user_id is referenced on a node.
    // Give them unpublish access to that node;
    if (in_array($current_user_id, $referenced_user_ids)) {
      return PUBLISHCONTENT_ACCESS_ALLOW;
    }
  }
  return PUBLISHCONTENT_ACCESS_IGNORE;
}
```
