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

Many Wagtail sites require the menus to be editable from the admin interface. For the main menu, this is usually done by setting the "show in menus" box on each page, then ordering them in the listing view.

Some sites also have secondary menus, such as footer links. These can't currently be edited using an out-of-the-box interface like the main menu can. The most common way to implement editable secondary menus at the moment is to use snippets.

Snippets are not ideal for this as they require a developer to rewrite the models for every new site and the admin UI isn't very well suited for editing multiple at once.

This RFC proposes a new "menus" contrib module that'll provide the models and user interface for building menus using the admin interface. It will also provide template tags to help render the menus on the site frontend.

Specification
=============

Rationale for design decisions
==============================

Open Questions
==============

