========================
RFC 9: Page translations
========================

:RFC: 9
:Author: Tim Heap
:Status: Draft
:Created: 2016-05-05
:Last-Modified: 2016-08-10

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

  * Easy to find all translations of a page

* Nice to use as an editor

  * The behaviour of editing pages not significantly changed

  * Adding a translation is easy

  * Page slugs can be translated

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
One language will be set as the default language:

.. code-block:: python

    from django.db import models

    class Language(models.Model):
        code = models.CharField(
            max_length=12,
            help_text="One of the languages defined in LANGUAGES")

        is_default = models.BooleanField(
            default=False, help_text="""
            Visitors with no language preference will see the site in this
            language
            """)

        order = models.IntegerField(
            help_text="""
            Language choices and translations will be displayed in this order
            """)

        live = models.BooleanField(
            default=True,
            help_text="Is this language available for visitors to view?")

        class Meta:
            ordering = ['order']

Each page that is translatable must subclass ``TranslatedPage``:

.. code-block:: python

    from django.db import models
    from wagtail.wagtailcore.models import Page
    from django.utils.translation import activate

    class TranslatedPage(Page):
        # Explicitly defined with a unique name so that clashes are unlikely
        translated_page_ptr = models.OneToOneField(
            parent_link=True, related_name='+', on_delete=models.CASCASE)

        # Pages with identical translation_keys are translations of each other
        # Users can change this through the admin UI, although the raw UUID
        # value should never be shown.
        translation_key = models.UUIDField(db_index=True, default=uuid.uuid4)

        # Deleting a language that still has pages is not allowed, as it would
        # either lead to tree corruption, or to pages with a null language.
        language = models.ForeignKey(Language, on_delete=models.PREVENT)

        class Meta:
            # This class is *not* abstract, so that the unique_together
            # constraint holds across all page classes. Translations of a page
            # do not have to be of the same page type.

            unique_together = [
                # Only one language allowed per translation group
                ('translation_key', 'language'),
            ]

        def serve(self, request, *args, **kwargs):
            activate(self.language.code)
            super().serve(request, *args, **kwargs)

Pages should then subclass ``TranslatedPage``:

.. code-block:: python

    from wagtail.wagtailadmin.edit_handlers import FieldPanel
    from wagtail.wagtailcore.fields import RichTextField
    from wagtail.wagtailcore.models import Page
    from wagtail.wagtailimages.edit_handlers import ImageChooserPanel
    from wagtail.wagtailimages.fields import ImageField

    class ContentPage(TranslatedPage, Page):
        body = RichTextField()
        feature_image = ImageField()

        content_panels = [
            FieldPanel('title'),
            FieldPanel('body'),
            ImageChooserPanel('feature_image'),
        ]

An ``AbstractTranslationIndexPage`` class will be provided.
This page class can serve as a home page for a site.
Site administrators can create one page tree for each language
as children of this index.
When the page is accessed directly,
visitors will be redirected to the most relevant language for them.

.. code-block:: python

    def get_user_languages(request):
        """
        Get the best matching Languages for a request, in order from best to worst.
        The default language (if there is one) will always appear in this list.
        """
        pass  # Implementation left as an exercise to the reader

    class AbstractTranslationIndexPage(Page):

        def serve(self, request):
            languages = get_user_languages(request)
            candidate_pages = TranslatedPage.objects\
                .live().specific()\
                .child_of(self)

            for language in languages:
                try:
                    # Try and get a translation in the users language
                    translation = candidate_pages.filter(language=language).get()
                    # Redirect them to that page instead
                    return redirect(translation.url)
                except TranslatedPage.DoesNotExist:
                    continue

            # No translation was found, not even in the default language! Oh dear.
            raise Http404

        class Meta:
            abstract = True


Developers should subclass ``AbstractTranslationIndexPage`` in their own site,
and define ``subpage_types`` and similar things:

.. code-block:: python

    class TranslationHomePage(AbstractTranslationIndexPage):
        subpage_types = ['HomePage']

``TranslatedPage``\s directly under an ``AbstractTranslationIndexPage``
may have slugs that relate to their language, in order to present urls like
``/en/`` for the English home page, or ``/nl/nieuws/`` for the Dutch news page.
If pages are created in this manner, they will produce URLs much the same as ``i18n_patterns`` would.

Admin UI
--------

Creating/editing a page behaves exactly like the current page editor.
A new "Translations" tab in the editing interface will be added.
This tab will allow editors to select the language this page is written in.
Editors can also choose an existing page that this page is a translation of.
For example, with a "News" page translated in to three languages,
selecting any other "News" page will add this page as another translation in the "News" group.
If this page is the first of the 'translation group', a unique translation key
will be generated without the editor having to make any changes.

In the explorer, a new drop down button will be added for each translatable page
that lists all the translations of that page,
so that editors can quickly review and edit them all.

Frontend
--------

Template authors will not have to change how they make templates.

``TranslatedPage``\s will have a ``get_translations`` method
which returns a queryset of all the translations of that page:

.. code-block:: python

    def get_translations(self):
        return TranslatedPage.objects\
            .select_related('language')\
            .filter(translation_key=self.translation_key,
                    language__in=Language.objects.live())\
            .order_by(language__order)

Template authors can iterate over this queryset to build a list of links to translations:

.. code-block:: html

    <nav><ul>
        {% for translation in page.get_translations %}
            <a href="{% pageurl translation %}" class="{% if translation == page %}current{% endif %}">
                {{ translation.title }} ({{ translation.language }})
            </a>
        {% endfor %}
    </ul></nav>

As each ``TranslatedPage`` activates the relevant language locale when served,
static site content can be translated using the standard .po/.mo files
and ``{% trans %}`` template tags.

Site visitors
-------------

A ``AbstractTranslationIndexPage``\s default serve view
will examine the preferred language of the user and the set of available languages,
and redirect the user to the most relevant child page.
Browsing the site should otherwise operate as normal.

Issues
======

The above approach is simple, and effective, but it is not perfect.

It requires site administrators to effectively make one site tree per language,
keeping the site structure in sync manually.
If the site needs to be restructured, and pages moved around,
the page tree for each language will need to be updated manually.
The proposed implementation does mean that the site structure for each language does not have to be identical though,
which means each language could get a highly localised site structure.

Content that should be kept in sync across translations of a Page
will require extra work on behalf of the site administrators.
This content could be background images or similar language-independent content.
Again though, while this might be a hassle, it does allow for maximum flexibility.
