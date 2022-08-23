# RFC 000: References index

- RFC:
- Author: Karl Hobley
- Created: 2022-08-23
- Last Modified: 2022-08-23

## Abstract

In this RFC, I propose to implement a `ReferencesIndex` to allow efficient and complete lookups of references to any object.

In summary this proposes:

- Extract references from objects on save. References include any external objects used in a `ForeignKey` , chooser block, `RichTextField` or `RichTextBlock`. They also include any block types that have been used in streamfields.
- Add a `ReferencesIndex` model where each record stores all of the references from a particular object in a consistent and easy to query way
- Add a `references` field to `Revision` so that we can search for revisions by the objects they reference or blocks they use
- Add a `rebuild_references_index` management command which can be used if the index gets out of sync with the data (on initial upgrade, or after a big migration)

## Motivation

As part of an initiative to upgrade snippets in Wagtail (which would eventually lead to having Workflows on snippets!), we plan to add support for draft state to snippets.

We think that when a snippet is in draft state, it would not be selectable in any snippet choosers.

We also think it should be clear to the editor where any snippet is being used on a site, and warn/block them if they attempt to unpublish a snippet that is selected in any existing snippet chooser.

Wagtail has a [utility function](https://github.com/wagtail/wagtail/blob/main/wagtail/admin/models.py#L27) that gets an objects usage and this is exposed in the admin for Images and Documents in their "usage" views. This method works by introspecting model definitions to find any `ForeignKey` that point at the given object's model, then performs a query on all those models to find objects that reference it.

This method does not check StreamFields or rich text fields, as these are more complex data types and we haven't got any method to unpack them in the database to find the references. Such a method could be very slow on large sites as the content of these fields can be quite big and we would be unable to use a database index.

## Specification

### Extracting references from objects

Whenever the `post_save` signal is called (which will happen whenever an object is edited), all references to external objects, and usages of blocks will be extracted out so they can be indexed separately. This extraction will scan for external references in `ForeignKey`s, `RichTextField` and chooser/rich text blocks within a `StreamField`.

Internally, we will create a new `References` class which stores all of the references that have been extracted from an object and is responsible for serialising them for storage.

The internal representation of a `References` object would be a dictionary where the key represents the external object or block being referenced and the value is a list of content paths on the page to describe where the object was referenced from.

For example, if a page contains a paragraph block in its body field, and there is a link from that paragraph block to a page with ID `10`, the references would contain the following data:


    {
        "object:wagtailcore_page:10": ["body.123456"]
        "block:body.paragraph": ["body.123456"]
    }


Using just this information, it’s possible to tell that the page with ID `10` was referenced from a paragraph block as the content path of the block is a prefix of the content path of the object. This could be useful information to display when showing the back-references of that referenced page. Utilities to perform this lookup will be available on the `References` object.

### Storage of references in the database

`**ReferencesField**`
We will introduce a `ReferencesField` field type which inherits from `JSONField`. This field type will convert `References` objects to/from their JSON representation stored in the database.

This field will implement lookups to allow finding instances which reference a particular object or block. For example:


    # Find all revisions that contain a reference to an image object
    image = Image.objects.get(title="find me")

    revisions = Revision.objects.filter(references__contains_object=image)
    objects = ReferencesIndex.objects.filter(references__contains_object=image)

Two instances of this field will be added into Wagtail. One one a new `ReferencesIndex` model which will store references for live objects, and another on `Revision` which will allow us to store external references from within revisions too.

`**ReferencesIndex**`
We will add a `ReferencesIndex` model which stores the references from any live object that is editable in Wagtail (pages, snippets, images, documents, and modeladmin).

This model will contain a `GenericForeignKey` field which points at the instance where the references were extracted from and a `ReferencesField` that contains the references

**Acceleration with database indexes**

For PostgreSQL users, we will add a `GIN` index to `ReferencesIndex` to allow very fast lookups of back-references for a particular object type. It’s unlikely that a full-scan of all revisions would be needed (generally, I expect us to only require this for looking up references on individual revisions such as latest drafts) so we will not add an index there.

For MySQL and SQLite users, a full table scan of the `ReferencesIndex` model would be required.

### Management command for regenerating the index

A new management command called `rebuild_references_index` will be added that will re-scan the whole site and fill the `ReferencesIndex` table with up-to-date data. This would be required after the first update that introduces this feature, and after any bulk update/migration task that could affect references.
