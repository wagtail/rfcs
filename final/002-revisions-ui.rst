===================
RFC 2: Revisions UI
===================

:RFC: 2
:Author: Matthew Westcott
:Status: Final
:Created: 2015-12-07
:Last-Modified: 2015-12-07

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========

From the outset, Wagtail has stored revisions of pages on save, to support features such as page moderation; however, these have not been directly accessible to editors. This proposal describes additions to the Wagtail admin interface to allow editors to view and roll back to earlier revisions.

Specification
=============

A new "Revisions" (or "History") button will be added to the footer of the page editor, alongside the "last modified by" details; proposed rendering here: https://github.com/torchbox/wagtail/pull/1946#issuecomment-160163727

This button will open a listing of previous revisions for the page, as a new view replacing the page editor. (Any edits made by the user prior to clicking the button will be lost, so we suggest that prior to developing this feature, a "you have unsaved changes; are you sure you wish to leave this page?" popup is implemented.)

Listing view
------------

This view will display a paginated listing of revisions of the page, most recent first. For each revision, this will display the following information:

* The date and time the revision was made
* The full name (as reported by ``User.get_full_name``) of the user who made the revision
* Whether this revision was ever published or submitted for moderation. (This information is not currently stored, so this may be left as a future enhancement.)

For each revision, the following actions will be available:

* View
* Revert
* Compare (optional)

View revision
-------------

This will display the chosen revision, rendered through the page's template, similarly to the Preview button.

Revert
------

This will return the user to the page editor, with the form fields prefilled with content from the chosen revision. No change to the page record will be made at this point; the user must save, submit or publish the page for the change to take effect. At this point the save / submit / publish action will work as normal, and create a new revision on top of previous changes; no history will be lost.

Compare
-------

If implemented, this will show a field-by-field comparison between the selected revision and the previous one, or the selected revision and the current page state. (Further specification of this feature is out of scope for this WEP, but see https://github.com/torchbox/wagtail/pull/1946 for an existing implementation.)


Rationale for design decisions
==============================

We considered several alternative locations for the link / button that opens the revision listing:

* A new button on the page explorer listing, alongside "Edit", "Move", "Copy" and so on. This was rejected because it would be more natural for a user to access this from the page editor rather than the listing; "View revisions" is not really an action in the same way that the others are; and it was felt that the list of actions is long enough already.
* A new 'Revisions' tab alongside "Content", "Promote" and so on. It was felt that this was too prominent for a feature that would probably only get occasional use. Also, if the listing is to exist as a separate URL view, then this would break the tab paradigm whereby users should be able to freely switch tabs without losing data.
* Within the "Settings" tab. Revisions are not really a 'setting' as such, and this would set us down the path of Settings becoming a catch-all "miscellaneous" placeholder.
* Making the "last modified by" text into a link. It was felt that this was not discoverable enough.

We also considered opening the revision listing in a modal, rather than a separate view, so that the user would not have to leave the page editor interface. However, modal popups are difficult to use on mobile.

Open Questions
==============

* Should 'view revision' open in a new window?
* Is mobile use the *only* downside of opening the listing in a modal? If so, why are we more concerned about it here than in all the other places we've used modals?
