============
RFC X: Menus
============

:RFC: X
:Author: Karl Hobley
:Status: Draft
:Created: 2016-02-10
:Last-Modified: 2016-02-10

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

Many Wagtail sites require the menus to be editable from the admin interface.
For the main menu, this is usually done by setting the "show in menus" box on
each page, then ordering them in the listing view.

Some sites also have secondary menus, such as footer links. These can't
currently be edited using an out-of-the-box interface like the main menu can.
The most common way to implement editable secondary menus at the moment is to
use snippets.

Snippets are not ideal for this as they require a developer to rewrite the
models for every new site and the admin UI isn't very well suited for editing
multiple at once.

This RFC proposes a new "menus" contrib module that'll provide the models and
user interface for building menus using the admin interface. It will also
provide template tags to help render the menus on the site frontend.

Specification
=============

Defining and registering menus
------------------------------

Each menu has its own model that inherits from the
``wagtail.contrib.menus.models.Menu`` class. Each individual instance of this
model represents an item in that menu.

Menu models are registered using a ``register_menu`` decorator/function:

.. code-block:: python

    # mysite/menus/models.py
    from wagtail.contrib.menus.models import Menu, register_menu

    @register_menu
    class MainMenu(Menu):
        pass

    @register_menu
    class FooterMenu(Menu):
        pass

Limiting the depth of a menu
----------------------------

Sometimes, menus should only be one or two levels deep. This constraint can be
added to the menu model by using the ``max_depth`` attribute:

.. code-block:: python

    @register_menu
    class SecondaryMenu(Menu):
        # This menu should be flat
        max_depth = 1

Internals
---------

The builtin ``Menu`` model is abstract and contains the following:

 - A title field
 - Internal and external link fields
 - Treebeard's materialised path tree.

Rationale for design decisions
==============================

1. Why does each menu require it's own model?

   The main reason for this decision is because each menu needs its own
   ``treebeard`` tree. Using different models provides this separation.

Open Questions
==============
