# RFC 43: Author specific page locking

* RFC: 43
* Authors: Karl Hobley
* Created: 2019-10-08
* Last Modified: 2019-10-11

## Abstract

This RFC proposes some changes we can make to the page locking behaviour to
make it more useful.

Currently, users with the "Lock" permission can lock a page to prevent anyone
(including the user who locked the page) from editing it.

This feature was originally developed for the RCA to allow student pages to be
locked after they have been reviewed by an administrator (this was to prevent
the student from editing the page again before it was published as all student
pages were published on the same day). It has also been used as a way to
prevent editors from editing the index page of section they should only have
permission to add/edit subpages.

The main proposal in this RFC is to change this behaviour so that the user who
locked the page can still edit it. This will allow the locking to be used by
editors to get an exclusive lock on a page they may be drafting changes to
offline for example.

This change shouldn't be a problem for those using this feature for these two
known use cases. However we will provide backwards compability so any uses of
this feature that we're not aware of won't be broken.

## Specification

### Models changes

We will add two new fields to the base `Page` model:

 - `locked_by` a nullable `ForeignKey` to the `AUTH_USER_MODEL`. This person is
   still allowed to edit the page despite it being locked
 - `locked_at` a nullable `DateTimeField` that is set to the time the lock was
    last acquired

The `locked` boolean field will remain on the model and continue to be set to
`True` whenever the page is locked.

Whenever the `locked` field is `False`, the page should not be seen as in any
locked state so the `locked_by` and `locked_at` fields should be disregarded.

If `locked` is `True` but `locked_by` is `None`, this should be treated as a
global lock.

We will add a new setting called `WAGTAILADMIN_GLOBAL_PAGE_EDIT_LOCK`. When
this is set to `True`, Wagtail will not set the `locked_by` field when the page
is locked.

The `locked_at` field will always be set to the time the page was locked,
regardless of the value of the `WAGTAILADMIN_GLOBAL_PAGE_EDIT_LOCK` setting.
But it may be set to `None` on any pages that were locked before this change
was deployed, so any frontend code would need to take that into account.

### Page lock/unlock permission changes

As we're adding a new use for the page locking that would make it useful for
editors as well as administrators, the "Lock" permission may now be added to
editors who didn't previously have "Lock" permission.

This means that we will need to tweak the way the permissions work so that
editors cannot unlock pages that they are not supposed to.

To do this, we will add a separate "Unlock any page" permission, this will
allow users with this permission to unlock pages that are globally locked or
locked by someone else.

When an editor has the "Locked" permission but not "Unlock any page", they can
only lock pages that aren't currently already locked and only unlock pages
that they have locked themselves.

### Page editor UI changes

The existing lock/unlock button will look the same as it does now. Also, when
a page is locked, the editor will look the same for everyone else except for
the locker (with the exception that we can now show who locked the page and
when they locked it).

The main difference is how the editr will look for the user who locked the page.
This user will still be able to edit the page but there needs to be a strong
visual indication that they have an exclusive lock on the page, so they are
unlikely to forget to unlock it.

Q: Possibly, we could automatically unlock the page when the user publishes, or
maybe just prompt to unlock the page somehow when they publish

### New "My locked pages" dashboard panel

We will add a new panel to the admin dashboard called "My locked pages" this
will display all the pages that the user has locked, each item in the list will
have "Edit" and "Unlock" action buttons next to them.

Like the other dashboard panels, this will hide itself when it is empty.

### New "All locked pages" view

Users who have the "Unlock any page" permission will have access to access to a
new "All locked pages" view. This gives view of all the locked pages in sections
where the user has the "Unlock any page" permission for. This will allow site
administrators to find and unlock any pages where the original locker may have
forgotten to unlock them.

The listing will be ordered by `locked_by` (most recent first) have the following
columns:

 - *Page* - The value of this column is the page title that links to the page editor
 - *Locked by* - The name of the user who locked the page
 - *Locked at* - The date/time where the lock was acquired
 - *Last edited at* - The date/time this page was last edited
 - *Unlock* - This column will contain a button to allow quickly unlocking the page

This view will be accessible from a link at the bottom of the "My locked pages"
dashboard panel.

Q: Is the dashboard panel the best place? This panel may not be shown if the
user has not locked any pages.

Q: Should there be an unlock confirmation? They can't undo this actiona as there
is no way to lock on behalf of someone else.

### Backwards compatibility

Pages that were previously locked for everyone will remain locked for everyone
after this change is deployed to a site.

The models will still support the global lock, and the global locking behaviour
can still be used through either custom Python code or reverting to the previous
behaviour by setting the `WAGTAILADMIN_GLOBAL_PAGE_EDIT_LOCK` setting to `True`.

We will add a data migration so all users and groups who have the "Lock"
permission will automatically get the new "Unlock any page" permission so they
will have the same permissions that they had before.
