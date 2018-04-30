# RFC 23: Release Notes on Revisions

* RFC: 23
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

I would suggest not

### Should this enhancement replace current functionality?

I do not know the details of how the revision model is managed by the Page model, but if possible this feature should be implemented as a way to enhance the existing model, most likely by specifying a different revision model to use, and allowing the developer to enhance this base model with one that accepts release notes.

