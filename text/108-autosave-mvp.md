# RFC 108: Autosave MVP

* RFC: 108
* Author: Matthew Westcott
* Created: 2025-05-22
* Last Modified: 2025-05-22

## Abstract

This RFC describes a minimum viable product for automatic background saving of pages and snippets, following preliminary work on concurrent editing notifications (RFC 95) and deferring validation when saving drafts (RFC 104).

## Specification

### Backend implementation

The existing admin views for creating and editing pages and snippets will be extended to accommodate background POST requests. These views are conventionally located at:

* `/admin/pages/add/<app_label>/<model_name>/<parent_id>/` (page creation)
* `/admin/pages/<pk>/edit/` (page editing)
* `/admin/snippets/<app_label>/<model_name>/add/` (snippet creation)
* `/admin/snippets/<app_label>/<model_name>/edit/<pk>/` (snippet editing)

The changes will be as follows:

* On receiving a POST request with an `Accepts:` header that does _not_ include `text/html`, the view will return a JSON response instead of the standard behaviour of returning a redirect (on success) or an HTML page (on failing validation), and no notification messages will be added to the user session.
* The edit views will accept a new optional POST parameter `overwrite_revision_id`. When this is passed, a successful save will update the `Revision` record with the given ID instead of creating a new revision. An error response will be returned if the revision does not belong to the object being edited or the current user, or the object being edited has a revision with a newer timestamp than the one given (which would indicate that there has been a conflicting edit in another editing session).

Other than these changes, the body of the POST request will be in the same format as a regular form submission. (It would arguably be neater for both the request and response body to be in JSON format, and it is hoped that we may support this in future through the use of Telepath adapters for all elements of the form - however, at the present time the only supported way of interacting with many elements such as InlinePanel is through server-side code handling a conventional form POST.)

### JSON response format

Errors will be indicated with a 400 Bad Request HTTP response and a body consisting of a JSON object with the following fields:

* `"error"`: a text description of the error
* `"error_code"`: a string identifier for the error that will remain stable across releases

Success will be indicated with a 200 OK HTTP response and a body consisting of a JSON object with the following fields:

* `"success"`: has the value `true`
* `"object_id"`: contains the primary key of the object that has been created or updated
* `"revision_id"`: contains the primary key of the revision that has been created or updated

### Hook integration

The editing workflow hooks `before_create_page`, `after_create_page`, `before_edit_page`, `after_edit_page`, `before_create_snippet`, `after_create_snippet`, `before_edit_snippet`, `after_edit_snippet` will be run for background JSON requests just as they would for a regular post. However, the hook mechanism allows hook functions to return HTTP responses to be returned to the user in lieu of the standard one, and this could disrupt the operation of client-side code that expects JSON responses in a well-defined format (if the hook function has not been written to specifically account for this). To address this, special-case behaviour will be put in place when responsing to a request with an `Accepts:` header that does not accept `text/html`:

* If the response returned from the hook has content type `application/json`, it will be returned to the user and later hook functions in the sequence will not be called.
* If a non-JSON response is returned by a `before_*` hook (which, as per the current hook semantics, blocks the save from occurring), a JSON error response is returned with the message "Request to edit page was blocked by hook" or similar. Later hook functions in the sequence will not be called.
* If a non-JSON response is returned by an `after_*` hook, that response is ignored and the standard JSON success response is returned. Later hook functions in the sequence will not be called.

### Client-side implementation

On page load, the client-side code sets up the following state:

* `last_saved_form_state`: the serialized data of the form
* `overwrite_revision_id`: initially null
* `submit_url`: the action URL of the form

At a defined interval (say 30 seconds), the serialized data of the form is assembled. If this differs from `last_saved_form_state`, and there is no active POST request pending a response, a POST request is made to `submit_url`, with an `Accepts: application/json` header, consisting of the new saved form state, plus `overwrite_revision_id` if this is non-null.

On receiving a 'success' response to this request, `last_saved_form_state` will be replaced with the updated form state as submitted; `overwrite_revision_id` will be updated with the revision ID in the response (so that subsequent POSTs overwrite this revision instead of creating a new one); and if the POST request was made to a 'create' endpoint, `submit_url` is changed to the appropriate 'update' endpoint, using the object ID in the response (so that subsequent POSTs are sent as updates rather than creations).

On receiving an 'error' response to the request, a message will be displayed to the user indicating that the page could not be saved. If the error code indicates that `overwrite_revision_id` is not the newest revision (meaning that another user has edited the page), the code will stop sending further auto-save requests for the current page.

In the initial implementation of the feature, validation errors encountered while auto-saving will not be displayed alongside the corresponding field, as the logic for doing this correctly is currently only in place in server-side code as part of a full page render. Editors will need to manually save a draft, triggering a full page render, to see the validation errors in place.

### Updating form state post-save

After a successful save, it may be necessary to update certain elements of the form to ensure that subsequent saves are valid. For example, child objects within an InlinePanel are inserted with a blank hidden ID field, and these are created as new database records upon saving a draft. If the form were to be fully re-rendered at this point, the hidden ID field would be populated with the record's primary key, causing subsequent saves to update the existing record. Since the form is not re-rendered in full upon auto-save, we must explicitly update the hidden field with the assigned value - failing to do so would cause duplicate objects to be created on the next save. A similar effect would be seen in the comments mechanism, which like InlinePanel is built upon Django's inline formsets.

(StreamFields do not appear to be affected by this - blocks are assigned a UUID client-side on insertion, so there is no change to the form state on save.)

To address this, the `WagtailAdminModelForm` class (and the panels mechanism if necessary) will be extended so that after a successful save, it is possible to retrieve a dictionary of form fields that have received updated values in the save operation, mapping form field names to their new values. This dictionary will be returned as part of the 'success' JSON response, and the client-side code will use this to update form fields within the HTML.

## Open Questions

* As a performance optimisation, can we combine the background requests to the 'save' endpoint with the background HTTP requests that already exist - namely, the 'ping' endpoint for concurrent editing notifications, and live previews?
* Is there a need to support user-defined server-side logic within the "updating form state" mechanism, or is it sufficient to support inline formsets?
* How should we handle POST requests that never complete or fail with a non-JSON response - due to a loss of network connection, for example? In this situation there is no way to know whether the save actually occurred, and thus whether it is safe to resubmit in the case that the previous save involved creating objects such as InlinePanel children. (This is probably equally true for regular manual saves, though.)
