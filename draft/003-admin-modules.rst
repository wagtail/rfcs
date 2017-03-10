=================================
WEP 3 - Admin Modules
=================================

:WEP: 3
:Author: Karl Hobley
:Status: Draft
:Created: 2015-12-15
:Last-Modified: 2015-12-15

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

Currently, each app with an interface in Wagtail defines a similar set of views that are linked with the rest of Wagtail admin interface using a few "hooks".

In order to customise one of these apps (either in a project or third-party app), you need to copy all of its views into a new location and override each hook one-by-one with your own. Not only is this tedious work but it is also very hard to maintain in the long-run.

There's also a big opportunity here to reduce the amount of duplicated code between apps.

This WEP proposes a low-level and stable interface for defining then registering "modules" into the admin interface (also allowing replacing existing modules).

Specification
=============

Modules
--------

A module is a single unit that encapsulates an entire section of the interface (eg, pages, images, docs, etc). A module is a Python class defined in the respective app that inherits from a base ``Module`` class defined in ``wagtailadmin``.

The module contains a URL config, a number of menu items and hooks for choosers and homepage panels. They also manage the permissions for their views as well.

Also:

- Module instances are both linked in and removed using "hooks".
- Module classes can be subclassed allowing existing apps to be modified in a maintainable way.
- A module class can have multiple instances registered at once in the same project (for example, this would allow multiple image models to be used at the same time).

This isn't a high-level API for building new parts of the admin interface from custom models (but it will make that much easier), it's more a reorganisation. The upcoming ``wagtailmodeladmin`` module will cover this use-case much better.

The module API
--------------

TODO

An example
----------

TODO

Open Questions
==============

TODO
