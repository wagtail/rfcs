# RFC 18: Meta fields

* RFC: 18
* Author: Karl Hobley
* Status: Draft
* Created: 2017-03-07
* Last Modified: 2017-03-07

## Abstract

We are currently designing an REST API for creating and updating pages. This
will be used by external editing applications and may also be used within
the Wagtail admin interface as well.

There is a group of fields on the ``Page`` model that are ignored by revisions[0].
They're ignored to prevent any unwanted side-effects happening if that revision
is viewed or restored later on.

This RFC proposes that we replace the "meta" section of pages in the API with
this set of fields. Initially, this will apply to the admin API only, but could
also become the default for the public API as well in the next major version.

## Why?

The definition of a "meta" field in the API is currently "the set of fields
that are not displayed to frontend users". This definition doesn't make much
sense for the admin API. I think we should define this better so we know what
fields belong there and what don't.

Changing which fields go in "meta" to what is proposed above simplifies
development of clients a bit.

- To create a new revision from an existing page, the client just has to remove
  "meta".
- To create a full page object from a revision, the client just has to append
  the live version of "meta" to the revision content.

## The fields

The list of fields that will be included in ``meta`` are as follows:

- ``id``
- ``type``
- ``detail_url``
- ``html_url``
- ``status``
- ``locked``
- ``owner``
- ``first_published_at``
- ``parent``
- ``children``
- ``decendants``

This list would be the same for all page types. Specific page types cannot
change these fields.

## Meta fields within Wagtail

If we want to add a new field that is ignored by revisions, we would need to
make sure we reconfigure the admin API as well. There may also be cases where
a third-party app would want to find out the list of unversioned fields for the
version of Wagtail they are running on.

I'd like to also propose we add an attribute to ``PageRevision`` listing these
unversioned fields. this would be used by ``as_page_object`` to exclude these
fields and also the admin API to automatically work out which fields need to be
in "meta".

Perhaps something like:

```python

class PageRevision(models.Model):
    META_FIELDS = [
        'id',
        'path',
        'depth',
        'numchild',
        'url_path',
        'live',
        'has_unpublished_changes',
        'owner',
        'locked',
        'latest_revision_created_at',
        'first_published_at',
    ]

    ...
```

## Images and documents

This also extends to images and documents as well. The only difference would be
the ``tags`` field of both models will be moved out of "meta".

## Alternatives

### Keep it the way it is

### Remove "meta" entirely

## Footnotes

[0] https://github.com/wagtail/wagtail/blob/da067679cdd7999c256c8916f78c82f7641d51cd/wagtail/wagtailcore/models.py#L1423-L1441
