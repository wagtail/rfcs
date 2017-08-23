# RFC 12: App rationalisation

* RFC: 12
* Author: Karl Hobley
* Created: 2016-07-19
* Last-Modified: 2017-08-18

## Abstract

Over time, some inconsistencies have developed in Wagtail's project structure.
This RFC proposes some changes we could make to rationalise the structure.

## Proposed changes

### The "wagtail" prefix

Before Django 1.7, two apps could not have the same name. To prevent name
clashes, we namespaced all of Wagtail's apps with an extra ``wagtail`` prefix.
Recently added apps, such as ``modeladmin`` and ``settings``, do not have this
prefix.

As we no longer support Django 1.6 we should drop this prefix from all apps in
Wagtail.

### docs vs documents

The ``wagtaildocs`` app should be renamed to ``documents`` as part of the work
to drop prefixes.

### Core apps that should be contrib apps

The ``wagtailforms`` and ``wagtailredirects`` apps were created before Wagtail
had a ``contrib`` section so we should move these to contrib.

## Backwards compatibility

As per our deprecation policy, we would usually want the old module paths to
continue to work for two releases after making the change so that third-party
apps can support multiple versions of Wagtail.

However, making everything in Wagtail importable from two places will be
extremely difficult to both do and maintain. We've tried both a manual and
automatic approach as a test on a few modules but neither way worked well and
both were difficult to set up for a small part of the project.

With this in mind, we've decided that we will make this change in Wagtail 2.0,
which is set to be released later this year, and not provide any backwards
compability. We will instead aim to make it as easy as possible to migrate by
providing a script that automatically rewrites imports to their new paths.
