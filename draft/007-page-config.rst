=================
RFC 7: PageConfig
=================

:RFC: 7
:Author: Tim Heap
:Status: Draft
:Created: 2016-04-28
:Last-Modified: 2016-04-28

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

The configuration for ``Page`` subclasses should be separated out in to a new ``PageConfig`` class.
This will help promote separation of concerns,
and proper divisions between models, views and templates;
as well as dealing with a number of circular import issues
that can crop up in Wagtail models currently.

Rational
========

The configuration for ``Page`` subclasses currently lives on the ``Page`` model itself.
While this is great for keeping everything in one easy to find location,
it goes against the recommended work flow in Django
of separating models from views,
and views from templates.
This RFC proposes a new ``PageConfig`` class
to hold all the configuration for a ``Page`` subclass.
This will help promote separation of concerns,
and proper divisions between models, views and templates;
as well as dealing with a number of circular import issues
that can crop up in Wagtail models currently.
It follows how Django configures its ``ModelAdmin``,
with a separate class defined in ``admin.py``,
so Django developers are likely already familiar with this kind of thing.

By making this change, various tightly coupled parts of Wagtail can be loosened.
Currently, methods for making sitemaps and static site configurations exist
on the base ``Page`` model,
even though the sitemap and static site generation modules
are in the ``contrib`` namespace.
Part of this is for ease of use for inexperienced developers,
but also because multi-table inheritance makes mixing different classes together difficult.
As sitemaps and static site generation
do not require extra database fields
or special access to the model to function correctly,
these modules would work very well as ``PageConfig`` mixins,
such as ``SitemapPageConfig`` and ``StaticSitePageConfig``.
Developers can then use these if they wish by mixing in these classes.
``RoutablePageMixin`` is a good example of how mixins work with ``Page`` models already,
but is hampered by the limitations of multi-table inheritance.
This limitation would cease to exist
if all of this configuration existed on a ``PageConfig`` class.

This proposal will also make the ``models.py`` shorter.
For any large Wagtail site,
the ``models.py`` file grows to be enormous.
There is currently no easy way around this,
as all the model configuration, page options, custom forms, and view logic
must live in this one file.

Specification
=============

A ``PageConfig`` class is proposed.
This class will be similar in concept to the ``ModelAdmin`` class
from the Django admin app.
It will hold all the configuration and behaviour for the ``Page`` subclass.
Developers shold subclass ``PageConfig`` to define any custom behaviour

.. code-block:: python

    from wagtail.wagtailadmin.edit_handlers import FieldPanel
    from wagtail.wagtailcore.pages import PageConfig, register
    from wagtail.wagtailimages.edit_handlers import ImageChooserPanel

    from .forms import ContentPageAdminForm
    from .models import ContentPage
    from .views import contentpage_serve


    @register(ContentPage)
    class ContentPageConfig(PageConfig):
        parent_page_types = [ContentPage, 'home.HomePage']
        subpage_types = [ContentPage]

        serve = contentpage_serve

        base_form_class = ContentPageAdminForm

        content_panels = PageConfig.content_panels + [
            FieldPanel('body'),
            ImageChooserPanel('header_image'),
        ]


All of the non-data/non-model methods and attributes currently on ``Page``
should be moved to the ``PageConfig`` class.
This includes ``template``, ``serve``, ``route``,
``get_context``, ``get_template``
``subpage``/``parent_page_types``, ``allowed_subpage_models`` and similar,
``search_fields``, ``panels``, ``content_panels``, and similar,
``is_creatable``, ``can_exist_under``, ``can_move_to``,
``dummy_request``, ``preview_modes``, ``default_preview_mode``, ``serve_preview``,
``get_cached_paths``, ``get_sitemap_urls``, ``get_static_site_paths``, and
``serve_password_required_response``.

A new ``Registry`` class will record all the ``PageConfig``\s for all the
``Page`` subclasses.

.. code-block:: python

    class PageConfig(object):
        def __init__(self, model):
            self.model = model

        # All of the methods and attributes from Page named above should be
        # moved here


    def register(*models, config=None):
        if config is None:
            return lambda config: register(*models, config=config)
        for model in models:
            registry.add(config(model))
        return config


    class Registry(object):
        def __init__(self):
            configs = {}

        def add(self, config):
            self.configs[config.model] = config

        def __getitem__(self, model):
            return self.configs[model]


    registry = Registry()

The ``Registry`` can then be queried to get the configuration for a Page
subclass:

.. code-block:: python

    registry[page.specific_class].serve(request, page)

Backwards compatibility
-----------------------

For an app in ``INSTALLED_APPS``:
if ``<app>.wagtail_hooks`` does not exist,
or does not register any ``PageConfig``s,
but ``<app>.models`` defines one or more creatable ``Page`` subclasses,
those ``Page`` subclasses should be automatically registered,
and a deprecation warning issued.
An ``AutoPageConfig`` class should be created for this purpose
that proxies all the methods and attributes
that have been moved from ``Page`` to ``PageConfig``
back the ``Page`` subclass.

All the methods and attributes moved from ``Page`` to ``PageConfig`` should still exist,
but should be deprecated.
