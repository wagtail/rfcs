# RFC 32: Extensible view restrictions

* RFC: 32
* Author: Matt Westcott
* Created: 2019-01-22
* Last Modified: 2019-01-22

## Abstract

Wagtail provides the ability to place view restrictions on pages and documents (at the collection level), requiring users to pass an authentication step before they are allowed to view that resource. As of Wagtail 2.4, three types of restriction are supported:

* Resource is password-protected by an ad-hoc shared password
* Resource is only available to logged-in users
* Resource is only available to users in specified groups

The logic for these restriction types is currently hard-coded into the Wagtail core and admin interface, and there is no supported way to add new restriction types. This RFC proposes an extension mechanism so that new restriction types can be defined within user code or third-party extensions, with no change to Wagtail required.

The primary motivation for this feature is to support permission rules that are too complex to express in Django's user / group model, such as: "this resource is only available to users who are members of an organisation that has an active subscription to a product with features X or Y". However, it could equally be used to implement restrictions that are unrelated to login accounts, such as:

* Resource is only available to users in a given IP range
* Resource is only available between 9am and 5pm each day
* Resource is only available over HTTPS

## Specification

### Models

`wagtail.core.models` provides two models `PageViewRestriction` and `CollectionViewRestriction`, both descending from the abstract model `BaseViewRestriction`, which represent a view restriction on a page and on a collection respectively. Currently, these models include a `restriction_type` field (one of `'password'`, `'login'` or `'groups'`) and `password` and `groups` fields to store additional parameters for password-based and group-based restrictions.

In the updated implementation, each restriction type will be implemented as a subclass of PageViewRestriction or CollectionViewRestriction via multi-table inheritance, with type-specific parameters stored in the subclass's specific table and a `content_type` parameter on the base table to identify the restriction type. Any concrete subclass of PageViewRestriction or CollectionViewRestriction will be automatically 'discovered' by Wagtail's view restrictions system, and exposed in the admin interface where appropriate - e.g. offered as an available restriction type when setting up a new view restriction.

The existing built-in restriction types (password, login and groups) will be reimplemented as subclasses of `PageViewRestriction` and `CollectionViewRestriction`, giving the following inheritance tree:

    BaseViewRestriction (abstract)
     |
     |- PageViewRestriction
     |   |
     |   |- PasswordPageViewRestriction
     |   |- LoginPageViewRestriction
     |   |- GroupPageViewRestriction
     |   '- (any custom view restriction types defined for pages)
     |
     '- CollectionViewRestriction
         |
         |- PasswordCollectionViewRestriction
         |- LoginCollectionViewRestriction
         |- GroupCollectionViewRestriction
         '- (any custom view restriction types defined for collections)

It is expected that the corresponding restriction type models for pages and collections (e.g. `PasswordPageViewRestriction` and `PasswordCollectionViewRestriction`) will have a considerable amount of shared logic and be implemented through a common abstract / mixin class, not shown in the above hierarchy.

> Q: This means that any external code that defines a new restriction type will generally have to define two classes - one for pages and one for collections. Furthermore, if a future version of Wagtail introduces a new kind of permissionable resource alongside pages and collections, the restriction type won't be available for that kind of resource until the external code is explicitly updated to support it. Wouldn't it be neater to define a single (possibly-abstract) class for the restriction type, and have it automatically apply to pages, collections and any other kinds of resources?
>
> A: No, it's better to have explicit handling for each kind of resource. Since the definition of what a restriction type can do has been left somewhat open-ended, we can't assume that the implementation will look the same for both pages and documents, or that a particular restriction type is meaningful for both pages and documents. For example, PasswordPageViewRestriction will support serving up a custom variant of the 'real' page as the password prompt view - e.g. displaying a preview paragraph in place of the full text - which can't be done for documents.

### Front-end behaviour

Subclasses of `PageViewRestriction` should implement a method `check(self, request, page)` which will be called just before serving a page which is covered by that view restriction, passing the `HttpRequest` and the `Page` instance. This method shall return either `None` (to allow the page to be served as normal) or an `HttpResponse` (which is served immediately to the user, cancelling the rest of the page serve workflow). This differs from the current implementation, where the `accept_request` method returns a boolean and the calling code (the `before_serve_page` hook in `wagtail.core.wagtail_hooks`) has to generate a suitable response in the case of `False` being returned.

Similarly, subclasses of `CollectionViewRestriction` should implement a method `check_document(self, request, document)` which will be called just before serving a document, and which returns either `None` or an `HttpResponse`.

> This method is named `check_document` rather than `check` to allow for images or other collection-based resources to support view restrictions as a possible future development; these would warrant a distinct "check" function on `CollectionViewRestriction`.

A number of helper methods / functions exist to support the built-in restriction types: `BaseViewRestriction.mark_as_passed` (which updates a field inside the user's session to keep track of password restrictions that have been cleared) and `require_wagtail_login` (which generates a redirect response to the configured front-end login page). As these are potentially useful in implementing custom view restriction types, they will be made available either as methods on one of the `BaseViewRestriction` / `PageViewRestriction` / `CollectionViewRestriction` classes, or a mixin class.

### Form handling

Within the 'Page Privacy' modal (accessible from the Privacy button in the top right of the page editor and explorer views) and the 'Collection Privacy' modal (accessible from Settings -> Collections -> Edit -> Privacy), each subclass of `PageViewRestriction` / `CollectionViewRestriction` will contribute an radio button to the "visibility" chooser, along with a (possibly empty) set of form fields to be shown when that radio button is selected. This requires each subclass to define the following attributes / methods:

* `option_label`: a class-level attribute containing the label for the radio button, as a string or lazy translation object, e.g. `_("Private, accessible with the following password")`
* `get_form_class()`: a class method which takes no arguments and returns a ModelForm subclass containing the custom fields specific to this restriction type
* `render_form(form)`: a class method which accepts an instance of this form class and returns an HTML rendering of it. `BaseViewRestriction` provides a default implementation of `render_form` which renders the form using [a generic Wagtail admin form styling](https://github.com/wagtail/wagtail/blob/8fa7a41011810696a7cfb83255091e91c441b23e/wagtail/admin/templates/wagtailadmin/generic/edit.html#L15-L32)
* `template`: a class-level attribute that specifies a template path for `render_form` to use instead of the default template

The view logic in `wagtail.admin.views.page_privacy` / `wagtail.admin.views.collection_privacy` will be responsible for constructing a form instance for each restriction type, populated or unpopulated as appropriate, with a distinct prefix for each form, and rendering each form in a container element so that it can be shown / hidden via Javascript when the appropriate radio button is checked. When the form is submitted, the view logic will instantiate the appropriate form for the currently selected radio button and call its `is_valid` / `save` methods so that the appropriate view restriction object is written back to the database.


## Open Questions
