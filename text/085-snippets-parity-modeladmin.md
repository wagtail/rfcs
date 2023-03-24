# RFC 85: Snippets parity with ModelAdmin

* RFC: 85
* Author: Sage Abdullah
* Created: 2023-03-24
* Last Modified: 2023-07-07

> The following RFC is based on [an existing GitHub Discussion][snippets-modeladmin-discussion].
>
> Throughout this RFC, the term "Snippets" with a capital "S" refers to the Wagtail feature, while "snippet(s)" with a lowercase "s" refers to the Django model.

## Abstract

In the [Supercharging Snippets][supercharging-snippets] discussion, we had planned to make Snippets the "definitive" way to edit Django models in the Wagtail admin. We began the work by progressively enhancing the capabilities of snippets to become nearly as powerful as pages. Since then, those features have been implemented in Wagtail.

Currently, we still have two primary ways to edit Django models in the Wagtail admin: [Snippets][snippets] and [ModelAdmin][modeladmin]. Based on previous discussions, the long-term goal is to make Snippets up to par with ModelAdmin, before we eventually "detach" ModelAdmin into a separate package.

To achieve that goal, we need to figure out which ModelAdmin features we'd like to reimplement in Snippets, which features we'd like to leave out, and how we're going to proceed with the implementation and deprecation. This RFC serves as the main place to discuss those details.

## Basic ground for parity

Up until Wagtail 4.0, registering a snippet is primarily done through the `register_snippet` decorator on the Django model.

```python
@register_snippet
class Advert(models.Model):
    title = models.CharField(max_length=255)
    url = models.URLField()
    text = models.TextField()
```

In Wagtail 4.1 and later improved in Wagtail 5.0, we added the ability to customise the `ViewSet` class to use for a given snippet. Along with this, we introduced another way to register a snippet, by calling `register_snippet` as a function inside `wagtail_hooks.py`.

```python
class AdvertViewSet(SnippetViewSet):
    model = Advert
    list_display = ("title", "url")


register_snippet(AdvertViewSet)
```

This method provides a better separation of concerns between the model and the admin views. It also aligns with the way we register Wagtail features and customisations, including ModelAdmin. As such, this method will be the recommended way to register a snippet.

The `SnippetViewSet` acts as the Snippets-equivalent for the `ModelAdmin` class, which will be the main point of comparison for the parity between the two in this document.

## ModelAdmin features to reimplement in Snippets

The following subsections summarise the currently documented customisation points in ModelAdmin for features that we would like to reimplement or have already reimplemented in Snippets. Features that we would like to leave out or undecided will be listed later in this document.

### [Customising the base URL path](https://docs.wagtail.org/en/stable/reference/contrib/modeladmin/base_url.html)

Support was added in [#10235][10235], which further expands the customisation by adding the following attributes to `SnippetViewSet`:

- `base_url_path`
- `admin_url_namespace`
- `chooser_base_url_path`
- `chooser_admin_url_namespace`

### [Customising the menu item](https://docs.wagtail.org/en/stable/reference/contrib/modeladmin/menu_item.html)

Support was added in [#10330][10330], which adds the following `ModelAdmin` attributes to `SnippetViewSet` with the same behaviour unless otherwise noted:

- [x] **`ModelAdmin.menu_item_name`**
- [x] **`ModelAdmin.menu_label`**
- [x] **`ModelAdmin.menu_order`**
- [x] **`ModelAdmin.menu_icon`**

  Supported via `SnippetViewSet.icon`. The attribute name is `icon` instead of `menu_icon` as it is used throughout the admin views (see [#10178][10178]), not just the menu item. A separate menu icon can be defined by overriding `SnippetViewSet.get_menu_icon`.

- [x] **`ModelAdmin.add_to_settings_menu`**
- [x] **`ModelAdmin.add_to_admin_menu`**

  Unlike ModelAdmin, this defaults to `False` for Snippets, as traditionally we have the top-level "Snippets" menu item instead. If all snippets have `add_to_admin_menu` set to `True`, the "Snippets" menu item will be hidden.

### [Customising `IndexView` - the listing view](https://docs.wagtail.org/en/stable/reference/contrib/modeladmin/indexview.html)

- [x] **`ModelAdmin.list_display`**

  - [x] **`{Model,ModelAdmin}.attribute.admin_order_field`**
  - [x] **`{Model,ModelAdmin}.attribute.short_description`**

  This is partially implemented in Snippets in [#9147][9147] through `SnippetViewSet.list_display` and `IndexView.list_display`. Unlike ModelAdmin that looks for the attributes on both the model and the `ModelAdmin` class, we only look for attributes/methods on the model and not the `SnippetViewSet`.

  In `SnippetViewSet`, this attribute also supports instances of the `wagtail.ui.tables.Column` class, which provides better control of the column's behaviour, such as the corresponding field used for sorting.

- [x] **`ModelAdmin.list_filter`**

  A similar support has been implemented in [#10256][10256], but it uses django-filter under the hood, similar to our `ReportView` implementation.

  Wagtail's ModelAdmin uses `django.contrib.admin`'s filters, and changes in Django may break our ModelAdmin filters, as shown by [#10209][10209] and [#10242][10242]. Thus, we have a good reason to move forward with django-filter.

  In addition, `SnippetViewSet` also allows customising the filters through `SnippetViewSet.filterset_class` for more advanced use cases.

- [x] **`ModelAdmin.list_export`**

  Implemented in [#10626][10626].

  - [x] **`ModelAdmin.export_filename`**

- [x] **`ModelAdmin.search_fields`**

  Implemented in [#10290][10290].

- [x] **`ModelAdmin.ordering`**

  Implemented in [#10276][10276].

- [x] **`ModelAdmin.list_per_page`**

  Implemented in [#10241][10241].

- [x] **`ModelAdmin.get_queryset(request)`**

  Implemented in [#10275][10275].

- [x] **`ModelAdmin.index_template_name`**

  Implemented in [#10271][10271] along with the other views.

- [x] **`ModelAdmin.index_view_class`**

  Implemented in [#8422][8422] along with the other views.

### [Customising `CreateView`, `EditView` and `DeleteView`](https://docs.wagtail.org/en/stable/reference/contrib/modeladmin/create_edit_delete_views.html)

- [x] **Changing which fields appear in `CreateView` & `EditView`**

  Implemented in [#10299][10299].

  - [x] **`ModelAdmin.get_edit_handler`**
  - [x] **`ModelAdmin.form_fields_exclude`**

    Implemented as `SnippetViewSet.exclude_form_fields` to align with the existing attribute in `ModelViewSet`.

- [x] **`ModelAdmin.create_template_name`**
- [x] **`ModelAdmin.create_view_class`**
- [x] **`ModelAdmin.edit_template_name`**
- [x] **`ModelAdmin.edit_view_class`**
- [x] **`ModelAdmin.delete_template_name`**
- [x] **`ModelAdmin.delete_view_class`**

### [Enabling & customising `InspectView`](https://docs.wagtail.org/en/stable/reference/contrib/modeladmin/inspectview.html)

Implemented in [#10621][10621].

- [x] **`ModelAdmin.inspect_view_fields`**
- [x] **`ModelAdmin.inspect_view_fields_exclude`**
- [x] **`ModelAdmin.inspect_template_name`**
- [x] **`ModelAdmin.inspect_view_class`**

### Template overrides

Implemented in [#10271][10271], which allows templates to be customised by creating the templates in the following directories.

1. `templates/wagtailsnippets/snippets/{app_label}/{model_name}/`
2. `templates/wagtailsnippets/snippets/{app_label}/`
3. `templates/wagtailsnippets/snippets/`

In addition, the `wagtailsnippets/snippets/` prefix can be customised by overriding the `SnippetViewSet.template_prefix` attribute. Specifying `{foo}_template_name` attribute for certain views that are supported in ModelAdmin is also supported in Snippets.

## ModelAdmin features we intentionally leave out

### Using ModelAdmin to manage Page models

Relevant docs: [**Customising `ChooseParentView`**](https://docs.wagtail.org/en/stable/reference/contrib/modeladmin/chooseparentview.html).

ModelAdmin allows the registration of Page models, but the main purpose is to create custom page listing views. The create and edit views are not supported, as there are page-specific operations in those views that are best handled by Wagtail's page views. For this reason, registering a Page model as a snippet is not a use case we want to support. Instead, we will introduce a new "treeless" listing view for pages, as outlined in [RFC 082][rfc-082] and the [Universal Listings discussion][universal-listings].

### Customisation of index view table rows and columns

ModelAdmin has a number of APIs that allow customisation of the index view table rows and columns. Meanwhile, we use Wagtail's generic tables UI framework for Snippets, which has its own set of APIs. We are gradually moving towards using the generic tables UI framework throughout the admin, so we should not introduce new APIs that diverge from that goal.

The following APIs will not be reimplemented in Snippets. The same effect can be achieved by using the generic tables UI framework, generally by implementing a `wagtail.ui.tables.Column` subclass and using it in `list_display`. Any further customisations can be done by overriding the index view template and/or class.

- [ ] **`ModelAdmin.get_extra_attrs_for_row(obj, context)`**

  Returns a dictionary of extra attributes to be added to the `<tr>` tag.

- [ ] **`ModelAdmin.get_extra_class_names_for_field_col(obj, field_name)`**

  Returns a list of CSS classes for the column of the field for the given object (row).

- [ ] **`ModelAdmin.list_display_add_buttons`**

  The column where the buttons will be shown. Currently unsupported in snippets, but can be achieved similarly by making use of the `SnippetTitleColumn` class and overriding `IndexView.get_columns()` if necessary.

- [ ] **`wagtail.contrib.modeladmin.mixins.ThumbnailMixin`**

  Allows showing a thumbnail image by specifying `"admin_thumb"` in `list_display`.

  - [ ] **`ModelAdmin.thumb_image_field_name`**

  The `ForeignKey` field to `wagtailimages.Image`

  - [ ] **`ModelAdmin.thumb_image_width`**

  Thumbnail width, defaults to `50`.

  - [ ] **`ModelAdmin.thumb_classname`**

  CSS class name to be added to the `<img>` element, defaults to `"admin-thumb"`.

  - [ ] **`ModelAdmin.thumb_col_header_text`**

  Column header text for the thumbnail, defaults to `"image"`.

  - [ ] **`ModelAdmin.thumb_default`**

  Fallback image if missing, can be from the static files or an external URL.

### Custom CSS and JS

ModelAdmin supports inserting custom extra CSS and JS files into the admin via dedicated attributes on the `ModelAdmin` class. This is not a pattern we want to encourage, as it makes it difficult to maintain a consistent UI across the admin. For JavaScript customisations, we are moving towards using Stimulus controllers as covered in [RFC 078][rfc-078]. Even so, adding custom CSS and JS can still be achieved by overriding the relevant templates and/or views.

- [ ] **`ModelAdmin.index_view_extra_css`**
- [ ] **`ModelAdmin.index_view_extra_js`**
- [ ] **`ModelAdmin.form_view_extra_css`**
- [ ] **`ModelAdmin.form_view_extra_js`**
- [ ] **`ModelAdmin.inspect_view_extra_css`**
- [ ] **`ModelAdmin.inspect_view_extra_js`**

### Helper classes

Helper classes encapsulate the logic that is commonly used across views in ModelAdmin. Instead of splitting the logic into ad hoc helper classes, centralising it in the `SnippetViewSet` and delegating specific mechanisms to existing solutions within Wagtail/Django would be a better approach in terms of maintainability and developer experience. Thus, we will not be implementing helper classes in Snippets.

There are three types of helper classes in ModelAdmin:

- [ ] **`ModelAdmin.url_helper_class`**

  Helps with the consistent generation, naming and referencing of URLs.

  We already have the `get_urlpatterns()` and `get_url_name()` methods on the `SnippetViewSet` that come from Wagtail's base `ViewSet` class to manage URLs of Snippets views. The URL names can then be used with Django's `reverse()` function to generate URLs.

- [ ] **`ModelAdmin.permission_helper_class`**

  Helps with ensuring only users with sufficient permissions can perform certain actions, or see options to perform those actions.

  With Snippets, we use an instance of the `ModelPermissionPolicy` class exposed as `self.permission_policy` on the `SnippetViewSet`. Subclasses can override this attribute to use a custom permission policy. Permission policies are the standard mechanism for managing permissions in Wagtail.

- [ ] **`ModelAdmin.button_helper_class`**

  With the help of the other two helper classes, this class helps with the generation of buttons for use in a number of places. This class mainly helps with the creation of buttons on the index view, but it can be used in other views as needed. Overriding each of the default buttons can be done by overriding the `foo_button()` method on the helper class. To add custom buttons, the undocumented `get_buttons_for_obj` method must be overridden.

  For Snippets, we have the existing [`register_snippet_listing_buttons`][register_snippet_listing_buttons] and [`construct_snippet_listing_buttons`][construct_snippet_listing_buttons] hooks. Due to the nature of hooks, these customisations are done directly in `wagtail_hooks.py`, separate from the `SnippetViewSet`.

  The hooks may be enough for most use cases. That said, they have been around since before `SnippetViewSet` was introduced, so we may also consider replacing them with a more cohesive approach within the `SnippetViewSet` in the future.

- [ ] **`ModelAdmin.search_handler_class`**

  The class that handles search, subclass of `wagtail.contrib.modeladmin.helpers.search.BaseSearchHandler`. Even though the attribute uses the term `handler`, the module location suggests that is part of the helper classes.

  The main purpose of this class is to allow switching between Wagtail's search backend and the Django ORM. Instead of providing this via a helper class, [#10290][10290] added support to specify the search backend via the `search_backend_name` attribute and will fall back to the Django ORM if the attribute is unset.

  - [ ] **`ModelAdmin.extra_search_kwargs`**

    This attribute is not part of the helper class but is related to it. This specifies the keyword arguments to be passed to `search_handler_class.search_queryset()`, e.g. `{"operator": OR}`.

    As we will not support helper classes, we will not support this attribute. Any further customisations to the search mechanism can be done by overriding the index view.

### Other attributes

- [ ] **`ModelAdmin.empty_value_display`**

  String to display in place of empty values (e.g. `None`, `""`, `[]`). Defaults to `"-"`.

  This is only used in the index and inspect views. For the index view, this can be delegated to the `Column` class of the tables UI framework. For inspect view, we will not support it, but the same effect can be achieved by implementing a `get_foo_display()` method on the model, which is an established pattern in Django.

- [ ] **`ModelAdmin.get_empty_value_display()`**

  As with `empty_value_display`, but can return different strings based on the `field_name`.

  Will not be supported for the same reason as `empty_value_display`.

- [ ] **`ModelAdmin.get_ordering(request)`**

  Like `ModelAdmin.ordering`, but can be customised per request.

  The `ordering` attribute covers the default ordering of the index view. Instances of Wagtail's `Column` class that can be used in the `SnippetViewSet.list_display` attribute can specify the corresponding database field for ordering, which covers most use cases of per-request ordering. For more complex use cases, the index view can be overridden.

  As explained in [a review comment][get-ordering-review], we will not support this method.

- [ ] **`ModelAdmin.prepopulated_fields`**

  A dict mapping prepopulated fields to a tuple of fields to prepopulate from, useful for slug generation on Page models.

  This feature uses a custom JavaScript file that concatenates and slugifies the values of the specified fields into a target field. The `TitleFieldPanel` added in [#10568][10568] demonstrates a different approach to this feature that builds on top of Wagtail's Panels API and it can be used for Snippets. In addition, there are requests for other similar enhancements ([#161][161], [#2172][2172], [#7417][7417]) that would be better implemented as Wagtail-wide solutions. Thus, we will not support this attribute.

### Deprecation timeline of ModelAdmin

Now that most features of ModelAdmin have been reimplemented in Snippets, and the remaining features will not be supported, we are aiming to start the deprecation process of ModelAdmin in the next feature release of Wagtail, i.e. Wagtail 5.1 (to be released in August 2023) at the time of writing.

The deprecation timeline will be as follows:

- Pre-release of Wagtail 5.1 (July 2023): Release the `wagtail.contrib.modeladmin` module as a separate package. This will be a copy of the `wagtail.contrib.modeladmin` module.
  - Issues in the `wagtai/wagtail` repository that are related to ModelAdmin will be moved to the new repository. A list of existing PRs for ModelAdmin will be made on the new repository, with the original PRs closed and contributors will be asked to resubmit their PRs to the new repository. Security fixes to `wagtail.contrib.modeladmin` may still be accepted in the `wagtail/wagtail` repository as deemed necessary.
- Wagtail 5.1 (August 2023): Deprecate `wagtail.contrib.modeladmin`.
  - A migration guide will be provided for users who want to migrate from ModelAdmin to Snippets or the external package.
- Wagtail 6.0 (TBD): Remove `wagtail.contrib.modeladmin`.
  - Documentation of ModelAdmin within Wagtail's documentation will be redirected to the external package documentation.

Considering that ModelAdmin is used by many Wagtail users, we may opt for a longer deprecation timeline. In addition, as Wagtail future version numbers are provisional, the Wagtail 6.0 release may or may not be the release after Wagtail 5.1. Per our deprecation policy, we will ensure that there are at least two feature releases between the deprecation and removal of ModelAdmin. As a result, if Wagtail 6.0 is released after Wagtail 5.1, ModelAdmin will be removed in Wagtail 7.0.

## Future enhancements

There are some possible improvements that are not within the scope of the RFC but may be considered in the future.

- Control the registration of CRUD views and the chooser separately

  There have been requests for the ability to register snippets without having the chooser registered, and vice versa, register the chooser but without the CRUD views. The work in [#10216][10216] brought us closer to this goal, but parts of the CRUD views still assume that the chooser is registered.

- Turn off specific views

  There have also been requests to turn off specific views. While it is possible to override the URL registration of the views, the current views implementation rely on the other views being available. For example, the ability to turn off the index, create, and delete views can be useful to enforce a singleton snippet within the admin.

## See also

- [Snippets vs ModelAdmin parity #10206][snippets-modeladmin-discussion]
- [Supercharging Snippets #8609][supercharging-snippets]
- [Ability to add custom icon to Snippet models #4095][4095]
- [Comment on Supercharging Snippets about rebuilding ModelAdmin as part of Wagtail core][comment-on-supercharging-snippets]

[snippets-modeladmin-discussion]: https://github.com/wagtail/wagtail/discussions/10206
[supercharging-snippets]: https://github.com/wagtail/wagtail/discussions/8609
[snippets]: https://docs.wagtail.org/en/v4.2.1/topics/snippets.html
[modeladmin]: https://docs.wagtail.org/en/v4.2.1/reference/contrib/modeladmin/index.html
[161]: https://github.com/wagtail/wagtail/issues/161
[2172]: https://github.com/wagtail/wagtail/issues/2172
[7417]: https://github.com/wagtail/wagtail/issues/7417
[4095]: https://github.com/wagtail/wagtail/issues/4095
[8422]: https://github.com/wagtail/wagtail/pull/8422
[9147]: https://github.com/wagtail/wagtail/pull/9147
[10178]: https://github.com/wagtail/wagtail/pull/10178
[10209]: https://github.com/wagtail/wagtail/pull/10209
[10216]: https://github.com/wagtail/wagtail/pull/10216
[10235]: https://github.com/wagtail/wagtail/pull/10235
[10241]: https://github.com/wagtail/wagtail/pull/10241
[10242]: https://github.com/wagtail/wagtail/pull/10242
[10256]: https://github.com/wagtail/wagtail/pull/10256
[10271]: https://github.com/wagtail/wagtail/pull/10271
[10275]: https://github.com/wagtail/wagtail/pull/10275
[10276]: https://github.com/wagtail/wagtail/pull/10276
[10290]: https://github.com/wagtail/wagtail/pull/10290
[10299]: https://github.com/wagtail/wagtail/pull/10299
[10330]: https://github.com/wagtail/wagtail/pull/10330
[10568]: https://github.com/wagtail/wagtail/pull/10568
[10621]: https://github.com/wagtail/wagtail/pull/10621
[10626]: https://github.com/wagtail/wagtail/pull/10626
[comment-on-supercharging-snippets]: https://github.com/wagtail/wagtail/discussions/8609#discussioncomment-5006831
[register_snippet_listing_buttons]: https://docs.wagtail.org/en/stable/reference/hooks.html#register-snippet-listing-buttons
[construct_snippet_listing_buttons]: https://docs.wagtail.org/en/stable/reference/hooks.html#construct-snippet-listing-buttons
[rfc-078]: https://github.com/wagtail/rfcs/pull/78
[rfc-082]: https://github.com/wagtail/rfcs/pull/82
[get-ordering-review]: https://github.com/wagtail/wagtail/pull/10276#pullrequestreview-1385503020
[universal-listings]: https://github.com/wagtail/wagtail/discussions/10446
