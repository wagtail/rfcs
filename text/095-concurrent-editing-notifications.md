# RFC 95: Concurrent editing notifications

* RFC: 95
* Author: Matthew Westcott
* Created: 2024-03-13
* Last Modified: 2024-03-13

## Abstract

Currently, if two users edit the same page or snippet at the same time, the second user to save their changes will overwrite the first user's changes. This doesn't seem to be a common problem, and can often be avoided through communication - however, it will need to be addressed in advance of implementing auto-saving, as this increases the likelihood of users overwriting each other's changes unknowingly.

The "holy grail" of avoiding edit conflicts would be a real-time collaborative editing feature where changes are reflected immediately on the other user's screen, but this will not happen any time soon. A locking mechanism has been proposed, but this raises further implementation questions (for example, who has authority to override a lock in the case that an editor leaves a page open and goes on holiday?) and would be overly disruptive.

This RFC proposes an approach where users are notified of any other users editing the same item. Users will not be blocked from editing, but will be provided with information about the potential for conflicts, and how to resolve them when they do occur.

## Specification

We introduce a new model `EditingSession` that tracks the users who currently have the 'edit' view for a given object open. This consists of a generic foreign key to the object, a foreign key to the user, and a 'last seen' timestamp.

### Base implementation - tracking active sessions

A minimal implementation would simply show a list of other users with an active editing session on the same object, without any notification of when those users actually make changes. This would be the only functionality available for non-versioned snippets, since we need a revision ID to identify whether the last saved version is the one that the current user is editing.

When a user visits the 'edit' view for a page or snippet, an EditingSession record is created for that object and user, with the current timestamp, and the EditingSession's ID is passed as a JS-accessible value on the template.

We add a 'ping' endpoint view that receives an EditingSession ID as a parameter, validates that the EditingSession belongs to the current user, and updates the 'last seen' timestamp to the current time. It returns a JSON response containing a list of all EditingSessions other than the current one that match this object and have a timestamp within the last minute.

> [!NOTE]
> It may also be necessary to perform a permission check to verify that the current user has edit permission on the object, so that users cannot use the 'ping' endpoint as a means to track user activity outside of their own area of the site. (This cannot happen if going through the 'edit' view is the only way to acquire an EditingSession ID, but see the "Handling unknown EditingSession IDs" section below for a setup where this could be bypassed.)

We also add a 'release' endpoint view that receives an EditingSession ID as a parameter, validates that the EditingSession belongs to the current user, and deletes the EditingSession.

A JavaScript handler is added to the edit view that sends a ping request to the 'ping' endpoint every 30 seconds, and updates the list of active users based on the response.

> [!NOTE]
> This will also identify cases where the current user has the edit view open in multiple tabs. We might consider highlighting this in the list as a special case, and / or removing duplicate users from the list.

The list of active users will also be included in the initial rendered response of the edit view, so that we don't have to wait for the initial 'ping' response for this information to be displayed.

When a user leaves the edit view (as determined by a [`visibilitychange` event](https://developer.mozilla.org/en-US/docs/Web/API/Document/visibilitychange_event) changing visibility to 'hidden'), the 'release' endpoint is called (using [`sendBeacon`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/sendBeacon)) to delete the EditingSession.

> [!NOTE]
> The `visibilitychange` event does not guarantee that the user has actually left the page - according to the Mozilla docs, it can occur when switching tabs or applications on mobile, for example. However, it seems to be the best option available across all browsers.

### Clean-up of outdated sessions

Since we cannot guarantee that the 'release' endpoint will be called in all cases (for example, if the user's browser crashes), it is possible that stale EditingSession records will be left in the database. These will not affect functionality (since we disregard sessions older than a minute), but over time they may impair performance and bloat the database. These could be cleaned up with a scheduled task, but the extra work for site owners to set this up is probably not worth the benefit. I therefore propose that we perform this clean-up in the edit view, at the point where an EditingSession record is created. This will delete all EditingSession records that are more than one hour old.

### Handling unknown EditingSession IDs

Given the above cleanup process, and the potential for a `visibilitychange` event to occur without the user actually leaving the page, it's possible that a browser could resume sending 'ping' requests for a given EditingSession ID some time after the record has been deleted. In this situation, we would want the user to reappear in the list of active sessions. To handle this, requests to the 'ping' endpoint should pass the object's ID and content type in addition to the EditingSession ID, and in the case that the EditingSession is not found, the view will create a new one. The ID of the EditingSession (whether retrieved or newly created) will also be included in the response, so that subsequent 'ping' requests can be made with the new ID.

### Distinguishing between "currently viewing" and "currently editing"

The above implementation will show all users who have the 'edit' view open for a given object, regardless of whether they have made any changes. It would be helpful to distinguish between users who are currently editing the object and those who are simply viewing it.

To do this, we add a 'has unsaved changes' boolean field to the EditingSession model. When the edit view sends a 'ping' request, it will use the existing 'dirty fields' mechanism to determine whether the current user has unsaved changes, and pass this flag as a parameter to the 'ping' endpoint. The 'ping' endpoint will then update this field along with the timestamp, and the list of existing sessions it returns will also include this flag. The edit view can then apply a visual distinction to those users in the list.

### Notifying users on changes

We wish to notify users when another user makes changes to the object they are currently editing. This is possible for pages, and snippets inheriting from `RevisionMixin`.

The edit view passes the current revision ID as a JS-accessible value on the template. This is passed as a parameter to the 'ping' endpoint, and the 'ping' endpoint response will include a list of revision IDs for the object that are newer than the passed revision ID, along with the user who created the revision and the timestamp.

If this list is non-empty, the edit view will display a notification within the 'active users' listing, against the user who made the change, saying "{username} has saved a new version". (This may require adding that user to the list, since it's likely that they have ended their editing session at this point.) The notification will include options to 'dismiss' and 'refresh'.

'Dismiss' will close the notification, and add the revision ID to a list of dismissed revision IDs (stored as a local JS variable for the current view) which will be ignored in any subsequent 'ping' responses.

If the current user has not made any changes, the 'refresh' option will immediately reload the page. If the current user has made changes, the 'refresh' option will display a confirmation dialog, warning the user that they will lose their changes if they proceed. If they confirm, the page will be reloaded.

When the user performs any save or publish action, if a 'ping' response has returned any revisions at any point (including ones that were dismissed), the user will be shown a confirmation dialog warning them that they will be overwriting existing changes made by other users. If they proceed, the save or publish action will be performed as normal.

### Applicability to auto-save functionality

It is expected that the approach presented here will remain largely in place when auto-save functionality is implemented. Auto-saving will occur as a background request that can take the place of the 'ping' endpoint whenever changes have been made, with the 'ping' request still being made at 30s intervals when no changes have been made. Auto-save will be initially active when the user opens the edit view, but will be paused if a notification of another user editing the item is received. In the simplest implementation, it will remain paused until the user reloads the edit view (either by selecting 'refresh' on the notification, or explicitly saving or publishing their changes and accepting the confirmation dialog to overwrite the other user's changes), at which point a new editing session will be started with auto-save active.

The "has unsaved changes" flag on EditingSession will largely fall out of use at this point, since users will generally be sending 'save' requests rather than 'ping' requests when they have unsaved changes. (The exception is when that user's auto-save state has been paused through the mechanism described above.) A non-editing user who has the 'edit' view open while another user makes an edit will see that user's state transition directly from 'viewing' to the "has saved a new version" notification. If the non-editing user dismisses that notification, the editing user should continue to be shown in the 'editing' state until their editing session ends.

## Open Questions

// Include any questions until Status is ‘Accepted’
