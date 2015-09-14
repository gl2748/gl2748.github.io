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

3. At this point we have our rule redirecting to /client-list/[account:uid]. However, the challenge of passing that UID to the links displayed in our view remains. To do this we add a contextual filter 'Global Null' and provide a default value from a path component of our URL.
![Drupal global null contextual filter]({{ site.url }}/assets/Drupal_6.png)
![Drupal global null default value path component]({{ site.url }}/assets/Drupal_7.png)

4. Adding this global null makes whatever value it pulls available as a token for our view. This is handy when we have a field or link that we want to re-write results for.
![Drupal global null default value path component]({{ site.url }}/assets/Drupal_8.png)

5. In our case we want to display an 'assign New User to this Client' link for each node displayed in our client-list view. Therefore we add a custom text field that pulls the token for the node:path, and appends a properly formatted string [pre-populate module](https://www.drupal.org/project/prepopulate) to prepopulate the relvant field on that node. However we will see that there are a few complicating factors Therefore our approach is to build tokens that can be combined to create a link that accomodates all of our variables.
  * Step 1: is to create the token for the path where the node lives. This is simply a matter of adding a field for content:path and exculding it from our display. Thereafter it is available as a token [path] to our view.
  * Step 2: is to create the token for the pre-populate string. This is also the field that pulls the Global Null that we just defined in the contextual filter Global:Null. It will be availble as token [nothing].
![drupal custom text field pre-populate formatter]({{ site.url }}/assets/Drupal_9.png)

  * Step 3: is to create a custom text field that pulls in the users that are already associated with the node, after all we don't want our new value over-writting the existing data for the field we are targetting. This field is also excluded from the display, it simply builds a token that we will use in the next step, this token is: [field_associated_user_id]. We can configure the multi-value results to conform with pre-populate module that we will need to patch to accomodate multi-values.
Finally note the ',' that precedes the token for the associated users field that is already available to views.
![drupal custom text field format extant field with comma]({{ site.url }}/assets/Drupal_11.png)
  * Step 3: is to combine these three tokens into a link that pre-populates the relevant field on our target nodes. Here are the tokens as they appear in the 'replacement patterns' dialogue:
![drupal available tokens]({{ site.url }}/assets/Drupal_11i.png)

and here is the complete re-direct link:

![drupal available tokens]({{ site.url }}/assets/Drupal_13.png)


