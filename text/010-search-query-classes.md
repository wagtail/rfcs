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

The following classes will be created:

### ``PlainTextQuery(query_string, fields=[])``

This is the default query type which will be used if a string is passed to the
 the ``.search()`` method. So I don't expect this class to be used directly,
but it is provided for completeness.

Example:

```python
from wagtail.wagtailsearch.query import PlainTextQuery

>>> Page.objects.search(PlainTextQuery("Hello world"))
[<Page: Hello world>]

# Simple version
>>> Page.objects.search("Hello world")
[<Page: Hello world>]
```

### ``MatchAllQuery()``

This query type matches all items in the index, replacing the current,
confusing ``None`` behaviour.

Example:

```python
from wagtail.wagtailsearch.query import MatchAllQuery

>>> Page.objects.search(MatchAllQuery())
[<lots of pages>]
```

### Future enhancements

To keep this RFC focused on the basic syntax, the following query types are out
of scope. I've included them here to show, potentially, what this enhancement
could lead to in the future.

#### ``QueryStringQuery(query_string, search_fields=[], filter_fields[])``

This query type is similar to ``PlainTextQuery`` but it has syntax for
filtering and operators.

Example:

```python
from wagtail.wagtailsearch.query import QueryStringQuery

>>> Page.objects.search(QueryStringQuery("live:true -author:karl Hello world", filter_fields=['author', 'live']))
[<Page: Hello world>]
```

#### ``SimilarItemsQuery(object, fields=[])``

NOTE: I'm not 100% sure of the name of this one

This query type performs a ``more_like_this`` query to find similar objects
to the one specified.

Example:

```python
from wagtail.wagtailsearch.query import SimilarItemsQuery

>>> Page.objects.search(SimilarItemsQuery(hello_page, fields=['title']))
[<Page: Hello world>]
```
