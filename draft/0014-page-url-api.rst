=============================================================================
RFC 14: Update Page URL API to facilitate caching and simplify business logic
=============================================================================

:RFC: 14
:Author: Tobias McNulty
:Status: Draft
:Created: 2017-02-10
:Last-Modified: 2017-02-10

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

Wagtail currently (as of 1.9) generates many calls to ``Site.get_site_root_paths``
for each page load. The Wagtail Demo site, for example, generates 40 calls to this
method when loading the home page. This results in 40 calls to Memcached, if installed,
or 40 identical SQL queries per page load. When running everything on a single server
this might only add up to a few milliseconds of wasted time, but when services are
split across networks (e.g., different AWS availability zones), these unnecessary
calls can add up. This issue was originally reported in August, 2016 as
`#2916 <https://github.com/wagtail/wagtail/issues/2916>`_.

There is an issue with the current low-level API for generating page URLs (i.e.,
the ``Page.url`` and ``Page.full_url`` properties and the ``Page.relative_url`` method)
that makes it near impossible to address this issue without an API change. To make
some of the determinations required in these methods, and to avoid re-querying
the database or cache each time one of these methods is called, they need
access to the ``request`` object (if any). This not only allows caching
``Site.get_site_root_paths`` in the request scope, but also facilitates simpler
business logic around determining whether to show a local or fully-qualified URL
(what the ``url`` property does now if only one ``Site`` exists and what the
``relative_url`` property does by inspecting the provided ``Site`` object).

An accompanying draft implementation of this RFC can be found in
`PR #3354 <https://github.com/wagtail/wagtail/pull/3354>`_.

Specification
=============

This proposal involves adding two new methods to the ``Page`` class. The goal
would be for these to become the primary (low-level) API for generating page
URLs (``{% pageurl %}`` should still be used in templates, and can easily be
modified to use ``get_url``).

New URL methods
---------------

The two new methods needed in the ``Page`` class are ``get_url`` and
``get_full_url``, which accept an optional but recommended ``request`` argument.

For example,

.. code-block:: python

    page_url = my_page.url

would become:

.. code-block:: python

    page_url = my_page.get_url(request=request)

These methods should cache the output of ``Site.get_site_root_paths`` on the
request, if available, or on the page instance itself if not.

Updating ``relative_url``
-------------------------

``Page.relative_url`` is already a method rather than a property that accepts
a single argument, the current site. This method will need to be updated to
accept and pass through the request object. This introduces some duplication
insofar as it should be possible to determine the current site from the request
object as well, however, ``relative_url`` should continue to accept the current
site as its first argument and use that to decide whether to return a local or
fully-qualified URL.

Providing custom URLs
---------------------

The current recommended approach for customizing URL patterns by page is via
``Page.get_url_parts()``. ``get_url_parts()`` will also need to be modified to
accept a request, and any project-level Page subclasses that override
``get_url_parts()`` will need to be updated to pass through the request, like so:

.. code-block:: python

    def get_url_parts(self):
    	result = super(MyPage, self).get_url_parts()

would become:

.. code-block:: python

    def get_url_parts(self, *args, **kwargs):
    	result = super(MyPage, self).get_url_parts(*args, **kwargs)

Updating template tags that generate URLs
-----------------------------------------

The ``{% pageurl %}`` template tag will need to be updated to pass the request
from the template context into ``get_url()``. Additionally, custom template
tags that utilize URL properties should be updated to use ``get_url()`` or
``get_full_url()`` instead.

An app that needs to support both ``get_url()`` and ``url`` can easily fall
back to the previous API, if needed:

.. code-block:: python

    # use get_url to enable caching of site_root_paths, if possible
    try:
        page_url = page.get_url(request=context.get('request'))
    except AttributeError:
        # We're still using Wagtail 1.9 or older
        page_url = page.url

Deprecating the existing API
----------------------------

Optionally, the existing ``url`` and ``full_url`` properties could be deprecated.

The argument for deprecating ``url`` and ``full_url`` is that, while convenient,
they (a) have no way in their current implementation to determine the current
site (unless only one site exists in the database, which requires a SQL or
cache query to determine) and (b) have access to no request-level objects (other
than the page itself, which is not sufficient as a given request may access many
pages, e.g., for generating navigation menus) that would facilitate caching
``Site.get_site_root_paths``.

The ``relative_url`` method is simpler to modify given it's a method rather than
a property, however, it becomes somewhat obsolete if ``get_url`` has access to a
request. With the request, we should be able to figure out exactly what site
the user is on (if called during an HTTP request, of course), otherwise, a
fully-qualified URL will be returned. Since ``get_url`` and ``relative_url`` would
do effectively the same thing at that point, ``relative_url`` could optionally
be deprecated as well.

Open Questions
==============

* Determine whether to deprecate ``Page.url`` and ``Page.full_url`` or defer
  that decision till a later date. While convenient, using them circumvents the
  possibility of caching ``site_root_paths`` and may be worth disincentivizing.
