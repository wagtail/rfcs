# RFC 10: Search query classes

* RFC: 10
* Author: Karl Hobley
* Created: 2016-06-22
* Last-Modified: 2016-06-22

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

#### ``PlainTextQuery(query_string, fields=[])``

This is the default query type which will be used if a string is passed to the
 the ``.search()`` method. So I don't expect this class to be used directly,
but it is provided for completeness.

```python
from wagtail.wagtailsearch.query import PlainTextQuery

>>> Page.objects.search(PlainTextQuery("Hello world"))
[<Page: Hello world>]

# Simple version
>>> Page.objects.search("Hello world")
[<Page: Hello world>]
```

#### ``MatchAllQuery()``

This query type matches all items in the index, replacing the current,
confusing ``None`` behaviour.

```python
from wagtail.wagtailsearch.query import MatchAllQuery

>>> Page.objects.search(MatchAllQuery())
[<lots of pages>]
```

### Low level query types

These query types allow developers to build up queries from individual terms.

#### ``TermQuery(term, fields=[])``

Matches if the specified term is in one of the specified fields.

#### ``PrefixQuery(prefix, fields=[])``

Matches if a term exists within one of the specified fields with the prefix.

#### ``FuzzyQuery(term, max_distance=3, fields=[])``

Matches if a term like the specified exists within one of the specified fields.

The distance is the Levenshtein distance to the term.

### Combinators

Queries can be combined with logical operators ``and``, ``or`` and ``not`` and
all of the query classes will support being combined with ``&``, ``|`` and ``~``
operators respectively. These operators will wrap the operands with one of the
following combinator query classes:

#### ``AndQuery(subqueries, score_function='avg')``

Combines the two queries with the and operator. This performs an intersection
of their result sets and performs the specified score function to combine the
scores of each subquery.

Valid score functions are: ``avg``, ``min``, ``max`` and ``sum``

```python
from wagtail.wagtailsearch.query import TermQuery, AndQuery

>>> Page.objects.search(TermQuery("Hello") & TermQuery("world"))
[<Page: Hello world>]

# This is equivilant to:
>>> Page.objects.search(AndQuery([TermQuery("Hello"), TermQuery("world")]))
[<Page: Hello world>]
```

#### ``OrQuery(subqueries, score_function='avg')``

Combines the two queries with the and operator. This performs a union of their
result sets and performs the specified score function to combine the scores of
each subquery.

Valid score functions are: ``avg``, ``min``, ``max`` and ``sum``

```python
from wagtail.wagtailsearch.query import TermQuery, OrQuery

>>> Page.objects.search(TermQuery("Hello") | TermQuery("world"))
[<Page: Hello world>, <Page: Hello everyone>]

# This is equivilant to:
>>> Page.objects.search(OrQuery([TermQuery("Hello"), TermQuery("world")]))
[<Page: Hello world>, <Page: Hello everyone>]
```

#### ``NotQuery(subquery, score=1.0)``

Returns all results that do not match the given query. All results are initially
given the specified score.

```python
from wagtail.wagtailsearch.query import TermQuery, NotQuery

>>> Page.objects.search(~TermQuery("Hello"))
[<Page: Goodbye>]

# This is equivilant to:
>>> Page.objects.search(NotQuery(TermQuery("Hello")))
[<Page: Goodbye>]
```

#### ``FilterQuery(subquery, include=None, exclude=None)``

Takes the results and scores from ``subquery`` and filters them to remove any
results that don't match ``include`` or do match ``exclude``.

```python
from wagtail.wagtailsearch.query import TermQuery, FilterQuery

# Removes any documents that match "World" from the query for "Hello"
>>> Page.objects.search(FilterQuery(TermQuery("Hello"), exclude=TermQuery("World")))
[<Page: Hello>]

# Similar to AndQuery(["Hello", "World"]) except for the score is only taken from "Hello"
>>> Page.objects.search(FilterQuery(TermQuery("Hello"), include=TermQuery("World")))
[<Page: Hello World>]
```

### Future enhancements

To keep this RFC focused on the basic syntax, the following query types are out
of scope. I've included them here to show, potentially, what this enhancement
could lead to in the future.

#### ``QueryStringQuery(query_string, search_fields=[], filter_fields[])``

This query type is similar to ``PlainTextQuery`` but it has syntax for
filtering and operators.

```python
from wagtail.wagtailsearch.query import QueryStringQuery

>>> Page.objects.search(QueryStringQuery("live:true -author:karl Hello world", filter_fields=['author', 'live']))
[<Page: Hello world>]
```

#### ``SimilarItemsQuery(object, fields=[])``

NOTE: I'm not 100% sure of the name of this one

This query type performs a ``more_like_this`` query to find similar objects
to the one specified.

```python
from wagtail.wagtailsearch.query import SimilarItemsQuery

>>> Page.objects.search(SimilarItemsQuery(hello_page, fields=['title']))
[<Page: Hello world>]
```
