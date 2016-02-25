=================================
RFC 5: Wagtail API fields
=================================

:RFC: 5
:Author: Karl Hobley
:Status: Draft
:Created: 2015-02-25
:Last-Modified: 2015-02-25

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

This RFC describes some syntax improvements we could add to the ``?fields=`` parameter
on all Wagtail API endpoints.

Specification
=============

We will use the images API endpoint as an example, this change would apply to all other
endpoints as well.

I've also cut down the examples to a single object for brevity.

The default response
--------------------

The default response returns the ``id``, ``type``, ``detail_url`` and ``title`` fields:

.. code-block::

    GET /api/images/

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image",
            "detail_url": "http://localhost:8000/api/v2beta/images/4/"
        },
        "title": "Wagtail by Mark Harkin"
    }

Specifying fields manually
--------------------------

As before, you can specify a plain list of fields. Doing this will override *all* of
the default fields (previously, you could only override the ``title`` field):

.. code-block::

    GET /api/images/?fields=id,tags

    {
        "id": 4,
        "tags": []
    }

Adding fields to the response
-----------------------------

Having to remember each field could be tedious and I expect those default fields would
be useful on most occasions. Prepending a ``+`` allows additional fields to be specified
without removing the defaults:

.. code-block::

    GET /api/images/?fields=+tags

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image",
            "detail_url": "http://localhost:8000/api/v2beta/images/4/",
            "tags": []
        },
        "title": "Wagtail by Mark Harkin"
    }

Removing fields from the response
---------------------------------

Prepending a ``-`` will remove a field from the response:

.. code-block::

    GET /api/images/?fields=-detail_url

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image"
        },
        "title": "Wagtail by Mark Harkin"
    }

All fields shortcut
-------------------

Setting fields to ``*`` will return all fields:

.. code-block::

    /?fields=*

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image",
            "detail_url": "http://localhost:8000/api/v2beta/images/4/",
            "tags": []
        },
        "title": "Wagtail by Mark Harkin",
        "width": 100,
        "height": 100
    }

This can be combined with the ``-`` operator to allow all except a particular field
to be returned:

.. code-block::

    /?fields=*,-tags

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image",
            "detail_url": "http://localhost:8000/api/v2beta/images/4/"
        },
        "title": "Wagtail by Mark Harkin",
        "width": 100,
        "height": 100
    }


Nested objects
--------------

The API has support for nesting objects, but doesn't yet allow specifying the fields
for those nested objects.

Using brackets, the syntax described above can be used to specify the fields for these:


.. code-block::

    /?fields=+myforeignkey

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image",
            "detail_url": "http://localhost:8000/api/v2beta/images/4/"
        },
        "title": "Wagtail by Mark Harkin",
        "myforeignkey": {
            "id": 10,
            "meta": {
                "type": "core.MyModel"
            }
        }
    }

    /?fields=+myforeignkey(id,name)

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image",
            "detail_url": "http://localhost:8000/api/v2beta/images/4/"
        },
        "title": "Wagtail by Mark Harkin",
        "myforeignkey": {
            "id": 10,
            "name": "Foo"
        }
    }

    /?fields=+myforeignkey(+name)

    {
        "id": 4,
        "meta": {
            "type": "wagtailimages.Image",
            "detail_url": "http://localhost:8000/api/v2beta/images/4/"
        },
        "title": "Wagtail by Mark Harkin",
        "myforeignkey": {
            "id": 10,
            "meta": {
                "type": "core.MyModel"
            },
            "name": "Foo"
        }
    }
