# RFC 12: App rationalisation

* RFC: 12
* Author: Karl Hobley
* Status: Draft
* Created: 2016-07-19
* Last-Modified: 2016-07-19

## Abstract

Over time, some inconsistencies have developed in Wagtail's project structure.
This RFC proposes some changes we could make to rationalise the structure.

## Proposed changes

The changes proposed here are rather idealistic and it may not be possible or
even worth implementing them all.

### Core apps that should be contrib apps

The ``wagtailforms``, ``wagtailredirects`` and ``wagtailembeds`` apps were
created before Wagtal had a section of the tree for "contrib" apps. These
should now be moved into ``contrib``.

### The "wagtail" prefix

Before Django 1.7, two apps could not have the same name. To prevent name
clashes, we namespaced all of Wagtail's apps with an extra ``wagtail`` prefix.
Recently added apps, such as ``modeladmin`` and ``settings``, do not have this
prefix.

As we no longer support Django 1.6 we should drop this prefix from all apps in Wagtail.

### docs vs documents

The ``wagtaildocs`` app should be renamed to ``documents`` as part of the work
to drop prefixes.

### Apps that just provide admin interfaces

``wagtailsites`` and ``wagtailusers`` only purpose is to add some views to the
admin. Sites are implemented in ``wagtailcore`` and users either use Djangoo's
default implementation or are customised by the developer.

Since these apps only implement admin ui, they should be merged into
``wagtailadmin``.

## How this should be done

The merging of ``wagtailsites`` and ``wagtailusers`` into ``wagtailadmin``
shouldn't be very difficult as neither of these apps expose a public API.

Removing of the ``wagtail`` prefix, however, would require major changes to
most sites built on Wagtail. As we need to make it as easy as possible to
upgrade between Wagtail releases, this part could be quite difficult for us
to implement.

How we go about deprefixing Wagtail apps is open to discussion. Here are a
couple of open questions:

### How will we make it backwards compatible?

Two solutions to this so far have been proposed:

#### Create replica apps using the old names that import the public API from the new apps and display a warning

See: https://github.com/torchbox/wagtail/pull/2089

This requires us to re-export every part of every module which is a lot of work.

#### Use a custom module loader to automatically redirect imports to the new apps

See: https://github.com/torchbox/wagtail/pull/2089/commits/980c4fd4bb625e2f96c68be9752a70bb357ff60d

We haven't yet managed to get this to redirect model definitions though so
developers will have to update references to ``wagtail.wagtailcore.models.Page``.

#### Should this be done in one go, or in phases?

Doing this in phases would be easier for us but could be annoying for developers
as this would mean they need to make major changes to their sites every 8 weeks.

