# RFC 58: Alias pages

* Authors: Karl Hobley
* Created: 2020-08-04
* Last Modified: 2020-08-04

## Summary

This RFC contains a specification of a new type of page which I call “Alias pages”.

Alias pages are exact copies of pages that live in another part of the tree. They are linked in the database and kept in sync with their source page. In the admin UI, they are displayed differently to make it clear they are an alias of another page.

Aliases pages are not editable directly, and they do not have any revisions or logging. Aliases can either be created by translating a page (where the page exists in another language with the exact same content) or copying a page (where the page needs to appear in another part of the tree or on another site in the same language).

## Motivation

### Partial/incremental translation

The main motivation for me suggesting this feature right now is implementing two features for internationalisation:

* Partial translation - the ability to translate part of your page tree, but not all of it. We need to copy all other pages in order for menus to be populated and allow the user to navigate the rest of the site without switching to another language tree.
* Incremental translation - allow a site to be translated piecemeal without any broken pages.

### Aliasing pages without customising routing

Aliasing is already common on Wagtail sites. For example, RCA show is a section of RCA’s website that contains aliases for all the student pages of a particular year.

Here’s an example page: [https://www.rca.ac.uk/showcase/show-2019/schoolofdesign/intelligent-mobility/pei-chan/](https://www.rca.ac.uk/showcase/show-2019/schoolofdesign/intelligent-mobility/pei-chan/)
Which is an alias of: [https://www.rca.ac.uk/students/pei-chan/](https://www.rca.ac.uk/students/pei-chan/)

This was implemented by customising the routing on the show index page to make it respond on sub-urls that match a student slug for that year and serve that student with a different template.

The advantages of “Alias pages” over customising the routing are:

* Cache invalidation is much simpler as Wagtail knows all the URLs the page will respond on
* Internal page links to aliases are possible (you have to use an external link to link to a custom route on a page)
* Should work nicely with headless sites (the `?html_path` filter isn’t able to resolve a custom route)
* We can add them to sitemaps and deal with them properly in search

## Detailed list of changes

### Core changes

**`alias_of` field**

We will a new field called `alias_of` to `wagtailcore.Page` which has the following definition:

`models.ForeignKey('self', on_delete=models.SET_NULL, null=True, related_name='aliases')`

If is field is `None` then this page is not an alias, otherwise the page is an alias and this field contains a link to the page that this page is an alias of. The reverse accessor `aliases` can be used to retrieve aliases of the page.

Constraints (checked on save and with `fixtree`):

* Aliases cannot have their own aliases
* Aliases must have the same page type as their original page
* Aliases must be one of:
  * A copy of a page in the same locale (shares `locale` with the source)
  * A translation of a page (shares `translation_key` with the source)

**`alias` parameter on `copy` and `copy_for_translation`**

When a page is copied with the `alias` parameter set to `True`, the copied pages will be created as aliases of the page(s) they were copied from.

If either of these methods are called on an alias, the `alias_of` field will be set to the source page of the current page.

#### Update alias pages on publish

When pages are published, we will check if they have any aliases and update them too. Aliases will be updated in bulk, which is possible to do as they don’t have revisions.

For regular fields, we will perform a single update query to update the aliases.

For child objects, we will perform two queries per child relation. The first one will delete all child from all the aliases, the second query will bulk create new child objects.

#### Unpublish aliases

When the source is unpublished, the aliases are all unpublished as well.

#### Unlink aliases on delete

If the source page is deleted, the aliases are unlinked, but not deleted.

Q: Should we allow deleting aliases as an option?

### Admin changes

#### Alias pages in page explorer

Alias pages will be displayed differently in the React explorer menu and the page explorer listings.

* In the react explorer menu, we would grey out the title of the alias page
* In the page listing, we will display “Alias of \<aliased page title\>”, where \<aliased page title\> would be a link if it’s editable by the current user

#### Alias pages in admin search

Alias pages will be excluded from admin search

#### Editing an alias page

It isn’t possible to get to the editor of an alias. All edit links will direct the user to the source page. This includes on the explorer and in the Wagtail edit bird.
