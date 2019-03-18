# RFC: Redirects changes

## Models changes

### Prefix redirects

We will add a new type of redirect that matches on prefixes rather than full URLs. The rest of the URL path that's not part of the redirect's ``old_path`` field will be appended to the destination path.

For example, say we have the following prefix redirect:

 - ``/articles`` to ``/news``

When a user then requests, ``/articles/the-article/`` this will be matched by the prefix redirect and the user will be redirected to ``/news/the-article/``.

To allow creating redirects of this type, a new "Match prefix" checkbox/boolean field will be added to redirects.

#### Choosing the most specific redirect

If multiple prefix redirects and/or a single redirect matches the same URL, the most specific redirect is used.

For example, say we have the following redirects:

 - prefix redirect from ``/articles`` to ``/news``
 - prefix redirect from ``/articles/events`` to ``/events``
 - direct redirect from ``/articles/events/christmas`` to ``/christmas``

Requests will be redirected as follows:

 - ``/articles/events/christmas/`` to ``/christmas/``
 - ``/articles/events/other-event/`` to ``/events/other-event/``
 - ``/articles/the-article/`` to ``/news/the-article/``

## Admin changes

### Adding new redirects automatically

#### On page move

We will add a new checkbox to the page move UI which asks the user if they would like to create a redirect from the old location to the new one.

![](https://d2mxuefqeaa7sj.cloudfront.net/s_C6D086527C63F45E9EA73587C2533A89CBE7313C89FA1DC0882B73424CAB09BB_1552329395110_Screenshot_2019-03-11+Wagtail+-+Move+Blog1.png)

If this is checked, a redirect to the page will be created from the previous URL of the page. If the page has any children, this redirect will be a prefix redirect.

#### On page slug change

When a page’s slug is changed, a checkbox will appear underneath the slug field asking the user if they would like to create a redirect from the old slug.

If this is checked when the page is published with the new slug, a redirect to the page will be created from the previous URL of the page. If the page has any children, this redirect will be a prefix redirect.

**Q: What if the page is saved as draft? The redirect cannot be created yet but the user who publishes it may not be aware that there is a checkbox on the promote tab.**

#### When unpublishing or deleting a page

When unpublishing or deleting a page, the user can add a redirect from that page’s URL to another page. If they create one, they will be given the option to repoint existing redirects at the new page, delete the existing redirects, or (when not deleting) do nothing.

### Redirect management

#### Searching for prefix redirects

The search will return any prefix redirect that matched the searched path. If multiple redirects match, the results will be displayed in order of specificity with most specific first.

#### Management from the page explorer interface

![](https://d2mxuefqeaa7sj.cloudfront.net/s_C6D086527C63F45E9EA73587C2533A89CBE7313C89FA1DC0882B73424CAB09BB_1552329309392_Screenshot_2019-03-11+Wagtail+-+Exploring+Welcome+to+the+Wagtail+Bakery+1.png)

We will add a “redirects” icon at the top right of the page explorer. This will indicate the number of redirects where their “from path” is from within this section. Clicking this will link the user to the redirect management which would be filtered to show only redirects from this section. This button will only be visible to users who have permission to manage redirects.

We will also add a new “Add redirect here” button in the “more” menu. This takes the user to the add redirect UI with the prefix pre-filled (and undeletable).

An alternative to this would be to create a “RedirectPage” type which is a page type that acts as a redirect when served, but these aren’t as flexible and are more heavier to handle in the database than redirects are.

#### Bulk actions

Removing many redirects at once is quite tedious. This will be more of a problem when we start adding new ways to add redirects.

We should improve the redirect search to allow filtering by prefix, and add a bulk delete action which can use checkboxes by each row or all redirects that match the current query.

## Drawbacks

### Redirect chains and loops

It's possible with both prefix and direct redirects to create "redirect chains" where one redirect just leads to another redirect. These are much more likely to happen after we introduce the automatic creation of prefix redirects. For example:

Say we have a section within a section: ``/articles/events``

If the user renames ``articles`` to ``news``, then moves ``events`` out into its own section, this will create two prefix redirects:

 - ``/articles`` to ``/news``
 - ``/news/events`` to ``/events``

A request to ``/articles/events/christmas/`` would be redirected to ``/news/events/christmas/`` then redirected again to ``/events/christmas/``.

No attempt will be made to resolve redirect chains/loops on the server-side as this depends on the knowledge that the each step of the redirect chain will result in a 404. While we can figure out if a URL points to a Wagtail page, it's impossible to know for sure that the redirect URL will result in a 404, as there may be URLs configured in the web server that Wagtail does not know about.

The same goes for redirect loops, we will rely on web browsers to limit the number of redirects the user receives to break out of any loops.
