# RFC 74: Reorganisation of core modules

* RFC: 74
* Author: Karl Hobley
* Created: 2021-10-29
* Last Modified: 2021-10-29

# Summary

- All of the required “core” apps (`core`, `admin`, `users`, `sites`, `locales`) will be merged into one app called `wagtail`
- All optional apps will be moved into `wagtail.contrib`. This includes `images`, `documents`, `embeds`, and `snippets`
- `search` will be moved out into a third party plugin
- Core models split into separate modules
- Template tags renamed (for example `wagtailcore_tags` => `wagtail`)

# Motivation

## `wagtail.core` vs `wagtail.admin`

The split between core and admin is not always clear. One rule is that `core` should never import from `admin` and this is [broken in some places](https://github.com/search?q=wagtail.admin+repo%3Awagtail%2Fwagtail+path%3Awagtail%2Fcore&type=Code&ref=advsearch&l=&l=).

It can add extra cognitive load to developers when deciding how to build a new feature that needs to work in both areas. You usually need to pick one or split between the two, this needs to be carefully considered.

## `wagtail` vs `wagtail.contrib`

It’s not clear in our current structure that some apps can be removed or overridden. For example, `images` and `documents` can be removed or replaced but `users` and `search` cannot.

# Detailed specification

The refactoring will be carried out in stages in the following order:

## Move `UserProfile` model into Wagtail Core

We firstly need to move the `UserProfile` model into Wagtail core so that the users app can be removed.

There is already a PR for this [here](https://github.com/wagtail/wagtail/pull/7564).

## Move `wagtail/core` to `wagtail`

This will be carried out by moving all modules currently within `wagtail/core/` up to `wagtail/` then updating all imports and docs. We will add dummy modules into `wagtail/core` that re-export anything that may have been previously imported from there (such as the app config, blocks and models).

Despite the folder name change, the app label will remain `wagtailcore`.

**Conflicts**

There are a couple of conflicting files:

- [`wagtail/core/utils.py`](https://github.com/wagtail/wagtail/blob/ce3a3daffac6406f8475a92d9a08169d1fb371e4/wagtail/core/utils.py) will conflict with [`wagtail/utils`](https://github.com/wagtail/wagtail/tree/a51c26da43d37f0bc17881dbdfcae44804c04fd2/wagtail/utils)
    We will implement a temporary solution by moving `wagtail/core/utils.py` to `wagtail/coreutils.py` . We will then split this file up into separate modules that would live in [`wagtail/utils`](https://github.com/wagtail/wagtail/tree/a51c26da43d37f0bc17881dbdfcae44804c04fd2/wagtail/utils)

- [`wagtail/core/sites.py`](https://github.com/wagtail/wagtail/blob/8413d00bdd03c447900019961d604186e17d2870/wagtail/core/sites.py) will conflict with `wagtail/sites`
    Even though we’re removing the sites module, we still need to keep this folder for a couple of releases to hold re-export modules that maintain backwards compatibility.

    To work around this conflict, we will merge the contents of `wagtail/core/sites.py` into `wagtail/models/sites.py` since that is the only module that uses these utilities

- [`wagtail/core/tests/`](https://github.com/wagtail/wagtail/tree/8e46d3b06877c059fef27dd025440d9af6487a7f/wagtail/core/tests) will conflict with [`wagtail/tests`](https://github.com/wagtail/wagtail/tree/803316080fef7dd53571f97f5dfd0b3cf3873c5c/wagtail/tests)
    We will rename [`wagtail/tests`](https://github.com/wagtail/wagtail/tree/803316080fef7dd53571f97f5dfd0b3cf3873c5c/wagtail/tests) to `wagtail/test` since this folder contains test infrastructure and models, but no actual tests. So I think it makes sense for the folder that contains the tests to have the plural name here.

**Other cleanups**

- `WagtailCoreAppConfig` will be renamed to `WagtailAppConfig`

## Merge `wagtail.admin` into `wagtail`

After this step, `wagtail.admin` would no longer be a separate app, but most of the modules will remain in the same place.

The following parts of `wagtail.admin` will be moved:

- `wagtail.admin.edit_handlers` will move to `wagtail.edit_handlers`
    We will also move the current monkey patching of `Page` that happens in that file into `wagtail.core.models`
- `wagtail.admin.models` will move to `wagtail.core.models.admin`. We will also move the `UserProfile` model into that module too.
- The `wagtailadmin.can_access_admin` permission will be renamed to `wagtailcore.can_access_admin`
- `wagtail/admin/templates/wagtailadmin` will be moved to `wagtail/templates/wagtailadmin`
- `wagtail/admin/templatetags` will be merged into `wagtail/templatetags`
- `wagtail/admin/static_src` will be moved to `wagtail/static_src`
- `wagtail/admin/locale` will be merged into `wagtail/locale`
- `wagtail/admin/wagtail_hooks.py` will be moved to `wagtail/wagtail_hooks/admin.py` temporarily. They will be merged with Wagtail core hooks in a later step. We may have to split this module into a sub-folder because it’s getting quite big.

We will then remove `wagtail.admin` from `INSTALLED_APPS` in the project template and any tutorials. Using this app will raise a deprecation warning. We will remove the `apps.py` file in a later release.

## Break up the core models

The core models have been partially broken up into separate module. This RFC proposes to completely break them up so there is no longer any implementation in [`__init__.py`](https://github.com/wagtail/wagtail/blob/3e64deb1ac34c4d21c36156f93624e30ee7e6e71/wagtail/core/models/__init__.py).

We will also:

- Move [`wagtail/admin/models.py`](https://github.com/wagtail/wagtail/blob/0a8ab74d7037ee3e6a1b13582c2ee86449a06b84/wagtail/admin/models.py) to `wagtail/models/admin.py`
- Move [`wagtail/core/query.py`](https://github.com/wagtail/wagtail/blob/c6017abca042031fb2f7931b406a4c12e7a9e8a4/wagtail/core/query.py) to `wagtail/models/query.py`

The proposed final structure of the `wagtail/models`  folder is as follows:

- `admin.py` - Moved form `wagtail/admin/models.py` with `UserProfile` added
- `collections.py` - Existing module. Unchanged.
- `commenting.py` - New module. Contains `Comment`, `CommentReply`, etc
- `copying.py` - Existing module. Unchanged.
- `i18n.py` - Existing module. Unchanged.
- `logging.py` - Renamed from `audit_log.py` We will add `PageLogEntry` and related models.
- `pages.py` - New module. Contains pages, revisions, manager, etc.
- `query.py` - Moved from `wagtail/core/query.py`
- `sites.py` - Existing module. Unchanged.
- `view_restrictions.py` - Existing module. Unchanged.
- `workflows.py` - Workflow models (`Workflow`, `Task`, `TaskState`, `PageWorkflow`, etc)

You can view this structure on my PoC branch [here](https://github.com/wagtail/wagtail/tree/f6cca37aa00d6cef855059969d1032271c15de12/wagtail/models)

## General reorganisation

The following modules that are currently in [`wagtail/core/`](https://github.com/wagtail/wagtail/tree/24b99a5ebc4b25986b89bbbfa0de6e93acfe6dcc/wagtail/core) will be moved to [`wagtail/utils/`](https://github.com/wagtail/wagtail/tree/a51c26da43d37f0bc17881dbdfcae44804c04fd2/wagtail/utils)

- [`utils.py`](https://github.com/wagtail/wagtail/blob/ce3a3daffac6406f8475a92d9a08169d1fb371e4/wagtail/core/utils.py) (broken up into smaller modules)
- [`compat.py`](https://github.com/wagtail/wagtail/blob/f2f4503f4ffb1130e770a3dd4d98ef6e47864de3/wagtail/core/compat.py)
- [`treebeard.py`](https://github.com/wagtail/wagtail/blob/8f8f2e12b731334bbe1e39508a12246a3503f532/wagtail/core/treebeard.py)
- [`url_routing.py`](https://github.com/wagtail/wagtail/blob/8e25960972a1663b14faca1f7ac6a1693a581b18/wagtail/core/telepath.py)
- [`whitelist.py`](https://github.com/wagtail/wagtail/blob/ba6f94def17b8bbc66002cbc7af60ed422658ff1/wagtail/core/whitelist.py) -> should be renamed to `allowlist` or `safelist` and then `Whitelister` module renamed to `Gatekeeper` or `Safelister`

[`log_actions.py`](https://github.com/wagtail/wagtail/blob/9b28fdba87515bd679806cd1c51ffb04918c15c7/wagtail/core/log_actions.py) will be renamed to `logging.py` as this module contains more than just log actions, but also for consistency with it’s models file (`wagtail/models/logging.py`)

[`telepath.py`](https://github.com/wagtail/wagtail/blob/8e25960972a1663b14faca1f7ac6a1693a581b18/wagtail/core/telepath.py) will be moved to `wagtail/admin`

`PageClassNotFoundError` will be moved from [`wagtail/exceptions.py`](https://github.com/wagtail/wagtail/blob/10312cb4da29b842ec04ed3c6f318664a5116749/wagtail/core/exceptions.py)` to `[wagtail/admin/views/pages/edit.py](https://github.com/wagtail/wagtail/blob/e17647e466ad0189e170bf6db45c8783e2e3df26/wagtail/admin/views/pages/edit.py) as this is the only module that raises and consumes this error. This is the only class in [`wagtail/exceptions.py`](https://github.com/wagtail/wagtail/blob/10312cb4da29b842ec04ed3c6f318664a5116749/wagtail/core/exceptions.py) so this file will be deleted.

## Move optional apps into `wagtail.contrib`

To make it clearer that they are optional, we will move the following apps into `wagtail.contrib`:

- `wagtail.images`
- `wagtail.documents`
- `wagtail.embeds`
- `wagtail.snippets`

This also gives the `wagtail.contrib` section a clear purpose as all apps that are not in this folder are compulsory.

## Merge compulsory apps into `wagtail`

All compulsory apps (`wagtail.users`, `wagtail.locales`, and `wagtail.sites`) will be merged into the main `wagtail` app.

There are a couple of exceptions:

- `wagtail.search` contains models and has a unique position in our app structure as being the only app that `wagtail.core` currently depends on. When we remove the `Query` models ([PR here](https://github.com/wagtail/wagtail/pull/7277)) this app will no longer depend on anything else in Wagtail so it could be moved into an external app
- `wagtail.api.v2` is only installable as an app because of it’s signal handlers that hook into `wagtail.contrib.frontend_cache`. We will look into removing this in the future.

**Merge strategy**

- Their admin views, forms and widgets and related tests will be moved into `wagtail/admin`
- Their `wagtail_hooks.py` files will be moved into the `wagtail/wagtail_hooks` folder. These hooks files will be merged later
- Their locale files will be merged into `wagtail/locale`
- Their template files will be moved under `wagtail/templates` and template tags moved into `wagtail/templatetags`

We will update the documentation and project template to no longer require these apps in `INSTALLED_APPS`. Re-export modules will be provided for two releases to keep existing imports working. After that, these folders will be completely gone.

## Rename and reorganise template tag libraries

As a bonus task, we will also rename the template libraries:

- `wagtailcore_tags` will be renamed to `wagtail`
- `wagtailuserbar` will be merged into `wagtail`
- All libraries that have `_tags` on the end will have this suffix removed
- `wagtailusers_tags` will be renamed to `wagtailadmin_users` and removed at a later date
- Tags related to the admin sidebar will be extracted into a `wagtailadmin_sidebar` tags library to reduce the size of the `wagtailadmin` tags library (there are a lot a tags just for the sidebar)
