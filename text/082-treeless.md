# RFC 82: Treeless page listings

* RFC: 82
* Author: Matthew Westcott
* Created: 2023-01-18
* Last Modified: 2023-01-18

## Abstract

In many situations, the standard tree-based page explorer is not the ideal interface for editors to find pages of interest. For example, a newspaper site might organise its articles into sections by topic - if an editor works across multiple topics and wants to find a draft article they were working on yesterday, they would probably prefer a flat listing of all their articles ordered by recency, rather than having to remember which topic the article was filed under and seek it out in that section.

This RFC proposes a mechanism for setting up a listing view in the Wagtail admin that displays all pages matching certain criteria, as a flat list. This will be similar to the index view provided by the ModelAdmin module when used with a page model, but will not be restricted to a single page type, and will provide more flexible filtering options.

## Specification

A `register_page_listing` function will be provided, similar to the existing `register_snippet` and `modeladmin_register` functions, which is passed a class containing configuration options and creates a listing view according to those options. Typically this would be placed in a `wagtail_hooks.py` file and look something like this:

```python
from wagtail.admin.views.page_listing import PageListing, register_page_listing

class ArticlePageListing(PageListing):
    # The page model to be listed; defaults to Page (i.e. list all page types)
    model = ArticlePage

    # Label to use for menu item and page title - defaults to the model's verbose_name_plural,
    # or "All pages" if model is Page
    label = "Articles"

    # Where the menu item will be placed in the menu
    menu_order = 800

    # Icon to use for menu item and page heading - defaults to a standard page icon
    icon = "news"

    # URL prefix under `/admin/` where the view will be available - defaults to a slugified
    # version of the label
    url_prefix = "articles"


register_page_listing(ArticlePageListing)
```

This will register a menu item in the main admin menu, and a listing view at the given URL.

The listing view will show a paginated listing of all pages of the given type for which the logged-in user has any permission (edit, publish, lock/unlock). The listing will consist of the columns:

* (checkbox for bulk actions)
* Title
* Parent page
* Last updated
* Page type
* Status (live / draft / scheduled)

Additional columns can be configured by the developer through the `PageListing` definition; if a specific page type has been specified for `model`, all fields specific to that page type will be available in addition to the core Page fields.

For each page in the listing, a set of action buttons will be provided similar to the existing tree-based view: "Edit", "View live", and a "More" menu containing all actions except "Sort menu order". ("Add child page" and "Sort menu order" have been omitted here as these are only really meaningful when the user is working from a mental model of the tree structure, which has been expressly hidden here.) When one of these actions is followed, the appropriate 'return' URL variable will be passed (where supported) so that on completion of the action, the user is returned to the treeless view.

Initially the listing is ordered by most recently updated first, but this can be changed by clicking on the column headings.

Alongside the listing, various filtering options are available - these will immediately refresh the listing on change:

* Search term
* Page type (if the listing is not already restricted to a single page type)
* Status
* Last updated (start/end date range)
* Locale
* Page owner
* Site (if more than one site record exists and user has permission for them)

Again, the developer can configure additional filters, and if a specific page type has been specified for `model`, these filters can apply to fields specific to that page type as well as the core Page fields. Filters will be implemented through the `django-filters` package, as currently used on reports.

### Add page

The listing view will also include a button in the page header for adding a new page. Since we need to know both the page type and parent page before we can go ahead and add a new page, we will need a suitably streamlined user interface for gathering this information.

By considering the user's page permissions in conjunction with the `parent_page_types` / `subpage_types` business rules, we can arrive at some set of page types that are available for the user to add. (Additionally, if the listing view has been set up for a specific page type via the `model` setting, only that page type and its subclasses will be offered as options.) It's expected that site owners making use of treeless views will keep permissions and `parent_page_types` / `subpage_types` rules appropriately tight so that this set of available page types remains at a manageable size - say, less than 10. With this in mind, I would suggest that the "Add page" button opens up a dropdown where the page type can be selected, eliminating the need for another page load on this step.

If the chosen page type only has one valid place where it can be added, we can immediately proceed to the page creation form; otherwise, the user is presented with a list of available parent pages. (This is the same workflow currently offered by ModelAdmin for creating a new page.) Again, it's expected that a well-constructed site will keep this list to a manageable size, but it could potentially be any size (e.g. on a site with hundreds of ArticlePages, and no restriction on creating new ArticlePages as children of other ArticlePages). I suggest that if this list exceeds a certain threshold (say 30), we provide the user with a page chooser widget to select the desired parent page (as we do for "Move page") rather than listing the options.

On choosing a parent page, the user is taken to the page creation form.

## Open Questions

// Include any questions until Status is ‘Accepted’
