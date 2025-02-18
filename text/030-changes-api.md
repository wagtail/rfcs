# RFC 30: Changes API

* RFC: 30
* Author: Karl Hobley
* Created: 2018-07-12
* Last Modified: 2018-07-12

## Abstract

This RFC proposes extending Wagtail's API to allow fetching all modifications
to pages since a point in time.

Use cases include periodically re-rendering static sites and replication with
external services.

This specification is inspired by the
[Realtime Paged Data Exchange](https://www.openactive.io/realtime-paged-data-exchange/)
specification from OpenActve.

## Specification

We will add a new endpoint at ``/api/v2/page-changes/`` which returns a list of
modifications.

### Configuration

The endpoint will not be enabled by default. To enable it, developers will have
to add the following
to their API router:

```python

# api.py

from wagtail.api.v2.endpoints import PagesAPIEndpoint, PageChangesAPIEndpoint
from wagtail.api.v2.router import WagtailAPIRouter

api_router = WagtailAPIRouter('wagtailapi')

api_router.register_endpoint('pages', PagesAPIEndpoint)
api_router.register_endpoint('page-changes', PageChangesAPIEndpoint)

```

### Basic usage

Performing a GET request will return a list of modifications with the oldest
first. If two modifications were created at exactly the same time, the ID
will be used to order them.

```
GET /api/v2/pages/changes/
```

```json
{
    "meta": {
        "total_count": 10,
        "next": "http://example.com/api/v2/page-changes/?after=2018-07-12T16:03:00Z/1"
    },
    "items": [
        {
            "time": "2018-07-12T16:03:00Z",
            "id": 1,
            "kind": "publish",
            "data": {
                "id": 1,
                "meta": {
                    "type": "home.HomePage",
                    ...
                },
                "title": "Home",
                ...
            }
        }
    ]
}
```

### Pagination

There's no defined number of items that each response should return. It's
possible that responses may be filtered in flight so the number of items
in each response may be different or even zero.

In order to fetch all updates, the client must keep following the "next" URL
until this URL no longer changes.

### Modifications

There are two of modification: ``publish`` and ``unpublish``. For each page,
only the latest modification will be returned.

#### ``publish``

This modification kind is used when the page has been published for both the
initial publish and subsequent publishes. The full detail representation of the
pages latest content is nested in the modification in the ``data`` property.

Note: When a live page is moved, this will also count as a modification in the
changes API.

```json
{
    "time": "2018-07-12T16:03:00Z",
    "id": 1,
    "kind": "publish",
    "data": {
        "id": 1,
        "meta": {
            "type": "home.HomePage",
            ...
        },
        "title": "Home",
        ...
    }
}
```

#### ``unpublish``

This modification kind is used when the page has been unpublished.

```json
{
    "time": "2018-07-12T16:03:00Z",
    "id": 1,
    "kind": "unpublish",
}
```

### URL parameters

#### ``?after=<date>[/<id>]``

This limits the results to only show modifications after the specified date/id.
The date must be ISO 8601 format.
