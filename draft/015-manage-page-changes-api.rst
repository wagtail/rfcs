=============================================
RFC 15: Manage page changes via the Admin API
=============================================

:RFC: 15
:Author: Marco Fucci, Ravi Kotecha, Tyom Semonov
:Status: Draft
:Created: 2017-02-22
:Last-Modified: 2017-02-22

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

This RFC proposes an extension to the Admin API to allow a page to be published and its revisions to be retrieved, created and managed.

It's based on the *RFC 11 Revisions Admin API* by Karl Hobley but inspired by the `Google Drive API <https://developers.google.com/drive/v3/reference/revisions>`_.


Specification
=============

The endpoints are grouped as some relate to revisions and some to pages.


Revisions
:::::::::

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
:::::


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
