# RFC: Redirects admin changes

# Adding new redirects automatically
## On page move

We will add a new checkbox to the page move UI which asks the user if they would like to create a redirect from the old location to the new one.

![](https://d2mxuefqeaa7sj.cloudfront.net/s_C6D086527C63F45E9EA73587C2533A89CBE7313C89FA1DC0882B73424CAB09BB_1552329395110_Screenshot_2019-03-11+Wagtail+-+Move+Blog1.png)


If they check this, and the page has subpages, they will be shown another checkbox asking them if they want to create recursive redirects for every descendant page.

![](https://d2mxuefqeaa7sj.cloudfront.net/s_C6D086527C63F45E9EA73587C2533A89CBE7313C89FA1DC0882B73424CAB09BB_1552329038086_Screenshot_2019-03-11+Wagtail+-+Move+Blog2.png)

## On page slug change

When a page’s slug is changed, a checkbox will appear underneath the slug field asking the user if they would like to create a redirect from the old slug.

If they check this box and their page has subpages, they will be given another checkbox asking them if they would like to create recursive redirects for every descendant page as well.

## When unpublishing or deleting a page

When unpublishing or deleting a page, the user can add a redirect from that page’s URL to another page. If they create one, they will be given the option to repoint existing redirects at the new page, delete the existing redirects, or (when not deleting) do nothing.

## Deleting redirects when pages are created in their place

Whenever a page is created, moved, or re-slugged in a way that causes it or one of its descendants to clash with a redirect. That or those redirects should be deleted. The user should always be given a warning if this is going to happen, with an option of overriding it if they want to keep them.

# Redirect management
## Management from the page explorer interface
![](https://d2mxuefqeaa7sj.cloudfront.net/s_C6D086527C63F45E9EA73587C2533A89CBE7313C89FA1DC0882B73424CAB09BB_1552329309392_Screenshot_2019-03-11+Wagtail+-+Exploring+Welcome+to+the+Wagtail+Bakery+1.png)


We will add a “redirects” icon at the top right of the page explorer. This will indicate the number of redirects where their “from path” is from within this section. Clicking this will link the user to the redirect management which would be filtered to show only redirects from this section. This button will only be visible to users who have permission to manage redirects.

We will also add a new “Add redirect here” button in the “more” menu. This takes the user to the add redirect UI with the prefix pre-filled (and undeletable).

An alternative to this would be to create a “RedirectPage” type which is a page type that acts as a redirect when served, but these aren’t as flexible and are more heavier to handle in the database than redirects are.

## Bulk actions

Removing many redirects at once is quite tedious. This will be more of a problem when we start adding new ways to add redirects.

We should improve the redirect search to allow filtering by prefix, and add a bulk delete action which can use checkboxes by each row or all redirects that match the current query.

