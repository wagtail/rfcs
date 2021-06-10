# RFC 34: Automatic redirects

A commonly requested feature for Wagtail is automatically creating redirects to pages on certain events such as moving, changing slug, and deleting a page.

This RFC contains a proposal to add this feature into Wagtail.

## Motivation

Currently, if the user changes a page's slug, or moves a page, the URL of that page will be changed and any requests going to the previous URL may result in a 404.

This is a highly undesirable outcome as links to those pages would be broken, and it's to automatically correct this if we haven't recorded anywhere where that page went. The burden is on the end user to figure out where the content they are looking for has gone, but this can sometimes be difficult if they are unfamiliar with the site and it's unlikely they have the patience to do that.

In order to prevent this from happening, editors must manually create redirects for pages that they move or change the slug of on their site, and repeat this for any child pages. This could be a tedious task and may be forgotten.

This is a task that can be quite easily handled by Wagtail. We just need to add a user interface to allow a user to specify if they want redirects to be created on moving pages or changing slugs and Wagtail can do the rest of the work.

## Specification

We will implement automatic creation of redirects in the following places. Note that users must have permission to add redirects in order for any of these interfaces to appear.

### On page move

We will add a new checkbox to the page move UI which asks the user if they would like to create a redirect from the old location to the new one.

![](https://d2mxuefqeaa7sj.cloudfront.net/s_C6D086527C63F45E9EA73587C2533A89CBE7313C89FA1DC0882B73424CAB09BB_1552329395110_Screenshot_2019-03-11+Wagtail+-+Move+Blog1.png)

If this is checked, a redirect to the page will be created from the previous URL of the page. Redirects are also created for all descendants of the page.

### On page slug change

When a page’s slug is changed, a checkbox will appear underneath the slug field asking the user if they would like to create a redirect from the old slug.

If this is checked, a redirect to the page will be created from the previous URL of the page. Redirects are also created for all descendants.

Note that if the page is saved in draft, the redirects are still created but they won't take effect until the slug change is published since redirects only work if there is not already a page at the URL.

### When unpublishing or deleting a page

When unpublishing or deleting a page or page tree, the user will be given the option to add redirects to a page. This will also update any existing redirects pointing at the page being deleted.

Note that if the user is deleting a page tree, the user will only be able to choose a single destination page for all the redirects that get created.
