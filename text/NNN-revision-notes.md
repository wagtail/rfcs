# RFC NNN: Release Notes on Revisions

* RFC: NNN
* Author: Edward Henderson
* Status: Draft
* Created: 2018-03-29
* Last Modified: 2018-04-30

## Abstract

Enhance the revision system to allow release notes to be added when publishing.

## Specification

The current revision system is great, but does not offer a way to provide release
notes for a revision. This RFC proposes that the mechanism be expanded to allow a custom release model to be used for a page, and a way to override the publish dialog to support addition of release notes.


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
