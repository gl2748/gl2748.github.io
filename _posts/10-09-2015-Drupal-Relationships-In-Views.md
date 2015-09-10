---
layout: post
category : drupal
tagline: "Exposing multi-stage relationships to Views"
tags : [ drupal, views, relationships]
---

This is a short guide to handling a scenario in Drupal where many entities are related to one another indirectly via intermediary entities.

For example imagine the following:

You have entities of type:node with a bundle: Samples and a bundle:Companies. You also have a User entity with role: Company Users.

Samples have an entity reference field (field_company_id) listing the Company who owns the sample. 
Companies have an entity reference field (field_associated_user_id) listing the Company Users who can log into the site and manage the Samples associated with the Company that they work for. However the Samples are not directly related to the Company Users.

![Drupal entity relationships (via entity reference fields) diagram]({{ site.url }}/assets/Drupal_relationships_2.png)

We want a view that shows the currently logged in user a list of Samples belonging to the company that they(the logged in user) are associated with. To do this we use Relationships in views, and crucially we chain multiple relationships together to access entities referenced at more than one step removed. (technically we can go infinitely deep with this approach.)

1. Create a view that only shows content type Samples:


2. Go to Advanced and select Add Relationship.
  * At this point your ‘Samples’ Node type should have an entity reference field relating it to a Company entity. Let’s say it’s called: field_company_id.
  * In the Add Relationship options window add the following relationship:
  * Entity Reference: Referenced Entity
A bridge to the Content entity that is referenced via field_company_id
  * Give this the default identifier of:
  Content entity referenced from field_company_id

3. Now if we select Add Relationship again we see entity reference fields from the ‘Companies’ node type are available. Specifically:
  * Entity Reference: Referenced Entity
  A bridge to the User entity that is referenced via field_associated_user_id
  * Give this the default identifier of:
  User entity referenced from field_associated_user_id

4. Now because we want to use these relationships as contextual filters for our views we need to add them as fields to the view. Importantly by adding these relationships fields from related content are available to display on our view. 
  * For example we can have a column that lists associated users for each sample.
Add Field: 
Content: Associated user Appears in: node:client
Select Relationship:
Content entity referenced from field_sample_client_id
  * More importantly we can add UIDs of associated users for each sample.
Add Field:
User:Uid
Select Relationship:
User entity referenced from field_associated_user_id

5. With the UIDs field in the view we can now add a contextual filter that gets its default value from the UID of the currently logged in user.

