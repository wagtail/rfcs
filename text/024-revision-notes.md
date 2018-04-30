# RFC 24: Release Notes on Revisions

* RFC: 24
* Author: Edward Henderson
* Status: Draft
* Created: 2018-03-29
* Last Modified: 2018-04-30

## Abstract

Enhance the revision system to allow release notes to be added when publishing.

## Specification

The current revision system is great, but does not offer a way to provide release
notes for a revision. This RFC proposes that the mechanism be expanded to allow a custom release model to be used for a page, and a way to override the publish dialog to support addition of release notes.

### User Experience

Adding comments for a draft should be optional, but allowed. I could see a simple option, possibly with a keyboard enabler, such as Control+Click on Save Draft will show
a comments dialog. Another option would be make comments be a page specific option, so they are turned on/off for a page

### Editing Notes

One appraoch here would be to allow editing revision notes in the Settings panel. I don't think this should be a "required" option, but more of a optional one, enabled
in configuraiton.

If notes are part of the UI, then they should be paged, since it would be possible to have many notes.

Also, if notes are captured during draft mode, they should be consolidated and condensed during release, so there would only be draft notes betwween publishes, and all drafts would be combined into the publish notes.

### Tagging Revisions

This was not part of my original idea. I don't personally see a need to tag revisions, since I assume that revisions are a linear history of a given page.


## Open Questions

### Should revision history be collected for Draft revisions?

I can see that it might be important to capture notes for a draft release, and then combine those notes into a published release.
If each draft offers the ability to add a note for the release, then a publish could then combine the notes from the drafts created since the last release, into
a single release note.

I do not suggest anything very fancy here. Limit the release notes to simle text, or, at best text with Markdown support that can be rendered later. This
format would be easy to work with, and easy to concatenate all notes from a list of "drafts".

### Should this enhancement replace current functionality?

I would suggest an "alternate" Revision model, that could be conigured in settings. This maintains the existing behaviour, but allows for an option to specify
a new PageRevision model. The core of wagtail could offere multiple such model, and users would be able to provide their own.

Custom models would require some ability to specify UI updates

