# RFC 25: Search query classes (revised)

* RFC: 25
* Author: Karl Hobley
* Created: 2018-05-08
* Last-Modified: 2018-05-08

## Abstract

In RFC 10 we introduced the idea of "Search Query Classes", a way to allow
Wagtail to support more complex querying methods.

This RFC is a replaces that previous RFC.

## Specification

I propose adding a new set of classes to Wagtail which represent different
types of search query that could be run. Instances of these classes can be
passed as the first argument of the search method.

These classes will be importable from ``wagtail.wagtailsearch.query``.

"queries" are only responsible for selecting and scoring results, so
features like autocomplete, faceting and highlighting are out of scope for
this API.

### Standard query types

#### ``PlainText(query_string, operator=None, boost=1.0)``

This is the default query type which will be used if a string is passed to the
 the ``.search()`` method. So I don't expect this class to be used directly,
but it is provided for completeness.

If fields or operator are not specified, they will fall back to the values
passed in to the ``search()`` method. Failing that, they will fall back to
the default values specified by the backend.

```python
from wagtail.wagtailsearch.query import PlainText

>>> Page.objects.search(PlainText("Hello world"))
[<Page: Hello world>]

# Simple version
>>> Page.objects.search("Hello world")
[<Page: Hello world>]
```

#### ``MatchAll()``

This query type matches all items in the index, replacing the current,
confusing ``None`` behaviour.

```python
from wagtail.wagtailsearch.query import MatchAll

>>> Page.objects.search(MatchAll())
[<lots of pages>]
```

### Combinators

Queries can be combined with logical operators ``and``, ``or`` and ``not`` and
all of the query classes will support being combined with ``&``, ``|`` and ``~``
operators respectively. These operators will wrap the operands with one of the
following combinator query classes:

#### ``And(subqueries)``

Combines the subqueries with the and operator. This performs an intersection
of their result sets and performs the specified score function to combine the
scores of each subquery.

```python
from wagtail.wagtailsearch.query import PlainText, And

>>> Page.objects.search(PlainText("Hello") & PlainText("world"))
[<Page: Hello world>]

# This is equivalent to:
>>> Page.objects.search(And([PlainText("Hello"), PlainText("world")]))
[<Page: Hello world>]
```

#### ``Or(subqueries)``

Combines the subqueries with the and operator. This performs a union of their
result sets and performs the specified score function to combine the scores of
each subquery.

```python
from wagtail.wagtailsearch.query import PlainText, Or

>>> Page.objects.search(PlainText("Hello") | PlainText("world"))
[<Page: Hello world>, <Page: Hello everyone>]

# This is equivalent to:
>>> Page.objects.search(Or([PlainText("Hello"), PlainText("world")]))
[<Page: Hello world>, <Page: Hello everyone>]
```

#### ``Not(subquery)``

Returns all results that do not match the given query. All results are initially
given the specified score.

```python
from wagtail.wagtailsearch.query import PlainText, Not

>>> Page.objects.search(~PlainText("Hello"))
[<Page: Goodbye>]

# This is equivalent to:
>>> Page.objects.search(Not(PlainText("Hello")))
[<Page: Goodbye>]
```

### Changes from RFC 10

#### Removal of "Low level query types"

After attempting to implement them, we found that it is difficult to make
them behave consistently across all search backends. We also decided that
they didn't add anything useful to the API that can't be achieved with
``PlainText`` queries.

#### Removal of "Filter" query type

It's possible to implement the same functionality using the combinator
types so it's not very useful to have a separate class for this.

#### Removal of "future enhancements"

New query types should be specified in future RFCs.
