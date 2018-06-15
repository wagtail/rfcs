# RFC 29: Immutable Revision Events

* RFC: 29
* Authors: Andy Chosak, Ryan Verner, and David Prothero
* Status: Draft
* Created: 2018-06-15
* Last Modified: 2018-06-15

## Abstract

Based on [these notes](https://hackmd.io/s/ByDdmUZWm) from discussions held at Wagtail Space US 2018.

Let's make Wagtail a first class system for reviewing and approving content. This is a large problem space, but we believe we have identified one architectural improvement that will enable future solutions in this problem space.

In short, we propose that we create a new RevisionEvents model that records immutable events that occur to a specific page Revision, such as being submitted for moderation, published, unpublished, etc. Further, we propose that this model be accompanied by an API to allow Wagtail developers to add their own events to facilitate their own custom workflows.

## Specification

### Goals

 - Revision list should be easier to grok (what changed on each revision, what is relevant to me).  Currently a new Revision is saved on every page save or major state change, so we end up with lots of revisions.  Should be able to see what fields changed, a label showing if the Revision was ever live or submitted for review, etc

 - Making the revision diff view easier to grok.  Look at pre-rendering content (including RichTextFields & StreamBlock elements) and displaying the rendered differences.
 
 - Making it easy for a dev to add new arbitary data associated with Revisions - i.e. ability to add comments on a piece, add a comment to an approval (or rejection) etc.
 
 - Make the revision review/approval workflow way less confusing.
 
 - Add extensibility points for plugging in more complex workflows.

### Task 1: Create an immutable revision events model

#### Intention
Revision model should deal strictly with content. A new RevisionEvents model should record immutable events that happen with respect to the Revision.

In other words:
 - Revisions should only be created when content is changed
 - State changes live in the RevisionEvents model

#### Proposed model
- FK to Revision
- Event Identifier (string)
  - **Examples:**
  - submit_for_moderation
  - publish
  - unpublish
  - *expandable by user...*
- Timestamp
    - When the event occurred. For example, if you wanted to know when a Revision was last published, you would get the most recent Timestamp for the `publish` event.
- Event Data (JSON)
    - Optional structure containing meta data about the event (e.g. for a theoretical "Comment" type event, it could contain the text of the comment)
- User

## Open Questions

TODO
