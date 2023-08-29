# RFC 87: Enhanced viewsets

* RFC: 87
* Author: Matthew Westcott
* Created: 2023-08-17
* Last Modified: 2023-08-17

## Abstract

This RFC proposes several improvements to the Wagtail admin's [viewsets](https://docs.wagtail.org/en/stable/reference/viewsets.html) mechanism - namely, a formal way to declare the attributes that are valid on each view in a viewset to minimise the amount of redundant boilerplate in a viewset definition, and a way for the viewset to define methods to be shared across multiple views, rather than only being able to share configuration settings. It is hoped that these improvements will make viewsets into a useful tool of value to the wider Django community, and so we also outline a path towards shipping viewsets as a general-purpose Django package.

## Specification

Viewsets provide an effective way to define a group of related views (and additional assets such as widgets and menu items) with shared properties. A good example is the set of CRUD views for a given model - properties shared between them might include the model itself, the form class (shared between the add and edit views), URL names (so that the listing view can link to each object's edit view, the edit view's POST action can redirect back to the listing view, and so on), icons, and text labels ("Edit this X"). Defining these through a viewset, rather than on the individual views, reduces duplication and improves maintainability. However, viewsets currently have a number of deficiencies, outlined below.

### Duplication of knowledge about valid view attributes

The viewset is responsible for constructing each of its views by passing the appropriate set of keyword arguments from its own configuration settings to the `as_view` method:

```python
    @property
    def add_view(self):
        return self.add_view_class.as_view(
            model=self.model,
            permission_policy=self.permission_policy,
            form_class=self.get_form_class(),
            index_url_name=self.get_url_name("index"),
            add_url_name=self.get_url_name("add"),
            edit_url_name=self.get_url_name("edit"),
            header_icon=self.icon,
        )
```

Passing the correct subset of the viewset's attributes is critical. For example, out of the views handled by Wagtail's `ModelViewSet`, the `edit_url_name` argument is accepted by the listing view (so that it can link to each item's edit view), the 'add' view (so that the success message on creation can incorporate a link to edit the newly-created item), the 'edit' view (so that the form can POST back to its own URL), but _not_ the 'delete' view. Failing to pass an argument that the view relies on will cause that view to work incorrectly; conversely, passing an argument that is not expected by the view will raise an error, since the standard behaviour of `as_view` is to reject any keyword arguments that do not exist as attributes of the view class.

This means that the viewset has to know a lot of low-level detail about each view - knowledge which we would rather keep contained within the view's implementation. For example, suppose we were to change the behaviour of the 'delete' view so that the "no, don't delete this" option on the confirmation screen links back to the edit view. Since the view now accepts the `edit_url_name` argument, the viewset would need to be updated to pass it in. This is a lot of work for a simple change, and it's easy to forget to do it.

#### Proposal

The ViewSet class shall define a `view_attributes` property, which returns a dict of all properties that may be passed to any view in the set, and their values. When calling a view class's `as_view` function, the dict of keyword arguments will be formed by filtering `view_attributes` to the keys that are valid for that view - namely, ones for which `hasattr(view_class, key)` returns True. This would correspond to the following code:

```python
class ModelViewSet(ViewSet):
    add_view_class = generic.CreateView

    @cached_property
    def view_attributes(self):
        return {
            "model": self.model,

            # attributes on views do not necessarily have to match the
            # attribute names on the viewset
            "menu_icon": self.icon,
            "header_icon": self.icon,

            "index_url_name": self.get_url_name("index"),
            "add_url_name": self.get_url_name("add"),
            "edit_url_name": self.get_url_name("edit"),
            "delete_url_name": self.get_url_name("delete"),

            # ...
        }

    @property
    def add_view_kwargs(self):
        return {
            key: value
            for key, value in self.view_attributes.items()
            if hasattr(self.add_view_class, key)
        }

    @property
    def add_view(self):
        return self.add_view_class.as_view(**self.add_view_kwargs)
```

To avoid duplicating this logic for every view in the viewset, the base `ViewSet` class will have an `__init_subclass__()` method (or failing that, a metaclass) that builds the definitions of `add_view_class`, `add_view_kwargs` and `add_view` (and the corresponding definitions for all other views) from a `views` attribute. The `ModelViewSet` definition will thus become:

```python
class ModelViewSet(ViewSet):
    views = {
        'index_view': generic.IndexView,
        'add_view': generic.CreateView,
        'edit_view': generic.EditView,
        'delete_view': generic.DeleteView,
    }

    @cached_property
    def view_attributes(self):
        return {
            "model": self.model,
            "menu_icon": self.icon,
            "header_icon": self.icon,
            "index_url_name": self.get_url_name("index"),
            "add_url_name": self.get_url_name("add"),
            "edit_url_name": self.get_url_name("edit"),
            "delete_url_name": self.get_url_name("delete"),
            # ...
        }
```

### Inability to define shared methods

The viewset is currently limited to defining shared configuration settings. However, there are many cases where it would be useful to define shared methods as well. For example, many of the views within `ChooserViewSet` make use of a `get_chosen_response_data` method (defined in `ChosenResponseMixin`) that converts the chosen object into a JSON representation to be returned to the chooser widget; we may wish to override this method to add a thumbnail image URL to the JSON, for example. Currently, the cleanest way to do this would be to subclass `ChosenResponseMixin` with the new method, subclass each of the relevant views to use the new mixin class, and then update the viewset to use the new view classes. This is a lot of work, and requires inside knowledge of which views require the override. It would be far better to be able to define the method on the viewset itself, and have it automatically shared between all of the views that need it.

While it may be possible to inject methods into views via the `as_view` mechanism - since there's no fundamental difference between a method and any other object attribute - this would be rather hacky.

#### Proposal

Before calling `as_view` on a view class, the viewset will check for any methods defined on itself that correspond to overrideable methods of the view. If any are found, it will dynamically create a subclass of the view class that overrides the method with the viewset's version, and invoke `as_view` on that new subclass. For example, a mechanism for overriding the `get_edit_url` of IndexView might look like this:

```python
class ModelViewSet(ViewSet):
    index_view_class = generic.IndexView

    @property
    def index_view_derived_class(self):
        methods = {}
        if hasattr(self, 'get_edit_url'):
            methods['get_edit_url'] = lambda view, instance: self.get_edit_url(instance)

        if methods:
            return type(
                self.index_view_class.__name__,
                (self.index_view_class,),
                methods,
            )
        else:
            return self.index_view_class

    @property
    def index_view(self):
        return self.index_view_derived_class.as_view(**self.index_view_kwargs)
```

As before, this pattern will be encapsulated in the base ViewSet class, so it will not be necessary to implement it individually for each view. Most likely, `ModelViewSet` will specify a list of method names that are valid to be overridden, and the base ViewSet class will implement the above logic based on that list. Care will need to be taken to account for methods across views that have similar functionality but different signatures - for example, `get_edit_url` on `IndexView` takes an `instance` argument, whereas `get_edit_url` on `EditView` currently retrieves `self.object` instead.

### Viewsets as a standalone Django package

Nothing in this implementation is fundamentally tied to Wagtail. Indeed, similar patterns in projects such as [Neapolitan](https://noumenal.es/neapolitan/) and [Django REST framework](https://www.django-rest-framework.org/api-guide/viewsets/) show that this has potential value for the wider Django community.

Once the implementation within Wagtail has stabilised, we envisage extracting it as a standalone package published on PyPI. Wagtail's base ViewSet class does have some Wagtail-specific functionality not covered here, such as registering menu items - it is anticipated that these would remain within Wagtail as a `WagtailAdminViewSet` subclass of the standalone package's ViewSet class, and all viewsets within Wagtail would inherit from that.

## Open Questions

// Include any questions until Status is ‘Accepted’
