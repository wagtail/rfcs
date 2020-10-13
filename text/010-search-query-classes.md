# RFC 10: Search query classes (superseded by [RFC 25](025-search-query-classes-revised.md))

* RFC: 10
* Author: Karl Hobley
* Created: 2016-06-22
* Last-Modified: 2017-08-24

## Abstract

Wagtail's search module currently only supports simple plain text queries.
There are many other ways to search such as "more like this" or more advanced
query string syntaxes.

This RFC proposes an API for supporting new types of query in Wagtail.

## What we have now

The first argument to the ``.search()`` method (on both the search backend and
on QuerySets) currently either takes a plain-text search query (as a string) or
``None`` (which triggers Wagtail to run a ``match_all`` query).

The ``None`` argument was implemented to allow testing of running filters
against Elasticsearch and it's not been very useful in actual sites. It is
also quite confusing that passing ``None`` as the search query returns all
items in the index but passing a blank string returns nothing.

## Specification

I propose adding a new set of classes to Wagtail which represent different
types of search query that could be run. Instances of these classes can be
passed as the first argument of the search method.

These classes will be importable from ``wagtail.wagtailsearch.query``.

The query classes are only responsible for selecting and scoring results, so
features like autocomplete, faceting, hightlighting, etc are out of scope for
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

### Low level query types

These query types allow developers to build up queries from individual terms.

#### ``Term(term, boost=1.0)``

Matches a term by exact value.

#### ``Prefix(prefix, boost=1.0)``

Matches any term with the specified prefix.

#### ``Fuzzy(term, max_distance=3, boost=1.0)``

Matches any term within the specified Levenshtein distance of the specified term.

#### ``Boost(query, boost)``

Multiplies the boost of all subqueries by the specified value.

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
from wagtail.wagtailsearch.query import Term, And

>>> Page.objects.search(Term("Hello") & Term("world"))
[<Page: Hello world>]

# This is equivalent to:
>>> Page.objects.search(And([Term("Hello"), Term("world")]))
[<Page: Hello world>]
```

#### ``Or(subqueries)``

Combines the subqueries with the and operator. This performs a union of their
result sets and performs the specified score function to combine the scores of
each subquery.

```python
from wagtail.wagtailsearch.query import Term, Or

>>> Page.objects.search(Term("Hello") | Term("world"))
[<Page: Hello world>, <Page: Hello everyone>]

# This is equivalent to:
>>> Page.objects.search(Or([Term("Hello"), Term("world")]))
[<Page: Hello world>, <Page: Hello everyone>]
```

#### ``Not(subquery)``

Returns all results that do not match the given query. All results are initially
given the specified score.

```python
from wagtail.wagtailsearch.query import Term, Not

>>> Page.objects.search(~Term("Hello"))
[<Page: Goodbye>]

# This is equivalent to:
>>> Page.objects.search(Not(Term("Hello")))
[<Page: Goodbye>]
```

#### ``Filter(query, include=None, exclude=None)``

Takes the results and scores from ``query`` and filters them to remove any
results that don't match ``include`` or do match ``exclude``.

```python
from wagtail.wagtailsearch.query import Term, Filter

# Removes any documents that match "World" from the query for "Hello"
>>> Page.objects.search(Filter(Term("Hello"), exclude=Term("World")))
[<Page: Hello>]

# Similar to And(["Hello", "World"]) except for the score is only taken from "Hello"
>>> Page.objects.search(Filter(Term("Hello"), include=Term("World")))
[<Page: Hello World>]
```

### Future enhancements

To keep this RFC focused on the basic syntax, the following query types are out
of scope. I've included them here to show, potentially, what this enhancement
could lead to in the future.

#### ``QueryString(query_string, allowed_filter_fields[])``

This query type is similar to ``PlainText`` but it has syntax for
filtering and operators.

```python
from wagtail.wagtailsearch.query import QueryString

>>> Page.objects.search(QueryString("live:true -author:karl Hello world", filter_fields=['author', 'live']))
[<Page: Hello world>]
```

#### ``SimilarItems(object, fields=[])``

NOTE: I'm not 100% sure of the name of this one

This query type performs a ``more_like_this`` query to find similar objects
to the one specified.

```python
from wagtail.wagtailsearch.query import SimilarItems

>>> Page.objects.search(SimilarItems(hello_page, fields=['title']))
[<Page: Hello world>]
```
