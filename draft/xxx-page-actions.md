# RFC X: Admin API - Page actions

* RFC: X
* Author: Karl Hobley
* Status: Draft
* Created: 2017-03-09
* Last Modified: 2017-03-09

## Abstract

In order to support external editing software and to improve the experience of
the existing Wagtail admin interface, we need to implement a set of actions
that can be performed on pages in the admin API.

This RFC aims to define the set of actions we need to implement and propose how
each action should be performed using the API.

This builds on top of work done in RFC15 which adds a revisions API that allows
editing, publishing, submitting and reviewing page content. So none of these
will be repeated here.

At the time of writing RFC18 (Admin API meta fields) is in draft, this RFC
assumes that RFC is accepted.

## The actions

### Creation

TODO

### Setting the edit lock

To set the edit lock, send a ``PUT`` request to: ``/api/pages/<page-id>/edit-lock/``.

If the edit lock was created, this will return a ``201`` response. If the edit
lock already existed, this will return a ``409`` response.

### Release the edit lock

To release the edit lock, send a ``DELETE`` request to: ``/api/pages/<page-id>/edit-lock/``

If the edit lock was released, this will return a ``200`` response. If the edit
lock doesn't exist, this will return a ``404`` response.

### Altering metadata

To alter a page's metadata send either a ``PUT`` or a ``PATCH`` request to
``/api/pages/<page-id>/meta/``

Initially, this will only allow changing the ``first_published_at`` field.

### Unpublishing

To unpublish a page, send a ``POST`` request to ``/api/pages/<page-id>/unpublish/``

Unpublishing a page just sets it's ``live`` status flag to ``false`` and
``has_unpublished_changes`` to ``true``. No revision data should be changed by
this action.

### Deleting

To delete a page, send a ``POST`` request to ``/api/pages/<page-id>/delete/?recursive={true|false}``

This permanently deletes the page.
If ``recursive`` has not been set to ``true`` and the page does have children,
the request will fail with with ``400`` response.

### Moving

To move a page, send a ``POST`` request to ``/api/pages/<page-id>/move/?to=<other-page-id>&rel={first-child|last-child|previous-sibling|next-sibling}``

The ``to`` parameter specifies a page in the tree where the page will be moved to.
The ``rel`` parameter specifies what relation the new position has to the ``to`` page.

Possible ``rel`` values:

 - ``first-child``
 - ``last-child``
 - ``previous-sibling``
 - ``next-sibling``

### Copying

TODO
