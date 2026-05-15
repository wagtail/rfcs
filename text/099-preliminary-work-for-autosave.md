# RFC 099: Preliminary work to support auto-save functionality

* RFC: 099
* Author: Matthew Westcott
* Created: 2024-06-20
* Last Modified: 2024-06-20

## Abstract

A much-requested feature for Wagtail is the ability to automatically save changes in the background while editing a page or snippet. Achieving this to an acceptable standard of user experience will involve completing various milestones that are best handled as separate development efforts. This RFC outlines various items of work that are pre-requisites for implementing auto-save functionality.

## Concurrent editing notifications

When multiple users have the editing interface for a page or snippet open at the same time, the last user to hit save will generally overwrite any changes made by the other users. This problem is compounded when auto-saving is active, since this overwriting will happen without the user's explicit action. To address this, we will implement a notification system that warns users when another user is editing the same item, and disables auto-saving when potentially conflicting changes are detected. This feature is covered by RFC 95.

## Deferring validation until publish

Currently, a revision can only be saved if it passes validation. This is undesirable because a user may spend an extended amount of time editing a page while it is in a state that would not pass validation - for example, a blog post might have a required 'category' field at the bottom of the form, and a user leaves this unset while they are writing the body of the post. In this situation, auto-save will not be able to save the user's work in progress.

To address this, we will relax the validation behaviour so that as standard, validation rules such as `blank=False` are not enforced when saving drafts. Instead, full validation will be enforced at the point that a page is published, scheduled, or submitted to a workflow.

Since we need to create an initial database record before we can save revisions, any data that is invalid at the database level (such as a nonsensical date in a date field, or non-numeric data in an integer field) will still produce an immediate validation error. Developers will also be provided with a way to override this behaviour and enforce immediate validation for specific fields where appropriate - this could perhaps be done via an argument on `FieldPanel`. Wagtail itself will make use of this mechanism to require the title and slug fields to be filled in before a page can be saved.

(Implementation note: Django does not enforce validation rules in a model's `save` method, so we are not inherently prevented from saving data that breaks those rules. Rather, it happens in the `full_clean` method, which is part of the `Page` model's `save` logic by default but can be skipped. We will also need to prevent validation from being applied at the form level; this could possibly be done by stripping flags such as `required=True` from the derived form fields.)

Deferring validation in this way will also improve the user experience of the live preview panel, which currently pauses updated while the page is in an invalid state. This does however mean that template authors will need to code more "defensively" to ensure that their templates are preview-friendly, since they will potentially be passed invalid or blank data that would ordinarily be prevented by the validation rules.

## Dynamic insertion of validation error messages

Even with the above change, there will still be situations where a validation error is triggered on saving a draft, such as a blank title or a slug that is already in use. For a conventional form submission, the entire form is re-rendered with error messages inserted next to the relevant fields. When auto-saving is introduced, we will need to communicate these errors to the user without performing a page reload. The placement of these errors varies according to the panel and field; for example, errors within a StreamField are attached to the block that caused the error, and `TabbedInterface` additionally shows the count of errors as a badge on the tab. Errors also need to be indicated on the minimap.

Currently, the [Telepath](https://wagtail.github.io/telepath/) library is used within StreamField to provide a client-side API for block objects that are defined in Python and would conventionally only support server-side rendering, and the same approach can be used across the whole form. We will implement Telepath adapters for panel objects, which will provide a method to obtain a client-side panel object when given a DOM fragment. This object will then provide a method for inserting error messages into the panel.

## Further applications for Telepath panel adapters (optional)

Having Telepath adapters for panels would open up the possibility of further enhancements which, while not strictly necessary for auto-save, may improve the consistency of the codebase.

Firstly, the client-side panel object could provide a method for retrieving the current value of any fields within that panel. Calling this on the top-level panel would return a dictionary of all field values, which could then be submitted to a JSON API endpoint to save the draft. This would avoid the need to construct a Django Form object to process the submission, which may provide a way to bypass form-level validation.

Secondly, the adapter could provide support for dynamically inserting panels, much as StreamField does; this would allow sharing more of the implementation between StreamField and InlinePanel, and provide functionality such as duplication and inserting at arbitrary positions, which is currently missing from InlinePanel.

## Open Questions

// Include any questions until Status is ‘Accepted’
