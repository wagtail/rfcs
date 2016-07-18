===========================
RFC 11: Revisions Admin API
===========================

:RFC: 11
:Author: Karl Hobley
:Status: Draft
:Created: 2016-07-18
:Last-Modified: 2016-07-18

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

Currently, the Admin API can only access the live version of a page's content. In order to support page editing, we must allow the admin API to access the latest draft and also create new revisions and publish them

I also think that the revisions UI currently in Wagtail would be better implemented using the admin API as this would allow us to implement it in a way that doesn't require the user to navigate away from the editor.

This RFC proposes an extension to the Admin API to allow page revisions to be retrieved and created.

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

We will add a new endpoint underneath the pages endpoint allowing access to revisions of specific pages.

View all revisions of a page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This will be paginated to show up to 20 revisions at a time.

.. code-block::

    GET /api/pages/1/revisions/

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

Filter by date/time range
`````````````````````````

Date or date/time in ISO8601 format. If a date is specified, both ends of the range are inclusive (equivilent to django's __gte and __lte queries)

The start and end times are separated by three dots: '...'. One end of the range may be omitted making that end unbounded.

.. code-block::

    GET /api/pages/1/revisions/?created_at=2016-07-18...
    GET /api/pages/1/revisions/?created_at=...2016-07-18
    GET /api/pages/1/revisions/?created_at=2016-01-01...2016-07-18
    GET /api/pages/1/revisions/?created_at=2016-07-18T16:37:00+01:00...

Filter by author
````````````````

Filters by the value of the ``USERNAME_FIELD`` on the user model

.. code-block::

    GET /api/pages/1/revisions/?author=karl

View a particular revision
^^^^^^^^^^^^^^^^^^^^^^^^^^

By ID
`````

.. code-block::

    GET /api/pages/1/revisions/123/

By earliest/latest created at date
``````````````````````````````````

These redirect to the earliest/latest revision of the page. The redirect to earliest is permanent but the redirect to latest is temporary as this could change at any time.

.. code-block::

    GET /api/pages/1/revisions/latest/
    GET /api/pages/1/revisions/earliest/


Create a new revision (edit a page)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Creating a new revision is done by submitting the value of the "content" field as a JSON dictionary to the following URL

.. code-block::

    POST /api/pages/1/revisions/new/

Submit for moderation
`````````````````````

Saves and submits the new revision for moderation

.. code-block::

    POST /api/pages/1/revisions/new/?submit_for_moderation=true

Publish
```````

Saves and publishes the new revision

.. code-block::

    POST /api/pages/1/revisions/new/?publish=true

Safeguarding against double-edit
````````````````````````````````

Double editing can be implemented by sending the previous revision id to the server on save. If there is a more recent revision that has already been committed to the server, the revision being submitted would be rejected.

.. code-block::

    POST /api/pages/1/revisions/new/?current_revision_id=1

Note that this doesn't mean that the user is told they must refresh the page and lose their changes. It may even be possible to resolve conflicts manually by retriving the latest revision and merging the two. This process is out of scope for this RFC though.
