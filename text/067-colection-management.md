# RFC 67: Instance-level Collection Management

* RFC: 67
* Author: Cynthia Kiser
* Created: 2021-03-16
* Last Modified: 2021-03-16

## Abstract

Permission to manage collections is currently controlled at the model level. As
a step on the way to fully separate multi-site installs, this RFC proposes
allowing groups to manage collections starting from any place in the collection
hierarchy.

In a multi-site installation, this would allow the superuser to create a site
and a base collection for that site. They could then delegate management of any
collections underneath that base collection to the site's main admin group.

Single site installs may or may not need to delegate collection management -
though it may be useful in larger sites to allow groups create private
collections within their section of the site's collection hierarchy

## Specification

### Permission changes

The collection management admin views currently use a ModelPermissionPolicy
which means they check the group permissions on the collection content type. We
need to change this to use a permission policy that checks the group permissions
on a specific section of the collection hierarchy.

We will support add, change, and delete permission types. Permissions will
propagate down the collection hierarchy - so giving a user 'add' permission on
the Root collection will give them 'add' permission everywhere.

Pages, and models that inherit from CollectionMember (e.g. Image and Document
models), have a concept of "Ownership" permissions that grants users with 'add'
permissions the right to also edit any instance they added. I think this is
unnecessarily complex for managing the collection hierarchy itself. There is an
existing CollectionPermissionPolicy that does not appear to be used anywhere
currently but may be appropriate for implementing this RFC.

The current collection management views do not allow the user to manipulate the
Root collection. This prevents users from deleting the entire collection
hierarchy. I propose we do similarly and only allow users to add/edit/delete
collections that are descendants of the collections on which they have
permissions.

### Admin UI changes

Collection permissions will move out of the "Object Permissions" section of the
group edit form and be inserted as it's own section name "Collection Management
Permissions" in the bottom part of the page near the Page, Document, and Image
permission fields. The UI will look like the documents and images lines but
offer add, change, and delete checkboxes.

### Documentation changes

*Current issues in docs*:

1. There does not appear to be a dedicated page explaining how to configure a
   group. The [Managing users and
   roles](https://docs.wagtail.io/en/stable/editor_manual/administrator_tasks/managing_users.html)_page
   only seems to cover creating users - with a small table explaining the
   capabilities of the Editor, Moderator, and Administrator (superuser) roles.
1. This note is at the bottom of [Collections](https://docs.wagtail.io/en/stable/editor_manual/documents_images_snippets/collections.html):

    Permissions set on ‘root’ apply to all collections, so a user with ‘edit’
    permission for images on root can edit all images; permissions set on other
    collections apply to that collection only.

    But that is not near docs on permissions for managing collections. It
immediately follows the section on Privacy settings for viewing items in a
collection.

We should add a page - or at least section - about configuring group permissions
and then move the note in item 2 to this section and update the note on the
regarding image editing to note that Privacy options for collections cascade in
much the same way Privacy options for pages do.

### Backwards compatibility

To make this change backwards compatible, we will include a data migration that
will grant any group that has permission to manage collections via the
ModelPermissionPolicy permission to fully manage the Root collection.

The Editor and Moderator groups created with 'wagtail start' do not have
permission to mange collections so we should not need to modify our start script.

## Open Questions

// Include any questions until Status is ‘Accepted’
