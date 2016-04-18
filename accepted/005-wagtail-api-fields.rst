=================================
RFC 5: Wagtail API fields
=================================

:RFC: 5
:Author: Karl Hobley
:Status: Accepted
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

Adding fields to the response
-----------------------

As before, you can specify a plain list of extra fields which will be added to
the response:

.. code-block::

    GET /api/images/?fields=tags

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

Prepending a ``-`` will remove a field from the response, this can be used on
any default field:

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

Setting fields to ``*`` will add all available fields to the response:

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

No fields shortcut
------------------

Like the "all fields shortcut" the ``_`` character can be used to remove all fields.
This would typically be used by developers who know they only need specific fields:

.. code-block::

    /?fields=_,title

    {
        "title": "Wagtail by Mark Harkin"
    }

Nested objects
--------------

The API has support for nesting objects, but doesn't yet allow specifying the fields
for those nested objects.

Using brackets, the syntax described above can be used to specify the fields for these:


.. code-block::

    /?fields=myforeignkey

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

    /?fields=myforeignkey(name)

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

    /?fields=myforeignkey(name,-type)

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
