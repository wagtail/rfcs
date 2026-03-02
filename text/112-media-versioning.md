# RFC 112: Media versioning

- RFC: 112
- Author: Thibaud Colas
- Created: 2025-12-02
- Last Modified: 2025-12-02

## Abstract

[Media versioning](https://github.com/wagtail/roadmap/issues/58) is a frequent need for the types of project Wagtail is used on. This RFC recaps the relevant features, their current status, and a path forward to implement them.

## Current vs. desired state

Wagtail has two built-in media types: images and documents. Audio and video are also frequently used, though they require third-party packages such as [wagtailmedia](https://github.com/torchbox/wagtailmedia). For Wagtail’s built-in types, versioning is limited to tracking of basic metadata, used inconsistently:

- `created_at`: only displayed in listing views
- `uploaded_by_user`: never displayed
- `file_size`: only displayed in detail views
- `file_hash`: never displayed

### User expectations

We can compare those limited versioning features to what is available for pages, to understand users’ expectations:

| Capability             | Pages  | Images         | Documents      |
| ---------------------- | ------ | -------------- | -------------- |
| Version history        | ✅ Yes | ❌ No          | ❌ No          |
| Version revert         | ✅ Yes | ❌ No          | ❌ No          |
| Version comparison     | ✅ Yes | ❌ No          | ❌ No          |
| Draft/Published states | ✅ Yes | ❌ No          | ❌ No          |
| Scheduled publishing   | ✅ Yes | ❌ No          | ❌ No          |
| Locking                | ✅ Yes | ❌ No          | ❌ No          |
| Workflows              | ✅ Yes | ❌ No          | ❌ No          |
| Audit trail            | ✅ Yes | ⚠️ Yes (basic) | ⚠️ Yes (basic) |

In addition to the surface-level functionality, a lot of those features also combine with Wagtail’s permissions system so sensitive actions can only be performed by authorized users. Users might expect the same level of control for media versioning features.

### Desired state

A lot of those page-centric features have been made available for snippets / arbitrary Django models (see [RFC 85](https://github.com/wagtail/rfcs/pull/85) and subsequent work). A natural step would be to extend all of the above features to images and documents. There is clear opportunity to reuse the mixins introduced for snippets (`RevisionMixin`, `DraftStateMixin`, `WorkflowMixin`, etc), but clear technical reasons why media types would need bespoke implementations.

**We expect there is enough of a need for this functionality to have it "on by default" for image/document media types, rather than requiring opt-in.**

### Expected challenges

- **Underlying files use external storage backends.** Their contents aren’t JSON-serializable in the database like how the `Revision` model works, and the capabilities of those backends vary widely.
- **Files comparison depends on file type.** Creating true comparisons becomes a complex task, while users have high expectations based on other software.
- **Media files can be large.** Versioning requires storing many copies, which means cost and performance concerns.
- **Media models can be overridden.** This means any core changes have to be compatible with a wide range of possible customizations, and ideally any versioning enhancements should also version any additional metadata.

## Features

### Versioning

Those requirements are the most important to meet user needs. From a UI perspective, versioning means:

- A "History" view for each media item, listing previous versions with metadata (upload date, uploader, file size, etc)
- A "Revert" action to restore a previous version as the current one. This likely requires a way to view a previous version first.
- A "Compare" view to compare two versions (at least their metadata, ideally their contents)

#### History view & metadata versioning

This aspect of versioning can likely be implemented with the existing `RevisionMixin`, associating media models with `Revision` instances. The history view would then be similar to the one for snippets, listing previous versions with relevant metadata coming from the `Revision` model:

- `created_at`
- `user`

As a bonus, for images, users would likely appreciate if each entry in the history can display a thumbnail of the image as well.

Implementing this with `RevisionMixin` would avoid introducing more fields to the image/document models, so should be compatible with custom image/document models. If we add `RevisionMixin` to the (abstract) base image/document classes, the functionality will also track fields added to custom image/document models.

Note: An alternative "MVP" history view could also be achieved by displaying audit trail entries (`ModelLogEntry` instances with `timestamp` and `user` fields) for a single media item. This would likely meet some user needs but wouldn’t qualify as versioning.

#### Version revert

This requires preserving not just the media metadata but also the underlying file. We would need to make sure `RevisionMixin` serializes `FileField` contents, and have multiple copies of the uploaded files. Some of the file storage backends support versioning natively, but we expect we would need to build for a low common denominator where versioned copies are managed via Wagtail by changing file names or file paths.

This means:

- On uploading of a new file, preserve the previous file (related: [Original image files are deleted in process of uploading replacement image #13225](https://github.com/wagtail/wagtail/issues/13225)).
  - Wagtail already disambiguates filenames on upload with a hash suffix, so we can likely reuse that logic to preserve previous files.
- Introduce a way to delete old files when no longer needed (e.g. when a revision is deleted, or after a certain retention period).
- The Revisions would then store different references to the different files.
- To revert, the flow would be similar to pages: users would navigate to the media item’s history view, select a previous version, click "Review this version" to view it with the media item’s default detail view and a "You are viewing a previous version" banner, then click "Revert to this version" to update all fields based on the revision contents - including the reference to the underlying file.

For performance / storage costs considerations of preserving many files, we expect there to be minimal concerns due to current usage patterns:

- Sites already provision their infrastructure for 10k to 1M images. Most images are likely to only have 1 to 3 versions over their lifetime, due to the inherent friction in image editing. Users already upload multiple versions of image files separately, so there is already a perceived need to clean up old versions.
- Documents are expected to have lower instance counts but higher file sizes to images. However, documents are also less frequently updated, so versioning is expected to have a lower impact overall.

The cleanup of old files could be implemented as a new management command, or as part of [`purge_revisions`](https://docs.wagtail.org/en/stable/reference/management_commands.html#purge-revisions). We expect this will create occasional breakage when CMS users link directly to media files that are later deleted, but this is already a common issue in the current model and we don’t expect it warrants special consideration.

#### Version comparison

For metadata comparison, we expect to use the same UI as for snippets and pages.

For images, Wagtail already has basic side-by-side display support when comparing two revisions of a page. We expect this can be reused for image version comparisons, both to display the underlying files but also focal points. There is a clear need to [Revamp revision comparison diff styles](https://github.com/wagtail/wagtail/issues/10576), this can be achieved outside the scope of this RFC.

For all other media types, we expect comparison views to only show side-by-side links to the two versions of the files. This is very basic but would work for arbitrary file types. If a better "preview" of document contents is created in the future, it can be repurposed for version comparisons.

### Draft vs. published state

This is out of scope for the purpose of this RFC, but could be considered in the future.

### Scheduled publishing

This is out of scope for the purpose of this RFC, but could be considered in the future.

### Workflows

This is out of scope for the purpose of this RFC, but could be considered in the future.

### Locking

This is out of scope for the purpose of this RFC, but could be considered in the future.

### Audit trail

Currently image and documents support two log entries: "Edited" and "Deleted". It would be nice if this was extended with a "Created". This is out of scope for the purpose of this RFC, but could be considered in the future.

## Configuration & settings

We don’t expect there is enough of a need to disable versioning to warrant a global setting for it.

There is a clear need for sites to have different retention policies for past versions ("never delete", or "always delete after X days"). Within the proposed model, this would be at the discretion of site implementers with their implementation of `purge_revisions`. For sites with the most stringent needs, we may encourage them to use a file storage backend that supports versioning natively.

## Open questions

Here are aspects of the work that have yet to be fully explored.

### UX changes to discourage manual versioning

On lots of projects, CMS users tend to create new media items to replace previous versions, rather than edit the existing image/document. It’s unclear if we should consider UX updates to encourage users to edit existing images / documents rather than just adding new ones.

### Suitability of RevisionMixin for media versioning

There are likely other unidentified blockers to its usage.

### Data migration onto existing sites

It’s unclear what expectations there would be when introducing revisions on existing sites’ data. The "History" view could simply start empty, or we could consider a pre-fill based on audit log entries. Options:

- Empty history
- Single initial revision based on existing data
- Backfill with audit log

### Storage backend versioning

Would it make sense to offload management of media version history to the storage backend?

### Collections permissions

Images and documents have admin-level permissions via collections, and collections support restricting access to the items within them. It’s unclear whether changes to a media’s collection should be handled differently to other field changes.

### Access to previous versions

There are potential performance / privacy / security reasons in restricting access to previous versions of media files. This is left open for further exploration.

### Renditions impact

It’s unclear if there are any needs to adjust how renditions are handled when versioning is introduced.
