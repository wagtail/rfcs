=====================================
RFC 15: Revisions Admin API (Revised)
=====================================

:RFC: 15
:Author: Karl Hobley, Marco Fucci, Ravi Kotecha, Tyom Semonov
:Status: Draft
:Created: 2017-02-22
:Last-Modified: 2017-03-01

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

View all revisions of a page
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Returns all the revisions of a page, 20 at a time.

.. code-block:: http

    GET /api/pages/<page-id>/revisions/


View a particular revision by ID
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: http

    GET /api/pages/<page-id>/revisions/<revision-id>/

``revision-id`` can be either the id of the revision or ``head`` to refer to the latest revision.


Create a new revision (edit a page)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: http

    POST /api/pages/<page-id>/revisions/

Submit for / reject moderation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To submit or reject a revision for moderation, make a ``PATCH`` call with ``submitted_for_moderation=true|false`` as data.

.. code-block:: http

    PATCH /api/pages/<page-id>/revisions/<revision-id>/

Pages
-----


Publish a page
^^^^^^^^^^^^^^

To publish a revision, call the ``publish`` ``POST`` action with the ``id`` of the revision you want to publish.

.. code-block:: http

    POST /api/pages/<page-id>/publish/?revision-id=<id>

Unpublish a page
^^^^^^^^^^^^^^^^

To unpublish a page, call the ``unpublish`` ``POST`` action. This will set ``live`` to ``False``.

.. code-block:: http

    POST /api/pages/<page-id>/unpublish/


Open Questions
==============
 
I suggest we keep this RFC simple and ignore double editing.