========================
RFC 9: Page translations
========================

:RFC: 9
:Author: Tim Heap
:Status: Draft
:Created: 2016-05-05
:Last-Modified: 2016-05-05

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

Translations of pages in Wagtail currently has no officially supported method.
Many people have implemented translations in many different methods,
some more or less elegant than others.
All of the current methods suffer from not being integrated with Wagtail itself.
This proposal is for adding official, first-class support for translations in Wagtail.

Background
==========

People around the world speak an amazing array of languages.
Wagtail, if it wants to stay relevant in a multilingual world
needs to support multilingual sites.

There are a number of approaches to translations for Wagtail pages
out in the wild currently.
None of these approaches are optimal,
and none are officially supported or recommended by Wagtail.
An official, supported method that has support in Wagtail core is required.

Requirements
============

* Backwards compatible with current Wagtail sites,
  although these sites and pages will not translatable.

* It must be possible to translate content in to multiple languages.

* Nicely behaved in the database

  * No extraneous model instances

  * Translations have explicit relations to each other at the database level

  * Able to use all features of the database (validation, data types, etc) and migrations - i.e. no JSON blobs for translated content

* Nice to use as a developer / template author

  * Fairly seamless, in terms of defining models and edit handlers

  * A clear distinction between translated and non-translated fields

  * Easy to find all translations of a page

* Nice to use as an editor

  * The behaviour of editing pages not significantly changed

  * Adding a translation is easy

  * Different translations can be reviewed, moderated, approved, and published separately

  * Adding a language can be done through the admin

What is not required
--------------------

* Translating non-page models

Current approaches
==================

One method involves manually duplicating the Page tree, having one tree per language.
Keeping the trees in sync is difficult,
and means that all pages need to be translated to properly keep the trees in sync.

One method involves adding fields for each supported language to the model,
such as ``body_en`` and ``body_fr``.
This makes for a very verbose model declaration,
a very large edit page, and requires each page to be completely translated
to pass validation.

Praekelt have developed a method involving one page per translation,
but in a way that does not require duplicate trees.
Instead, the translated page is inserted as a sibling of the base page,
and then hidden from view.
This has the downside of needing to translate every field in a model,
even for fields such as background images that should be consistent across translations.

Proposal
========

Models
------

A few model classes will be provided.

A ``Language`` model will be created as part of the translations app.
Each language a site supports will have a ``Language`` entry in the database.
Administrators can add and remove these through the settings menu.
One language will be set as the default language.

Each page that is translatable must subclass ``TranslatedPage``.
Each ``TranslatablePage`` subclass must have a ``TranslatedFields`` class as an attribute.

The fields for a ``TranslatedPage`` that need translating
must be added to the ``TranslatedFields`` class:

.. code-block:: python

    from modelcluster.fields import ParentalKey
    from wagtail.wagtailadmin.edit_handlers import FieldPanel
    from wagtail.wagtailcore.fields import RichTextField
    from wagtail.wagtailimages.edit_handlers import ImageChooserPanel
    from wagtail.wagtailimages.fields import ImageField

    class ContentPage(TranslatedPage):
        feature_image = ImageField()

        class TranslatedFields:
            page_title = models.CharField(max_length=255)
            body = RichTextField()

            panels = [
                FieldPanel('title'),
                FieldPanel('body'),
            ]

        content_panels = [
            ImageChooserPanel('feature_image'),
        ]

When fetched from the database, a ``TranslatedPage``
will automatically fetch the translation for the currently active language,
or for the default language if there is no translation for the current language.

Proxy attributes for each of the translated fields will be added to the ``Page`` instance
which reference the translation in the current language.

Admin UI
--------

Creating/editing a page behaves exactly like the current page editor.
All non-translated fields are editable in the normal manner.
Translated fields are not editable from the page editing interface.

Once a page has been created (as a draft, or published),
editors will be able to add translations for the page.
A list of supported languages, the status of their translations,
and a link to edit these translations will be present on the page listing,
and on the page edit screens.
Clicking one of the languages will take the editor
to a page where they can fill out the translated fields for that language.


TODO: Is there a nice way of allowing editors to set the translated fields while creating a page?
The two-step process is not great from a usability point of view.
The problems with this is how to integrate the two sets of edit handlers -
for the ``Page``, and for the ``TranslatedFields``.
Editing the translated fields from the ``Page`` edit view
once the ``Page`` has been created should not be possible.
Translations exist in their own moderation cycle, separate from pages.

Frontend
--------

Template authors will not have to change how they make templates.
Translated fields will be accessible right next to normal fields on a ``Page`` instance.

A template tag that prints a form, allowing visitors to switch language will be provided.
The template for this form will be overridable in the normal Django manner.

.. code-block:: html+django

    <html>
        <head>
            <title>{{ page.page_title }}</title>
        </head>
        <body>
            <h1>{{ page.page_title }}</h1>
            {{ page.body|richtext }}

            {% load wagtailtranslations_tags %}
            {% language_picker %}
        </body>
    </html>

A route must be added to a sites ``URLConf`` for the language picker to work:

.. code-block:: python

    url_patterns += [
        url('_set-language$', include('wagtail.wagtailtranslations.urls')),
    ]

Site visitors
-------------

On the first visit to a site,
a new visitor will be presented content in the default language.
Visitors will be able to select their preferred language
from the set of available languages defined for that site.
This selection will affect the whole site,
and be stores in the visitors session for later visits.

Django's existing support for switching languages on the front end will be used.
This will automatically support using standard .po/.mo files
to translate non-dynamic sections of the site,
such as section titles and footer content.
