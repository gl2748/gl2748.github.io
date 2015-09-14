---
layout: post
category : drupal
tagline: "Passing URL Arguments as Tokens to your View"
tags : [ drupal, views, tokens, url arguments, contextual filter]
---

In Drupal URLs are a useful place to store arguments, furthermore as paths they often record the journey a user has taken to reach a certain page and hint at where they might be headed.

For this reason we often encounter situations where we need to pass arguments from a URL to something on our site that a user sees or interacts with. Furthermore we may want to preserve arguments as they pass from a rule to a view for example. 

For example let's say we have a view that displays a list of entities of bundle type: Node and that for each node we want to display a link to edit that node. Finally we want that link to pre-populate an entity-reference field on the node. 

As a concrete example, let's say after adding a new user to a site we want to be able to add that user to a node as an 'associated user' in an entity reference field.


![Drupal passing URL arguments through rules and views diagram]({{ site.url }}/assets/Drupal_3.png)


1. Create a rule that redirects to a url /client-list/uid 'After Saving a new User Account.'
  * Importantly configure the re-direct URL to include the [account:uid] token.
![Drupal rules page redirect on new user account creation diagram]({{ site.url }}/assets/Drupal_4.png)

2. Create a view that displays all of our nodes that we want to associate users with. In this example our view is configured to be at /client-list.
![Drupal basic views page configuration]({{ site.url }}/assets/Drupal_5.png)

3. At this point we have our rule redirecting to /client-list/[account:uid], of course no content exists at that address but when given a path to non-existent content drupal will fall back to the nearest existing content. The challenge of passing that UID to our view remains. To do this we add a contextual filter 'Global Null' and 

  

3. Now if we select Add Relationship again we see entity reference fields from the ‘Companies’ node type are available. Specifically:
  * Entity Reference: Referenced Entity
  A bridge to the User entity that is referenced via field_associated_user_id
  * Give this the default identifier of:
  User entity referenced from field_associated_user_id

4. Now because we want to use these relationships as contextual filters for our views we need to add them as fields to the view. Importantly by adding these relationships fields from related content are available to display on our view. 
  * For example we can have a column that lists associated users for each sample.
  * Add Field: 
Content: Associated user Appears in: node:client
Select Relationship:
Content entity referenced from field_sample_client_id
  * More importantly we can add UIDs of associated users for each sample.
      * Add Field:
User:Uid
Select Relationship:
User entity referenced from field_associated_user_id

5. With the UIDs field in the view we can now add a contextual filter that gets its default value from the UID of the currently logged in user.

