# RFC 42: Data transfer between Wagtail installations

* RFC: 42
* Author: Matthew Westcott
* Created: 2019-09-26
* Last Modified: 2019-09-26

## Abstract

A Wagtail extension app to allow administrators to transfer selected pages, images, documents and other models between different installations of the same Wagtail project (for example, staging and production sites). The data transfer process will support the following features:

 * Identifying related database objects (such as images referenced on pages through foreign keys, rich text and StreamFields) that need to be transferred in order to fully replicate the requested items on the destination site
 * The ability to re-import items that were previously transferred, and update the existing records on the destination site where appropriate, rather than creating duplicates
 * When importing items that reference other objects through foreign keys, rich text or StreamFields, and those related items have themselves been imported and assigned a different primary key at the destination, it will handle remapping those references to the correct IDs
 * Importing the revision history of pages

The design of this app is inspired by the existing [wagtail-import-export](https://github.com/torchbox/wagtail-import-export) app, but substantially extends it.

## Specification

### Installation

The app will be installable by running a `pip install` command and adding to the project's `INSTALLED_APPS` setting.

### Behaviour

The app will add an 'Import' item to the Wagtail admin menu, available to users with administrator (superuser) access. This will present a form with the following fields:

 * Source site - a dropdown of other installations available to import data from
 * Source page - a chooser interface allowing the user to browse the page tree of the source site and select a root page to import - all descendant pages of that page will be included in the import.
 * Target page - a chooser interface allowing the user to select a page on the current installation which will become the parent of the incoming root page. This field will be disabled if the chosen source page is one that already exists on the current installation from a previous import; in this case the page will be updated in its current location, and descendant pages (if any) will be created relative to that location instead.

On submitting the form, the import will proceed as follows:

 * Starting with the selected root page, the destination site's page tree will be checked to see if an existing version of the page is already present there. If it is, the existing page will be updated with the data from the source site, and the relevant part of the page's revision history will be copied across; if not, a new page will be created as a child of the selected target page, with the full revision history copied across.
 * The status of the page as it exists on the source site (i.e. published or draft) will be copied to the destination site.
 * Any images or documents linked from the page (either from foreign key fields, within rich text, or within StreamField content) will be imported along with the page, if they do not already exist on the destination site. However, this does not apply to links to other pages - if the linked page does not already exist on the destination site, and is not part of the subtree being imported, it will become a broken link. Similarly, links and references to other site-specific models, such as snippets, will not cause those objects to be pulled across as part of the import unless this is explicitly enabled in the site configuration (see 'Configuration' below).
 * The above process is then repeated for every descendant page under the root page. Tree structure will be preserved - i.e. each page will be created under the appropriate parent page to match its position on the source site. The only exception is for pages that already exist on the destination (potentially having been moved to a new place in the tree in the meantime) - these will keep their existing location.

### Future functionality

The following features could potentially be added in a future version of the app, but we propose to leave them out of the initial development:

 * Selecting multiple pages as "Source page". This would require additional UI work, and since the proposed behaviour is to always descend the page tree (so that, for example, an entire news section can be imported by selecting the news index as the source page), it is anticipated that the most common use-cases for bulk importing will be covered as is.
 * Initiating an export from within the source site, as opposed to an Import workflow at the destination. In some respects this would offer a more natural user interface, as it could be incorporated into the page explorer as a new item within the 'More...' action menu against each page. However, this makes authentication more complicated - we would need to exchange user credentials between the sites to verify that the user has administrator access on the destination site.
 * Selecting non-page models to import. Since the app will need to support importing images, documents etc as a sub-task of importing pages, it wouldn't be a huge leap to provide "Import images", "Import documents" etc options alongside the facility for importing pages. However, each such addition would require further UI development.
 * Additional configuration options at import time, e.g. specifying whether or not child pages or linked images / documents should be imported. This would make the app considerably more complicated, and - given that there's effectively no upper bound on how complex the inter-object relations on a Wagtail site can get - it's doubtful whether users would be well served by being prompted to make these decisions at import time, rather than having them decided up-front by a developer.
 * Offline export via files. The current wagtail-import-export app includes the ability to export a section of the page tree as a JSON file, which can then be imported into the destination site - avoiding the need for a direct communication channel between the two instances. However, with the extra logic proposed here to identify items that already exist on the destination site and skip reimporting them where appropriate, it's likely that we'll benefit from full two-way communication between the two sites, so it makes sense to develop that method first.

### Data model

As standard in Wagtail, database objects (pages, images etc) are assigned sequential numeric IDs. When two installations of a Wagtail project exist, and are actively being edited in parallel, they will inevitably end up with the same ID being used for two different pieces of content between the two installations. When a piece of content is transferred from one installation to another, therefore, it will generally not retain its original ID from the source site - as that ID is already taken by another piece of content.

For the purposes of data transfer, each piece of transferred content will be assigned a universal ID (UID) that is shared by all installations where that content exists, and guaranteed to be unique to that piece of content across all instances. These UIDs will be generated using Python's [`uuid.uuid1`](https://docs.python.org/3/library/uuid.html#uuid.uuid1) function. This information will be used for two purposes:

 1. To identify when an incoming content object already exists on the destination site, from a previous import; in this case the existing item shall be overwritten, rather than creating a duplicate record.
 2. When handling a content object that contains references to other objects (such as images included on a page) that may have different IDs on the destination site as a result of a previous transfer, those objects can be looked up by UID and remapped to the correct ID as it exists on the destination site.

To avoid the need to modify Wagtail's core data model, the import-export app will define a new model / database table to store the mappings from UIDs to local installation-specific IDs:

    class IDMapping(models.Model):
        uid = models.CharField(primary_key=True, max_length=255)
        content_type = models.ForeignKey(ContentType, on_delete=models.CASCADE)
        local_id = models.CharField(max_length=255)
        content_object = GenericForeignKey('content_type', 'object_id')

### Configuration

The list of instances offered as import source sites will be configurable through the Django settings file. In cases where sites are locked down by basic authentication, IP range or similiar, it is the implementor's responsibility to ensure that each instance is able to make HTTP / HTTPS requests to its 'sibling' sites.

    WAGTAIL_IMPORT_SOURCE_INSTANCES = [
        {
            'label': "Staging",
            # Basic auth credentials included in the URL
            'base_url': 'https://mysite:mysite@staging.example.com/',
        },
        {
            'label': "Production",
            'base_url': 'https://production.example.com/',
        },
    ]

When an incoming content object contains references to other objects, the appropriate method for resolving those references is likely to vary depending on the object type and field type. For example:

 * A page containing an image gallery - the corresponding images should be looked up by UID and imported from the source site if they do not already exist on the destination site.
 * A page containing a tag field - tags should be looked up by name and created if they do not already exist on the destination site.
 * A page containing a list of "see also" page links, pointing to pages that may exist on the destination site already, may be part of the import dataset, or neither of the above - in this case, the links that cannot be matched by UID to an existing or incoming page should be omitted from the final imported page.
 * A page with a rich text field that contains links to other pages (that, again, may or may not exist at the destination or be part of the import) - in this case, any links that cannot be matched by UID will remain in the text as dead links (and a warning will be returned to the user performing the import).
 * A page with a required 'hero' link to another page, that does not exist on the destination and is not part of the import dataset - in this case, the import cannot proceed and must abort with an error (or, at least, skip the page in question).

(Note that in the latter three cases, the outcome is analogous to what would happen on a single Wagtail installation if the link's target page was deleted, depending on the foreign key's `on_delete` property - `CASCADE` in the case of the "see also" links, and `PROTECT` in the case of the hero link.)

The import app will ship with sensible defaults for Wagtail's built-in data types, but this isn't possible for custom project-specific models that may be linked from pages, such as snippets. For these, the app will provide a "registration" system similar to Wagtail's ModelAdmin and the Django admin, to configure how the importer handles specific models.


## Open Questions

 * What does the API for communicating between Wagtail instances look like?
 * How do we determine the order of database operations to minimise the occurrence of circular dependencies? (For example, importing a news index page with a "Featured news article" foreign key pointing to a child page; the child page cannot be created until the index page exists, and the "Featured news article" field cannot be populated until the child page exists. This may require creating the index page with the field set to null and going back to update it afterwards, or creating the base Page records for all imported pages first, so that the tree structure and IDs exist before we start importing the specific page type records.)

