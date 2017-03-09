# RFC 8: Wagtail API

* RFC: 8
* Author: Karl Hobley
* Status: Accepted
* Created: 2016-04-29
* Last Modified: 2016-04-29


# Abstract

This RFC gives an overview of the plan I'd like to take for the next phase of
Wagtail API development.

## 1 - The APIs

This section gives a quick overview of the APIs we will support in Wagtail,
why we want them and how we'll support them.

### 1.1 - Public and admin APIs

Wagtail will have two APIs, the public API and the (private) admin API. The
public API would be used primarily by the site implementor for providing
content to their site frontend, mobile phones, external sites/tools, etc. The
admin API would be used for providing data for some components of the Wagtail
admin (eg explorer interface, choosers, etc).

Some of the main differences between the two APIs are:

- The admin API requires a logged in user with privileges to access the admin
  interface; all relevant admin permissions will be applied to the admin API as
  well
- All content will be available in the admin API but only publically-accessible
  content will be available in the public API (no draft pages or pages behind a
  restriction)
- The admin API would also include fields that either don't make sense to be in
  the public API (live/has_unpublished_changes) or are expensive to query in
  listings (parent/descendants)
- Write actions may be implemented in the admin API, but not in the public API

We'll try to keep them as similar as possible to improve maintainability and
also allow us to create a single client-side library that works for both.

RFC for the admin API here: https://github.com/torchbox/wagtail/issues/1760

PR containing initial work on the admin API: https://github.com/torchbox/wagtail/pull/2507

### 1.2 - Stable and unstable versions

For the public API, two versions will be supported, the "stable" version and
the "unstable" version.

Site implementors will have the choice of which version to use. The "stable"
version is almost guaranteed to never change whereas the "unstable" version
contains the latest features but may have breaking changes between Wagtail
releases. I expect projects that have clients which are hard to update (eg,
mobile phone apps) would use the "stable" version.

All new features and changes will only be made in the "unstable" version and
the "stable" version will only receive bug fixes. However, there may be some
exceptions:

- Fixing security issues
- Changes elsewhere in Wagtail that change/remove a feature exposed in the API

We will support only one version of the admin API at a time, which would be
based on the "unstable" version of the public API. As such, there will be no
guarentees for its stability between Wagtail releases.

#### Stabilisation process

Here's what we'll do when we want to stabilise the next version of the API:

- The current "stable" version will be removed from Wagtail (it could be moved
  into a separate package so those who rely on it can still use it, but support
  would be community based)
- The current "unstable" version will become the new "stable" version
- The code for the new "stable" version will be copied to create the new
  "unstable" version
- The admin API will be updated to be based on to the new "unstable" public API

#### Release cycle

We haven't defined period of time for releasing new versions of the public API
so they'll be released when we feel they're ready.

## 2 - Project structure

This section describes where the implementations for the API's described in the
previous section will sit in Wagtail.

### 2.1 Public API

``v1`` of the public API will continue to live at ``wagtail.contrib.wagtailapi``.

New versions starting with ``v2`` will be implemented across Wagtail. The
"core" parts containing the base classes and page endpoints will be implemented
in the ``wagtail.api`` module. The images endpoint will be implemented in the
``wagtail.wagtailimages.api`` module and the documents endpoint will be
implemented in the ``wagtail.wagtaildocs.api`` module.

Each API module will contain submodules for each version (eg,
``wagtail.api.v2``, ``wagtail.wagtailimages.api.v2``, etc).

#### Routers

The API module has a concept of a "router" which is used to combine a bunch of
endpoints together to form an API itself.

This class was previously hidden away in a ``urls.py`` file and the developer
just had to include that file in their root url conf. Starting with ``v2``,
developers are required to construct this class and register it themselves.

There's two reasons for this:

- The API is now spread across multiple modules so there isn't really a good
  place for us to define the router anymore
- Gives the developer full control of the API endpoints and makes customising
  the API more obvious

### 2.2 Admin API

The "core" parts of the admin API will be implemented in ``wagtailadmin.api``.
As it's not versioned, there will not be any version-specific subfolders.

The ``images`` endpoint will be implemented in
``wagtail.wagtailimages.api.admin`` and the ``documents`` endpoint will be
implemented in ``wagtail.wagtaildocs.api.admin``. Any new endpoints we add will
follow this same pattern (unless it makes more sense to implemented them in the
"core" like the ``pages`` endpoint.

The endpoint classes will be registered with a hook and the router will be
built by ``wagtailadmin``.
