# RFC 15: Revisions Admin API (Revised)

* RFC: 15
* Author: Karl Hobley, Marco Fucci, Ravi Kotecha, Tyom Semonov
* Created: 2017-02-22
* Last-Modified: 2017-06-30

## Abstract

This RFC 15 is an updated version of the [RFC 11](011-revisions-api.md) by Karl Hobley.
It was created to allow more time for discussing the new Admin API and
has some parts copied from the original RFC 11.

Currently, the Admin API can only access the live version of a page's content.
In order to support page editing, we must allow the admin API to access the
latest draft and also create new revisions and publish them.

I also think that the revisions UI currently in Wagtail would be better
implemented using the admin API as this would allow us to implement it in a
way that doesn't require the user to navigate away from the editor.

This RFC proposes an extension to the Admin API to allow page revisions to be
retrieved and created.

Some of the changes are inspired by the
[Google Drive API](https://developers.google.com/drive/v3/reference/revisions).

## Specification

### Data format

All of the responses return a dictionary containing the content fields of the
page with a "meta" section for the revisions metadata. All editable fields on
the page (except meta fields as defined by RFC 18) are included and they are
all serialized using Django REST Framework (so not the same as the JSON in the
database).

The "meta" section contains other fields from the ``PageRevision`` model. This
includes: ``id``, ``page``, ``author`` and ``in_moderation``

For example:

```json
{
    "meta": {
        "id": 1,
        "page": {
            "meta": {
                "id": 1,
                "type": "demo.HomePage"
            }
        },
        "created_at": "2016-07-18T16:37:00+01:00",
        "author": "karl",
        "in_moderation": false,
    },
    "title": "Home",
    "body": "foo"
}
```

### URL structure

We will add a new endpoint underneath the pages endpoint allowing access to
revisions of specific pages.

### Endpoints

#### View revisions

This will be paginated to show up to 20 revisions at a time.

The list will be formatted the same way as other listings in the API:

```json
{
    "meta": {
        "total_count": 50,
    },
    "items": [
        # Revisions here
    ]
}
```

##### All revisions of a page

```http
GET /api/pages/<page-id>/revisions/
```

##### All revisions

You can use ``'-'`` if you don't want to filter by a specific page id.

```http
GET /api/pages/-/revisions/
```

##### Filter by author

Filters by the value of the ``USERNAME_FIELD`` on the user model

```http
GET /api/pages/<page-id>/revisions/?author=<author-username>
```

#### Get a particular revision

##### By page and revision id

```http
GET /api/pages/<page-id>/revisions/<revision-id>/
```

This returns a 404 error if ``<revision-id>`` does not reference a
revision that belongs to the page.

##### By revision id only

You can use ``'-'`` instead of ``<page-id>`` if you only care about or know the revision id.

```http
GET /api/pages/-/revisions/<revision-id>/
```

This redirects to ``/api/pages/<page-id>/revisions/<revision-id>/``

##### Get the latest revision of a page

To get the latest revision of a page, you use ``'head'`` as
``<revision-id>``:

```http
GET /api/pages/<page-id>/revisions/head/
```

This redirects to ``/api/pages/<page-id>/revisions/<revision-id>/``

This also works when the page id is ``-``, this redirects to the
latest revision of the site:

```http
GET /api/pages/-/revisions/head/
```

#### Create a new revision

Creating a new revision is done by submitting a revision in the above
format (excluding "meta") to the following URL:

```http
POST /api/pages/<page-id>/revisions/
```

Note: The currently logged in user must have permission to edit the page
or a 403 error will be returned.

Note: Unlike the previous URLs, you must specify a page number. Specifying
``-`` as the page ID will return a 404 error.

#### Create and publish a revision

To publish a revision, you pass ``?then=publish`` to the create endpoint.

```http
POST /api/pages/<page-id>/revisions/?then=publish
```

Note: The currently logged in user must have permission to publish the page
or a 403 error will be returned.

#### Create and submit a revision for moderation

To submit a revision for moderation, you pass ``?then=submit-for-moderation``
to the create endpoint.

```http
POST /api/pages/<page-id>/revisions/?then=submit-for-moderation
```

#### Approve moderation

To approve a revision previously submitted for moderation:

```http
POST /api/pages/<page-id>/revisions/<revision-id>/moderation/approve/
```

This returns a 400 error if the revision has not been submitted for moderation.
This returns a 403 error if the user does not have permission to approve the revision.

#### Reject moderation

To reject a revision previously submitted for moderation:

```http
POST /api/pages/<page-id>/revisions/<revision-id>/moderation/reject/
```

This returns a 400 error if the revision has not been submitted for moderation.
This returns a 403 error if the user does not have permission to approve the revision.

### Locked pages

A page cannot be edited in any way if it is locked, so attempting to create a
new revision for a locked page will result in a ``423 Locked`` response code
and the new revision will not be saved.
