# RFC 50: Commenting in the Wagtail Editor

* RFC: 50
* Author: Jacob Topp-Mugglestone
* Created: 2020-05-14
* Last Modified: 2020-05-14

## Abstract

Currently, commenting on a draft version of a page is done through [Wagtail Review](https://github.com/wagtail/wagtail-review/), which has been reworked for the upcoming Workflow feature. However, this has several downsides:

- Comments are only visible on a preview version of a page, not in the editor
- Comments are linked to offsets in the frontend html, so cannot be accurately traced back to positions in the database representation (due to filtering, particularly the `richtext` filter). As a result, inline comments cannot be transferred between revisions - viewing a comment made on a previous revision requires previewing that revision.
- Templates must be specifically tagged, with markup around individual commentable fields, to enable this behaviour (by allowing Wagtail Review to identify fields)

As part of the Workflow feature, we’d like enable to in-editor commenting, with a view to eventually integrating with Review, or a Review-like comment-on-preview as well. In-editor commenting would not require users add specific markup to their templates, would provide a more fluid user experience when switching between viewing comments and editing, and can be made more robust and transferable between revisions.

Commenting is primarily expected to be used as more specific feedback during moderation, but programmatic generation of comments could be used for automated feedback - eg grammar or spelling checks - or imported comments from external sources like Google Docs. 

We propose this goes into Wagtail in the following ways:
1. In core/admin: additional APIs/hooks for modifying the ContentState to database html conversion process, storing a unique id for paragraphs between Draftail and database html (also useful in diffing and translation), only saving new page revisions when content has changed, `contentpath` attributes
on fields.
2. In core/admin: the ability to add comments on fields or individual StreamField blocks
3. In contrib/a third party app: the ability to add anchored/inline comments on specific substrings within RichTextFields, as a new Draftail feature

## Specification

This feature should be divided into two forms of commenting. The first is commenting on a field, or streamfield block level. The second is inline/anchored commenting within rich text.

### Field and Block Commenting

This will be the standard form of commenting. Comments can be left on either a field or a StreamField block.

To identify the field or block, we use a `contentpath` identifier, in the form: `field.streamfield_block_id.field…`. This is stored
in a field on the comment model, along with the page and revision it was created on, the date/time of creation, and the commenting user.

Comments can be added and viewed from the page edit view, via an API similar to the existing Wagtail Review approach. This means that adding field comments do not require saving the revision. Users can also reply to comments in threads (again, similarly to the Wagtail Review approach).

The visibility of comments can be toggled: perhaps using a comment icon and a number in the red field heading to indicate hidden comments.

Potential features:
- Comments may have quote text
- Comments left on previous revisions include a link to view that revision, or potentially the contents of the field on that revision


### Inline Draftail Commenting

Anchored comments in Draftail pose a more complex problem:
- Their offsets change as a field is edited, so cannot be saved to the database until a page is saved
- Draftail cannot easily access the offsets in the database representation, due to server-side logic in converting ContentState to database html
- While adding a comment anchor inline entity in Draftail, and saving offsets to a comment model during ContentState conversion is a good approach, and enables comment positioning to change appropriately between revision, saving currently triggers the creation of a new page revision, even if no changes have been made in the resulting page data. This can result in pages going back through previously approved workflow stages, depending on site settings.

To resolve this, we suggest an additional model for inline comments, which links a comment to its paragraph and character offsets within a particular revision, paired with a comment anchor inline entity in Draftail, which functions as follows:
- We persist a unique id between Draftail paragraphs and the database representation paragraphs - we save this id to the database.
- When an anchored comment is created in the editor, it is initially linked to the paragraph id, with no offsets
- When the page is saved, during ContentState conversion, the comment anchor entity is stripped out and the offsets (relative to the paragraph) saved in the comment linking model.
- To prevent spurious revisions being saved, we compare the revision content with the previous revision on save draft, and only save a new revision if there have been changes. Because comments are stripped out upon conversion, just adding comments to a page will result in no new revision.

While anchored comments should be a third party or contrib app due to the different UX, to provide this functionality, the following changes in core would be necessary:
- Adding a unique ID shared between Draftail paragraphs/ContentBlock and the database html representation.  This is also useful for diffing and translation.
- Adding a way to hook in at specific points in the ContentState to db html conversion process (to generate the offsets in the db representation accurately, this is necessary, and the existing rich text features hook is insufficient). This could be in the form of additional hooks for pre- and post- conversion functions.
- Comparing revision content before saving a new revision. This is more generally useful as well: it would prevent the creation of spurious revisions and make for a cleaner edit history, as well as working better for detecting edits during a workflow.


## Open Questions

- How should many comments on a single field be displayed?
