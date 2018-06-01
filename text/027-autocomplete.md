# RFC 27: Search autocomplete API

* RFC: 27
* Author: Karl Hobley
* Status: Draft
* Created: 2018-06-01
* Last Modified: 2018-06-01

## Abstract

This RFC proposes an overhaul of the autocomplete (aka "partial match") API for
Wagtail search.

# Current status

Currently, partial matching is automatically performed by the `.search()` API
on all `SearchField` s that have `partial_match=True`. There are some problems
with this approach:

## Lack of control

Without hacks, it’s not possible to switch partial matching off for a search
query. So the only way to disable partial matching is by removing
`partial_match=True` from your `search_fields` configuration.

In general, “partial match” It’s only useful for showing suggestions while the
user is typing their query. Once they’ve finished typing their query and they
hit enter, the search that is performed shouldn’t use partial matching.

## Magic

Both partial matching and regular searching are performed together in the same
query. It’s not clear to the developer how this affects ranking.

## Inaccurate search results

All words in search queries are treated as prefixes. For example, if your
search query contains the word “up”, this would match “upper”, “upset”,
“upgrade”, etc and incorrectly rank those items.

## Hard to implement for PostgreSQL

TODO

# Specification

We propose to remove the partial match feature from the `.search()` API and
replace it with a separate `.autocomplete()` API.

## Configuration

The `partial_match` argument will be replaced with a new field type called
`AutocompleteField`.

Fields that require autocomplete will need to configure the `AutocompleteField`
in addition to any other field types they are using. For example:

```python
class Book(index.Indexed, models.Model):
    title = models.CharField(max_length=255)
    description = models.TextField()

    search_fields = [
        index.SearchField('title'),
        index.AutocompleteField('title'),
        index.SearchField('description'),
    ]
```

## Usage

The existing `.search()` API will no longer do partial matching. Instead, a new
`.autocomplete()` API will be added which can be called on the backend or on
page/image/document `QuerySet`s.

Like `.search()`, this will support filtering through the `QuerySet` and will
return a `SearchResults` object. Unlike `.search()` this will not support
search query classes (but support for that may be added in the future).

For example:

```python
>>> Book.objects.autocomplete('The lor')
<SearchResults [<Book "The Lord of the Rings">]>

# This will no longer partial match
>>> Book.objects.search('The lor')
<SearchResults []>
```

# Deprecation path

The purpose of introducing the `.autocomplete()` API is so that we can fix the
problems with the `.search()` API. But this API is very commonly used so we
should change the behaviour considerately and make it clear to developers how
they should update their code.

As per Wagtail’s deprecation policy, the partial match behaviour should still
work for two releases after introducing the `.autocomplete()` API. It should
be possible for developers to completely disable partial match once they’ve
updated their code

Developers could disable partial match by removing all the `partial_match=True`
from `SearchField`s they’ve defined. But for page models this is tricky as
search configuration of the “title” field is within Wagtail itself, and we
can’t just remove it here as that’s a breaking change.

Instead, I think we should allow developers to opt-out of partial matching
through the `.search()` API itself and raise a deprecation warning whenever the
developer hasn’t opted out. This should provide an impulse for developers to
consider whether they need to switch to the new `.autocomplete()` API.

Search code should be updated in one of two ways:

```python
>>> Page.objects.search("foo")
# Deprecation warning raised. Asking the developer to update their code

# 1. I don't want partial matching. opt-in to the new behaviour
>>> Page.objects.search("foo", partial_match=False)

# 2. I want to keep partial matching. Switch to the autocomplete API
>>> Page.objects.autocomplete("foo")
```

Once partial match is removed, we then deprecate the `partial_match` argument.

This is the proposed deprecation timeline:

**Wagtail 2.2**

- Introduce `.autocomplete()` API
- Deprecate `SearchField(partial_match=True)` (`Page.title` should still have
  this)
- Deprecate `.search()` without `partial_match=False`

**Wagtail 2.4**

- Remove `SearchField(partial_match=True)` (can be disabled on `Page.title`
  now)
- Remove all partial matching logic from `.search()` API
- Deprecate `partial_match` on `.search()` API
