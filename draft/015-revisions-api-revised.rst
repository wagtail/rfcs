=====================================
RFC 15: Revisions Admin API (Revised)
=====================================

:RFC: 15
:Author: Karl Hobley, Marco Fucci, Ravi Kotecha, Tyom Semonov
:Status: Draft
:Created: 2017-02-22
:Last-Modified: 2017-03-02

.. contents:: Table of Contents
   :depth: 3
   :local:


Abstract
========

This RFC 15 is an updated version of the `RFC 11 <#11>` by Karl Hobley.
It was created to allow more time for discussing the new Admin API and
has some parts copied from the original RFC 11.

Currently, the Admin API can only access the live version of a page's content.
In order to support page editing, we must allow the admin API to access the
latest draft and also create new revisions and publish them.

I also think that the revisions UI currently in Wagtail would be better
implemented using the admin API as this would allow us to implement it in a
way that doesn't require the user to navigate away from the editor.

This RFC proposes an extension to the Admin API to allow page revisions to be
retrieved and created.

Some of the changes are inspired by the 
`Google Drive API <https://developers.google.com/drive/v3/reference/revisions>`_.


Specification
=============


Data format
-----------

All of the responses return revisions in the following format


.. code-block:: json

    {
        "id": 1,
        "created_at": "2016-07-18T16:37:00+01:00",
        "author": "karl",
        "in_moderation": false,
        "content": {
            "title": "Home",
            "body": "foo"
        }
   }


URL structure
-------------

We will add a new endpoint underneath the pages endpoint allowing access to
revisions of specific pages.

The endpoints are grouped as some relate to revisions and some to pages.


Revisions
---------

View all revisions
^^^^^^^^^^^^^^^^^^

This will be paginated to show up to 20 revisions at a time.

.. code-block:: http

    GET /api/pages/revisions/

The list will be formatted the same way as other listings in the API:

.. code-block:: json

    {
        "meta": {
            "total_count": 50,
        },
        "items": [
            # Revisions here
        ]
    }


Filter by author
````````````````

Filters by the value of the ``USERNAME_FIELD`` on the user model

.. code-block:: http

    GET /api/pages/revisions/?author=<author-username>


View all revisions of a page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This will be paginated to show up to 20 revisions at a time.

.. code-block:: http

    GET /api/pages/<page-id>/revisions/

The list will be formatted the same way as other listings in the API:

.. code-block:: json

    {
        "meta": {
            "total_count": 50,
        },
        "items": [
            # Revisions here
        ]
    }

Filter by author
````````````````

Filters by the value of the ``USERNAME_FIELD`` on the user model

.. code-block:: http

    GET /api/pages/<page-id>/revisions/?author=<author-username>


Get a particular revision
^^^^^^^^^^^^^^^^^^^^^^^^^

By page and revision id
```````````````````````

.. code-block:: http

    GET /api/pages/<page-id>/revisions/<revision-id>/

This returns an error if `<revision-id>` does not refer to `<page-id>`.

By revision id only
```````````````````

You can use `'-'` instead of `<page-id>` if you only care about or know the revision id.

.. code-block:: http

    GET /api/pages/-/revisions/<revision-id>/


Get the latest revision of a page
`````````````````````````````````

To get the latest revision of a page, you use ``'head'`` as 
``revision-id``:

.. code-block:: http

    GET /api/pages/<page-id>/revisions/head/

This redirects to ``/api/pages/<page-id>/revisions/<revision-id>/``


Create a new revision
^^^^^^^^^^^^^^^^^^^^^

Creating a new revision is done by submitting the value of the "content" field
as a JSON dictionary to the following URLs


Create a new revision of an existing page
`````````````````````````````````````````

.. code-block:: http

    POST /api/pages/<page-id>/revisions/


Create the first revision of a new page
```````````````````````````````````````

.. code-block:: http

    POST /api/pages/-/revisions/


The return value will include the related `<page-id>` and `<revision-id>`.


Pages
-----

Moderation
^^^^^^^^^^

Submit for moderation
`````````````````````

.. code-block:: http

    POST /api/pages/<page-id>/moderation/submit/


This works the same as creating a new revision but it submits it 
for moderation as well instead of just saving it as a draft.

If previous revisions were submitted for moderation, this will
reset those to make sure that there's only one awaiting moderation.

Reject moderation
`````````````````

.. code-block:: http

    POST /api/pages/<page-id>/moderation/reject/

This will reject the revision awaiting moderation.


Publishing
^^^^^^^^^^

Publish a page
``````````````

.. code-block:: http

    POST /api/pages/<page-id>/publish/

This works the same as creating a new revision but it publishes it 
as well instead of just saving it as a draft.

Unpublish a page
````````````````


.. code-block:: http

    POST /api/pages/<page-id>/unpublish/

This will unpublish the currently published revision.

Safeguarding against double-edit
--------------------------------

We will ignore double editing to keep this RFC simple.


Locked pages
------------

A page cannot be edited in any way if it is locked, so attempting to create a
new revision for a locked page will result in a ``423 Locked`` response code
and the new revision will not be saved.
