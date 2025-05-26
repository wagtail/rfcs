# RFC 34: Automatic redirects

This RFC contains a proposal to implement automatic creation of redirects and a couple of improvements to managing redirects in the admin interface.

## Motivation

Currently, if the user changes a page's slug, or moves a page, the URL of that page will be changed and any requests going to the previous URL may result in a 404.

This is a highly undesirable outcome as links to those pages would be broken, and it's to automatically correct this if we haven't recorded anywhere where that page went. The burden is on the end user to figure out where the content they are looking for has gone, but this can sometimes be difficult if they are unfamiliar with the site and it's unlikely they have the patience to do that.

In order to prevent this from happening, editors must manually create redirects for pages that they move or change the slug of on their site, and repeat this for any child pages. This could be a tedious task and may be forgotten.

This is a task that can be done automatically by Wagtail.

## Prior art

The [``wagtail-automatic-redirects``](https://github.com/themotleyfool/wagtail-automatic-redirects/) package, created by The Motley Fool, currently provides the "[Automatic creation of redirects when pages URL changes](#automatic-creation-of-redirects-when-pages-url-changes)" feature in an installable package.

## Specification

### ``WAGTAIL_AUTOMATIC_REDIRECTS_ENABLED`` setting

The automatic redirects feature would be enabled with a Django setting:

```python
WAGTAIL_AUTOMATIC_REDIRECTS_ENABLED = True
```

This setting will be ``False`` by default so that existing sites can opt-in to it.
This setting would be added to the project template and release notes to encourage as many developers as possible to use it.

### Automatic creation of redirects when pages URL changes

When the feature is enabled, Wagtail will automatically create a redirect when a page's URL changes.

This can happen when:

 - The page is moved
 - Its slug is changed
 - Either of those two events occur on an ancestor page

The redirect will be created regardless of the user's permissions to add or manage redirects.

#### Should editors be allowed to choose when they create redirects?

We couldn't think of a case why you wouldn't want to create a redirect when a page's URL changes.
Redirects will only take effect when there is no page currently live at the requested URL so having extra redirects will not break anything.

### Automatic creation of redirects on page deletion

When the feature is enabled, a new page chooser field (optional) would be displayed on the "delete page confirm" view.
This allows users to pick a destination page to redirect any users visiting the old URLs to.

If they are deleting a tree, a redirect will be created for every page in the tree, but they will all point to the same destination.

### Redirects listing filters

The redirects listing would now include three new filters on the right hand side:

 - "Redirect type" (Choice) - Filters the redirects listing by redirect type (All / Internal / External)
 - "Page" (Page chooser) - When a page is chosen, the redirects will be filtered to include only those that point at the chosen page
 - "Hide automatically created redirects" (Checkbox) - When checked, any redirects that were automatically created (and not subsequently edited later) would be hidden

### Redirects panel on promote tab

On the promote tab on all pages, we will display the number of redirects that link to the page. This number would be a link to the redirects listing filtered to the current page.

This is only displayed to users who have permission to see the redirects listing.
