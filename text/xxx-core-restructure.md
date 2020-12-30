# RFC XXX: Simplifying Wagtail's project structure

## Abstract

This RFC proposes some changes to Wagtail's project structure to make it clearer for new users and provide some space for Wagtail to grow. Except for some module renaming we did for Wagtail 2.0, we haven't revisited Wagtail's project structure since it was first developed in 2013. Wagtail has grown a lot since then, and all trees need pruning once in a while to keep them healthy.

## Summary of proposed changes

 - Merge ``admin``, ``api``, and ``search`` into ``core``
 - Move the core app from ``wagtail.core`` to ``wagtail``
 - Restructure it by functionality, for example:
   - Common admin/API functionality would live in ``wagtail.admin``, ``wagtail.api`` and ``wagtail.admin.api``
   - Split into sub-apps for ``pages``, ``sites``, ``locales``, and ``media`` (common code for images/documents like collections)
 - Move all other optional apps into ``wagtail.contrib``
   - This means that ``images``, ``documents``, ``snippets``, ``sites``, ``locales``, ``users``, and ``embeds`` would be moved under ``wagtail.contrib``
   - `sites``, ``locales`` and ``users`` could later be moved into the core when we introduce admin modules

## Motivation


### core vs admin

If you want edit handlers, you import from ``wagtail.admin.edit_handlers`` but if you want StreamField blocks, you need to import these from ``wagtail.core.blocks``.

Rich text, logging, and hooks are all examples of functionality that isn't clear on which app actually owns it.

Merging into core allows us to structure the code by type, so all pages code including models and admin could live under ``wagtail.core.pages`` and sites could live under ``wagtail.core.sites``, and so on.

### wagtail vs wagtail.contrib

It's not clear why some apps go into ``wagtail.contrib`` and others are placed directly under ``wagtail``

### An outlier: wagtail.search

Wagtail Search is unique in that it is the only app that ``wagtail.core`` itself depends on.
This is because pages need to be searchable, but the ``Page`` model is defined in core.

### Core features are optional apps

Features like core, admin, search, api are all separate apps.

This actually makes it easier for us to support, we know that all sites have the admin module available, even if they're not using Wagtail admin

### Code isn't well co-located

If you were looking for the page model, you should look in ``wagtail.core.models``

If you want to look at the pages admin views, have a look at ``wagtail.admin.views.pages``

If you want to look at the pages adin forms, look in ``wagtail.admin.forms.pages``

The API is in ``wagtail.api.v2.pages`` and the admin API is in ``wagtail.admin.api.pages``

### Supporting new media classes

Separate RFC

## Proposed changes

In summary:

 - Merge ``admin``, ``api``, and ``search`` into ``core``
 - Move the core app from ``wagtail.core`` to ``wagtail``
 - Restructure it by functionality, for example:
   - Common admin functionality would live in ``wagtail.admin``
   - Ditto that for API
   - All pages, models, apis, views, etc will be under ``wagtail.pages``
 - Move all other optional apps into ``wagtail.contrib``

I've picked these operations as they are easy to do without breaking backwards compatibility.

In summary, I propose that we:

 - Merge the ``admin`` and ``api`` modules into ``core``
 - Divide ``core`` into sub-applications (for example ``wagtail.core.pages``) that contain the models, admin views and api for a particular entity co-located in a single place
 - Create a new ``wagtail.core.media`` sub-app that contains common functionality for media classes (documents and images) to make it easier for people to implement new media classes for Wagtail (videos, vector graphics, audio, 3D objects, etc)


NOTE: Present the idea sort of like this:

 - We should structure the code by functionality (pages, media, etc) instead of aspects (views, api, admin).
 - The easiest way to restructure without harming backwards compatibility would be to merge api and admin into core, then split that up by functionality. With common functionality directly under `wagtail.core` and specific functionality in sub-apps (`wagtail.core.pages`, etc)


More stuff:

 Also merge search into core
 Move all optional apps into contrib
 Move wagtail core to wagtail

## Motivation

### Wagtailcore models.py is now really big

Wagtail core's models.py has grown rapidly over the past year. This makes it difficult to find code that you're looking for, but more importantly, makes it harder for new contributors to understand how the code is linked together.

#### Doesn't merging admin and API make this situation worse?

To improve the modularity of wagtail core, we would need to split it up somehow anyway. I feel that once this is done, the resulting apps will be pretty small, allowing plenty of space to merge admin and api into these apps.

I think it's time to start thinking of core as a 

### Functionality that is split between the admin and core

Logging, edit handlers, rich text, streamfield, hooks, are all examples of code that is implemented across both admin and core already

### There's no need for a separate admin app

I don't think there is a benefit for having admin as a separate app. I think the only argument for it is that it allows users to easily remove the admin interface from Wagtail, but this could also be done by not registering it in urls.py.

### Code should be co-located

The page model is implemented in core, but its admin interface is implemented in admin. I think it's better to divide modules

### Since Django 1.7, large apps don't really matter

I remember in the early days of Wagtail, apps had to have all of their models in the models.py file at the top level of the app. This is no longer the case, we can now divide the models file up between multiple sub-apps. So we get the benefit of having one app to register but we have code nicely divided up.

### Splitting functionality into separate Django apps makes it hard to restructure code

IF all functionality is within one app, you can build an internal structure in that app. 

If you split it up, it's very hard to move models between apps which can make refactorings quite difficult.

There isn't a need to have separate apps for non-contrib.


PErhaps there's a beenfit to being able to remove the admin app, but I've never came across that. Even if people did remove the admin from URLs, I don't think people would complain about having a few extra static files.:

## Proposed changes

### Merging ``admin`` and ``api`` into ``core``

The first change this RFC proposes is to merge the admin and api modules into core.


None of these apps currently have models so this merging should be quite easy

### Divide the core app into multiple sub-apps

The sub-apps that I propse are as follows:

  - Pages - Includes the ``Page`` model along with the admin/api
  - Media - Includes the ``Collection`` model and admin code
  - Sites - Includes the ``Site`` model
  - Locales - Includes the ``Locale`` model

wagtail/core
    /admin - Common admin functionality
    /api   - Common API functionality
    /pages
        /admin
        /api
        models.py
    /media
        /admin
        /api
        models.py
    /sites
        models.py
    /locales
        models.py

    models.py  - Re-exports models from sub-apps so that Django can see them


### Merge sites, locales, and users into core

These modules only provide admin interfaces

### Create a ``media`` sub-app that contains the ``Collection`` and a new ``AbstractBaseMedia`` model.

The idea of this app is that it will later hold the common functionality that all media classes (images, documents, etc) will need to support:

 - Uploading
 - Serving
 - Permissions
 - Choosers
 - Object usage
 - CRUD views
 - API and Admin API

This should make it very easy for people to build their own custom media classes such as video, audio, 3D objects and vector graphics. Implementors of new media classes will only need to work on the code that is specific for that media class (eg, video encoding, rendering 3D models on the frontend, etc). Wagtail would provide the common functionality. This change will make it very easy for custom media classes to be implemented, and also make all media classes very consistent.

This app will initially contain the ``Collection`` and a new abstract model called ``AbstractBaseMedia``. ``AbstractBaseMedia`` contains the fields that all media classes need such as the title, and a foreign key to ``Collection``. This class will become the base class of the existing ``AbstractImage`` and ``AbstractDocument`` models.

Over time, we would move the functionality listed above out of ``images`` and ``documents`` into ``core.media``.

## What about merging ``locales``, ``sites`` and ``users``?

These three apps only provide admin interfaces for models that are defined in ``core`` and Django, so I think they also make sense as sub-apps of ``core``.

However, if there were moved here, there would be no way to disable them so that they could be overridden.

So I think we should leave these as separate apps for now, but we can merge them later if we add a setting or hook to allow them to be disabled.





## Models

Before we can restructure Wagtail in this way, we firstly need to deal with some models.

### ``wagtailusers.UserProfile``

As the ``wagtailusers`` app is going to be moved into contrib, we must move this model into core since it is required by the admin.

### ``wagtailadmin.Admin``

This should be easy to move into core as it is an unmanaged model and only exists to add a permission. The only issue here is we would have to rename the ``wagtailadmin.can_access_admin`` permission to ``wagtailcore.can_access_admin``

### ``wagtailsearch`` models

``wagtailsearch`` has two models ``Query`` and ``QueryDailyHits``, these need to be moved somewhere else before we can merge ``wagtailsearch`` into core.

These models are only used for the ``wagtailsearchpromotions`` app to get statistics on popular search queries so that effective promotions can be made. So for now, I think the best approach would be to move those models into this app. I think long term Wagtail could do with a general analytics app for this sort of thing, but maybe for another RFC!

``wagtailsearchpromotions`` also relies on ``wagtailsearch`` to create an ``EditorsPick`` model. ``wagtailsearchpromotions`` renames this model in its initial migration. It was done this way because ``EditorsPick`` used to be part of ``wagtailsearch`` and this was the easiest way to move it into the other app at the time. We will need to rethink this if we are going to remove the ``wagtailsearch`` app though.


# Step 1

Add a migration to delete the ``EditorsPick`` table from the database if it exists, and update the initial ``wagtailsearchpromotions`` migration to create a fresh model.

This was done years ago so anyone who needs to migrate to ``wagtailsearchpromotions`` has probably already done so.

# Step 2

Create new ``Query`` and ``QueryDailyHits`` models in ``wagtailsearchpromotions``. Add a data migration to migrate all the data across to the new models. Deprecate the models in ``wagtailsearch``.

# Step 3

Delete the models from ``wagtailsearch``.

We can keep the migrations around in the ``wagtail.search`` but recommend that people remove ``wagtail.search`` from ``INSTALLED_APPS`` after applying the migrations.

The ``wagtail.search`` module will continue to exist but will no longer be its own Django app. But we can keep the migrations folder in place so that people can block-update Wagtail.
