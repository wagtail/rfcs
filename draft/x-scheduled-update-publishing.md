# RFC X: Scheduled Publishing for Page updates


*   RFC: 21
*   Author: Patrick Woods
*   Status: Draft
*   Created: 2017-10-04
*   Last Modified: 2017-10-13

## Abstract


Scheduled publishing currently only effectively allows you to schedule
new pages. If you schedule a revision for a currently published page, the
page will be unpublished in the process. This behavior limits the
feature's funtionality to only new pages or pages that can be
unpublished until the new publish time.

This proposal is to allow for the ability to schedule the publishing of
revisions to current live pages without unpublishing them in the
process. This would allow editors to schedule updates to pages in
advance.

## Specification


Scheduling a page that is currently live should not change the live
status of that page, the revision should be marked for later publishing
and handled through the management command.

Scheduling a page that has not previously been published should not have
its status set to live until the go live time is met

### New Page Workflow

This would follow the current publishing workflow

1.  User creates a new page with a `go_live_time` in the future
2.  User publishes page
3.  Page does not go live
4.  User receives a message indicating that page has been scheduled

### Existing Page Workflow

1.  User updates a page with a `go_live_time` in the future
2.  User publishes page
3.  New changes do not go live
4.  Page is still accessible with old revision
5.  User receives a message indicating that page has been scheduled

### Naive implementation

This seems to be achievable by changing `PageRevision.publish` to not
update the page object if `go_live_time` is in the future. So the
scheduled part of `PageRevision.publish` would resemble:

``` python
if page.go_live_at and page.go_live_at > timezone.now():
    # Set the approved_go_live_at of this revision
    self.approved_go_live_at = page.go_live_at
    self.save()
    # And clear the approved_go_live_at of any other revisions
    page.revisions.exclude(id=self.id).update(approved_go_live_at=None)
	# if we are updating a currently live page leave the rest for
	# the publish scheduled task
	if page.live and page.live_revision:
		return
	# new page with go_live in the future don't make the page live
	page.live = False
```

This achieves the basic functionality of allowing scheduling of
revisions to existing pages. The messaging in the view needs to be
adapted to look at revision for the `go_live_time` instead of the page
since the underlying page object has not been updated.

## Open Questions


### No indication that there is a scheduled revision.

Currently there will be no status change to indicate a new revision is
scheduled. Do we need another status like \[live + scheduled + draft\]?

### No indication of which revision is scheduled

The current revision history view provides no way of seeing which
revision is scheduled. This would be a useful indicator.

### Can we have multiple scheduled revisions?

The current behavior assumes the last published change will cancel any
scheduled revisions. This could be made opt in when publishing if we
want to allow multiple ones. Some of the revision publish logic would
have to change.

### How to handle changes to current published revision if another revision is already scheduled?

How should we handle a user scheduling a revision and then having to
edit the current live revision, say to edit a typo? If we are only
allowing one published revision the current reverting workflow can
handle this.

1.  user goes back to current live revision edits it an publishes it.
    This cancels the currently scheduled revision
2.  User goes back to the previously scheduled revision replaces it into
    the current revisions and publishes it, returning it to the
    originally scheduled window.

This workflow has the advantage in that it maintains the linear stack of
the revision history.

```
    [ Re-scheduled Revision 2 ]
              |
    [ Edited Revision 1 ]
              |
    [ Scheduled Revision 2 ]
              |
    [ Initial Revision 1 ]
```

The re-scheduling workflow could probably be handled automatically when
publishing a page with a scheduled revision, but it is currently
possible with existing tools.

### Should Go live time be limited to certain permissions?

Currently go live and expiry dates are exposed to non privileged users.
Should this change? Not clear if this needs be done as part of this
functionality.

### Synchronize admin messaging and revision publishing/scheduling

Currently the admin messaging goes through its own logic to figure out scheduling vs publishing.
Since that behavior is really controlled by `PageRevision.publish` should that method return a status
that the view can leverage to determine messaging?  The naive implementation above for example causes
the behavior and messaging to get out of sync.
