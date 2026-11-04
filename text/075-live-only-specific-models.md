# RFC 75: Live-only specific models

* RFC: 75
* Author: Matt Westcott
* Created: 2022-02-03
* Last Modified: 2022-02-03

## Abstract

This RFC proposes a change to the database representation of unpublished pages, where the database record for the specific page model is omitted.

## Motivation

Currently, a newly-created page must pass all validation constraints, including filling in all required fields, before it can be saved as a draft for the first time. We now wish to introduce auto-saving to the page editor. It's a reasonable expectation that auto-saving should start saving work-in-progress immediately, without having to bring the page up to some threshold of completeness. This presents a problem in how we save those initial snapshots of the page data to the database: we can't create PageRevision records until a Page record exists (allocating an ID and storing essential metadata such as the page type and tree position), but currently Page records are only created in conjunction with a database entry for the specific page type, and that entry can't be created while there are unsatisfied validation constraints (e.g. unfilled required fields) at the model or database level.

The proposed solution is to save only a base Page record for non-live pages, omitting the record for the specific page type so that the complete draft page data only exists within PageRevision records.

## Current behaviour

For the purposes of the following description, let's say that `blog.BlogPage` is a model for a specific page type.

On receiving a valid form submission with an action of 'save draft', the 'create page' view saves a record to the `wagtailcore_page` table with the `live` flag set to False, along with a database record in `blog_blogpage` linked via the `page_ptr_id` field. It then saves a `wagtailcore_pagerevision` record containing the full page data serialised as JSON.

Visiting the 'edit page' view for that page will retrieve the latest `wagtailcore_pagerevision` record and render the edit form based on that; a valid form submission with an action of 'save draft' will add a new `wagtailcore_pagerevision` record and leave the `wagtailcore_page` record unchanged other than certain metadata fields (`draft_title`, `latest_revision_created_at` and `has_unpublished_changes`) and the `blog_blogpage` record completely unchanged. This means that for an unpublished page, the page data in the 'real' database record will always remain in either the initial state from the 'create page' view, or in the case of a page which was published and then unpublished, the last published state.

In some situations, third-party packages may use the otherwise stale `blog_blogpage` record for their own purposes. For example, `wagtail-experiments` stores the variations of a published page as distinct page objects with `live=False` (so that they don't show up as duplicates in front-end navigation), but with real approved content that may be shown to a front-end user.

## Model-level operations

In the proposed new model, non-live pages (including ones submitted for moderation, or awaiting go-live) will be represented as `wagtailcore_page` records with `live` set to False and `content_type` set to `BlogPage`, but with no corresponding `blog_blogpage` record. `wagtailcore_pagerevision` will track changes to the page content as it does now.

`wagtailcore_page` records can be trivially created without a linked `blog_blogpage` record, by instantiating and saving a basic `Page` object.

Taking a page from draft state to published will require adding a `blog_blogpage` record to the existing `wagtailcore_page` record. Formally supporting this is [currently an open issue in Django](https://code.djangoproject.com/ticket/7623), but this can be achieved as follows:

    page = Page.objects.get(id=123)
    blogpage = BlogPage(body="some specific content")
    blogpage.page_ptr = page
    blogpage.path = page.path
    blogpage.depth = page.depth
    blogpage._state.adding = False
    blogpage.save()

The `blogpage._state.adding = False` line is needed to ensure that the uniqueness check on the `path` field excludes the existing page record.

Unpublishing a page would require dropping the `blog_blogpage` record while preserving the `wagtailcore_page` record. This may require a direct SQL query (to be confirmed).

## Impact on Wagtail and user code

This change is likely to have a significant impact on end-user code, and would certainly call for a major version bump of Wagtail when introduced.

### Querying on specific Page subclasses

A query such as `BlogPage.objects.all()` will now only return live pages. For typical frontend-facing code such as listing pages, this will most likely be a good thing. Code that intentionally works with unpublished pages (e.g. ModelAdmin index pages within the Wagtail admin) would need to use `Page.objects.type(BlogPage)` instead, and be able to handle basic `Page` instances as results. (Given that as things currently stand, the 'specific' data for unpublished pages is liable to be stale - hopefully existing view code of this sort is not relying on it, beyond simple things like `get_admin_display_title`.)

### `.specific` methods

For non-live pages, there is no longer any meaningful value we can return from `Page.specific`, and so this will raise a `BlogPage.DoesNotExist` exception (with a suitable explanatory message). It's up to the developer to decide on the most sensible alternative action in this case; if it's appropriate to retrieve the current draft instead, they can use `Page.get_latest_revision_as_page()`.

(Note: The [`get_specific`](https://docs.wagtail.org/en/stable/reference/pages/model_reference.html#wagtail.core.models.Page.get_specific) method introduced in 2.12 sets a precedent of returning a basic `Page` instance when the specific record is missing; however, this currently only happens when the page model definition has been dropped, which can be considered an edge case. My feeling is that extending this to happen more generally for unpublished pages will lead to behaviour that's unexpected by most developers.)

The behaviour of `PageQuerySet.specific` is more contentious, since this may be dealing with a mix of published and unpublished pages - I'd propose that any non-live pages from the result set are left as `Page` instances in the output, as this keeps the behaviour as close as possible to how it is now, while also avoiding 'slow-burning' failures where a previously-working query (that was previously only tested against all live pages) suddenly stops working in the presence of a draft page. (An alternative would be for non-live pages to be silently dropped from the result set; this would be more consistent with the new behaviour of `BlogPage.objects.all()`, but would make it unusable for listings in the Wagtail admin - see the `get_admin_display_title` section below.)

### `save()` operations

Calling `save()` on a specific page model with `live=False` will throw a validation error, as otherwise this would create a `blog_blogpage` record that we don't want. (The alternative would be that we override `save` and decide on some 'correct' behaviour for it to implement, but this feels like too much 'magic' - should it create a revision, or silently leave the specific data unsaved? Better for it to fail and make the developer fix their code to be explicit about what they want.)

Hopefully, all situations where a developer might be inclined to use `save()` have more explicit counterparts:

* Making an immediate change to live page content:

      blog_page = BlogPage.objects.get(id=123)
      blog_page.body += '...updated'
      blog_page.save_revision().publish()  # NB will overwrite any existing draft edits

  (`blog_page.save()` would update the live page, but the change would fail to show in the edit view as this takes data from the latest page revision instead.)

* Making a draft edit to a page that may or may not be live:

      blog_page = Page.objects.get(id=123).get_latest_revision_as_page()
      blog_page.body += '...updated'
      blog_page.save_revision()

  (It would really be a good idea for Wagtail to provide a helper function for automated edits combining the above two cases, so that the necessary change happens on draft and / or live revisions as appropriate. But that's out of scope for this development)

* Updating non-content metadata, e.g. setting `path` to move a page - `save()` on the base `Page` model will continue to work for this
* Publishing a draft: `Page.objects.get(id=123).get_latest_revision().publish()`

### `get_admin_display_title` and other per-page-model overrideable methods

It's common practice for page listings within the Wagtail admin to apply `specific()` to the queryset - this is primarily so that overridden `get_admin_display_title` methods are respected, although the same principle comes into play for certain other overrideable methods (such as `get_url_parts` when outputting 'view live' links). As noted above, this mechanism is already somewhat flawed for non-live pages since the 'specific' database record is likely to be stale; but in the new implementation, where the specific record is missing entirely for these pages and thus `specific()` returns a basic Page instance, custom `get_admin_display_title` methods won't be respected at all.

To address this, a new `admin_display_title` field will be added to the `Page` model; this will be populated from the 'edit page' view whenever a new revision is saved, by calling `get_admin_display_title` on the in-memory specific page instance. This way, the up-to-date display title will always be available for use in listings even for basic Page instances.

If there are any other overrideable methods that need to be respected for non-live pages in listings (or other situations where calling `get_latest_revision_as_page()` would be too much overhead), they will need to be accounted for in a similar way. (Clearly `get_url_parts` is not an issue here, as it will only ever be meaningful for live pages.)
