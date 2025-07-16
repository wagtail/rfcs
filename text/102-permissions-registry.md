# RFC 102: Permissions registry

* RFC: 102
* Author: Sage Abdullah
* Created: 2024-07-30
* Last Modified: 2024-08-02

## Abstract

This RFC proposes a new permissions registry that maps a model class to a permission configuration. Everywhere a permission test is performed in Wagtail, the registry will be consulted to determine the configuration to use for the model or model instance in question. The registry will make it easier for developers to access the permission configuration of a model from anywhere in the code. It will also allow developers to define custom permission logic for both custom models and Wagtail's built-in models, to be used in place of the default.

This is a follow-up to [RFC 92](https://github.com/wagtail/rfcs/pull/92) with a different approach to how the generic permission tests are implemented.

## Requirements

Based on the research done in RFC 92, we need a way to do permission tests from anywhere in the codebase, provided that you have the necessary information to perform the test. The following requirements are derived from the research:

1. Given a model class, we can get the permission policy for the model.
   - Use case: performing efficient permission-informed database queries for that model
2. Given a user instance, a model instance, an action, and arbitrary arguments based on the action, we can test whether the user can perform the action for that instance.
   - Use case: most permission tests, e.g. "can delete"
   - Use case: permission tests that require additional arguments, e.g. "can move to"
3. Given a user instance, a model class, an action, and arbitrary arguments based on the action, we can test whether the user can perform the action for that model.
   - Use case: permission tests that do not require an instance, e.g. "can create" for non-page models
   - Built-in actions that only test against the model likely never require additional arguments, but developers may define custom actions that do

## Specification

We introduce a new registry called `PermissionRegistry` that:
1. maps a model class to an instance of a `BasePermissionPolicy` subclass
2. maps a model class and an action string to a new `PermissionTester` subclass

```python
class PermissionRegistry:
    def register_policy(model: type[Model], policy: BasePermissionPolicy):
        ...

    def get_policy(model: type[Model]) -> BasePermissionPolicy:
        ...

    def register_tester_class(model: type[Model], action: str, tester_class: type[BasePermissionTester]):
        ...

    def get_tester_class(model: type[Model], action: str) -> type[BasePermissionTester]:
        ...

    def get_tester(user: AbstractUser, action: str, obj: Union[type[Model], Model], **kwargs) -> BasePermissionTester:
        # Allow using a model class or a model instance
        if isinstance(obj, type):
            model = obj
            obj = None
        else:
            model = obj.__class__

        kwargs.pop("model", None)  # Ensure the model is not passed as a keyword argument
        tester_class = self.get_tester_class(model, action)
        return tester_class(user, model, obj=obj, **kwargs)

    def test(user: AbstractUser, action: str, obj: Union[type[Model], Model], **kwargs) -> bool:
        return self.get_tester(user, action, obj, **kwargs).test()


permission_registry = PermissionRegistry()
```

The registry will respect the MRO of the model class – so if a model is subclassed, the policy of the most specific class will be used. For example, Wagtail's default permission configuration for pages will be registered with the `Page` model, but specific page types can register their own configuration that will be used in place of the default.

The registry will also respect the order of registration, so if the same model is registered multiple times, the last registration will be used. This allows developers to override the default configuration for all models of a certain type, e.g. to override the default configuration for all pages.

Once the registry is implemented, its instance will live in the `wagtail.permissions` module.

We also introduce a generic `{% permissions %}` template tag as a shorthand for `permission_registry.test()` within templates.

```html+django
{% permissions user 'edit' page as can_edit %}
{% if can_edit %}
    ...
{% endif %}
```

To ensure its deliverability, the work will be split into separate milestones that can be achieved in different releases.

### Model to permission policy mapping

A permission policy class is a subclass of `BasePermissionPolicy` that provides methods for performing permission-informed database queries of a model. It is intended to only provide basic permission checks, such as against the existence of corresponding Django `Permission` objects, and not to perform business logic checks. It also implements caching of permission objects on the user object to avoid repeated database queries.

The registry will have a `register_policy(model, policy)` method to register the mapping from a model class to a permission policy instance.

The aim is to have a "canonical" permission policy instance for each model registered within Wagtail.

For some built-in models, we already have them defined in [`wagtail.permissions`](https://github.com/wagtail/wagtail/blob/stable/6.2.x/wagtail/permissions.py) and `permissions` module inside other apps (e.g. [`wagtail.documents.permissions`](https://github.com/wagtail/wagtail/blob/stable/6.2.x/wagtail/documents/permissions.py)). The main problem with this approach is that they are defined at the top-level of the module. This means they are evaluated at import-time and cannot be overridden by developers.

For custom models, a default `ModelPermissionPolicy(model)` is usually defined in the view or the viewset. The main problem with this approach is that it creates unnecessary coupling between permissions and views. This is not ideal, as permission tests may be performed outside of the view code (e.g. in a background task, a hook, or generic code for different models). In such cases, we have to reach out to the viewset to get the permission policy, e.g. `model.snippet_viewset.permission_policy`. Doing so is impossible for models registered via `ModelViewSet`, as you can have multiple viewsets registered for the same model, so there is no "canonical" viewset to get the permission policy from.

The registry will have a `get_policy(model)` method that returns the permission policy instance for the given model.

To make policy overrides possible, we need to make sure the `get_policy(model)` method is used at request time whenever the model's policy is needed. This requires refactoring any code that uses permission policies defined at import time, e.g. views that set `permission_policy` as a class attribute or during their `.as_view()` instantiation in `ViewSet.construct_view()`.

You can register a permission policy like the following:

```python
from wagtail.permissions import permission_registry


class CustomPermissionPolicy(ModelPermissionPolicy):
    # Some customisation here
    ...


permission_registry.register_policy(model, CustomPermissionPolicy(model))
```

You can get the policy at request time like the following:

```python
from wagtail.permissions import permission_registry


class EditView(...):
    @cached_property
    def permission_policy(self):
        return permission_registry.get_policy(self.model)
```

At this point, we can conclude the first milestone, i.e. a Wagtail release can be made without having to implement the rest of the RFC. The following goals will be achieved:
- Ability to have custom permission policies, including for built-in models
- Elimination of tech debt where we access the permission policy through the viewset
- Basic version of the permission registry as the foundation for the next part of the implementation

### Model to permission tester mapping

The `BasePermissionTester` class is a new interface for defining the permission test of a specific action.

```python
class BasePermissionTester:
    def __init__(self, user, model, obj=None, **kwargs):
        self.user = user
        self.model = model
        self.obj = obj

    def test(self) -> bool:
        raise NotImplementedError
```

It accepts any arguments that are needed to perform the test. In most cases, these will be the user and the object being tested, but some testers may only need the model and not the instance. Different actions may require different arguments – we cannot enforce a specific signature for the `__init__` method, so we use keyword arguments to make their usage explicit.

The `test` method requires no arguments and returns a boolean indicating whether the user has the permission to perform the action on the object.

This interface acts as a replacement to the `PagePermissionTester` class. Instead of having a single class that handles all the permission tests for a model, we will have multiple classes, each handling a specific action. This will make it easier to override the permission test for a specific action, as well as providing a way for other apps to plug in their own actions and permission tests.

The registry will have a `register_tester_class(model, action, tester_class)` method to register the model and action mapping to the tester class, and a corresponding `get_tester_class(model, action)` method.

You can register a permission tester class like the following:

```python
from wagtail.permissions import permission_registry


class CustomDeletePermissionTester(BasePermissionTester):
    def test(self):
        return self.user.groups.filter(name='Cleaners').exists()


permission_registry.register_tester_class(model, 'delete', CustomDeletePermissionTester)
```

To ease the use of the registry, it will also have `get_tester(user, action, obj, **kwargs)` and `test(user, action, obj, **kwargs)` methods that will return an instance of the tester class and call its `test` method, respectively.

You can test the permission at request time like the following:

```python
from wagtail.permissions import permission_registry


class DeleteView(...):
    def dispatch(self, request, *args, **kwargs):
        self.object = self.get_object()
        if not permission_registry.test(request.user, 'delete', self.object):
            raise PermissionDenied
        return super().dispatch(request, *args, **kwargs)
```

### Template tag

The `{% permissions %}` template tag will be a shorthand for `permission_registry.test()` within templates.

```python
@register.simple_tag
def permissions(user, action, obj, **kwargs):
    return permission_registry.test(user, action, obj, **kwargs)
```

This acts as a generic replacement for the `page_permissions` template tag. Ideally, permission tests should be done in the Python code instead. However, there are cases where you may want to do it in the template for simplicity, e.g. when looping through a list of model instances.

## Migration plan

The migration plan will be split into different parts:

1. How permission policies are accessed
2. Usage and deprecation of the `PagePermissionTester` class.
3. Implementation of generic permision tester classes and their usage in views.

### Access to permission policies

The main target for refactoring is any code that uses permission policies defined at import time.

For views, permission policies tend to be used via the `PermissionCheckedMixin`. Most (if not all) of these views operate on a model, so we can refactor the mixin to have a default property that gets the policy from the registry based on the model.

Before:

```py
# wagtail/admin/views/pages/listing.py
from wagtail.permissions import page_permission_policy

class IndexView(generic.IndexView):  # the generic class uses the PermissionCheckedMixin
    permission_policy = page_permission_policy
    ...
```

After:

```python
# wagtail/admin/views/generic/permissions.py
class PermissionCheckedMixin:
    @cached_property
    def permission_policy(self):
        if model := getattr(self, "model", None):
            return permission_registry.get_policy(model)


# wagtail/admin/views/pages/listing.py
class IndexView(generic.IndexView):
    # No need to set permission_policy, as long as the model is set
    ...
```

Most other places can be refactored in a similar way.

For `ModelViewSet` and its subclasses, we can change the `get_{foo}_view_kwargs()` method to _not_ pass the `permission_policy` attribute. Instead, the viewset's `permission_policy` will be registered with the registry in the viewset's `on_register()` method. The corresponding views will then automatically get the policy from the registry, using the default logic in `PermissionCheckedMixin`.

This migration will be done as part of the first milestone.

### `PagePermissionTester` class deprecation

The first target for refactoring is the `PagePermissionTester` class. This will be done by creating a new tester class for each action that the `PagePermissionTester` class handles. This likely means a tester class for each `can_*` method. We currently have:

| Method                                      | Action                  | Notes                                                             |
| ------------------------------------------- | ----------------------- | ----------------------------------------------------------------- |
| `user_has_lock()`                           | -                       | not really a permission test, only used in `can_unlock()`         |
| `page_locked()`                             | -                       | short for `page.get_lock().for_user(user)`, only used in `can_unpublish()` and `can_submit_for_moderation()` |
| `can_add_subpage()`                         | `add` ❓                | "subpage" is implicitly always the case, given the tree structure |
| `can_edit()`                                | `edit` ❓               | `edit` vs. `change` (what the `Permission` object uses)?          |
| `can_delete(ignore_bulk=False)`             | `delete` ❓             | split into `delete` and `bulk_delete`?                            |
| `can_unpublish()`                           | `unpublish`             |                                                                   |
| `can_publish()`                             | `publish`               |                                                                   |
| `can_submit_for_moderation()`               | `submit_for_moderation` |                                                                   |
| `can_set_view_restrictions()`               | `set_view_restrictions` |                                                                   |
| `can_unschedule()`                          | `unschedule`            |                                                                   |
| `can_lock()`                                | `lock`                  |                                                                   |
| `can_unlock()`                              | `unlock`                |                                                                   |
| `can_publish_subpage()`                     | `publish_subpage`       |                                                                   |
| `can_reorder_children()`                    | `reorder_children`      |                                                                   |
| `can_move()`                                | `move`                  |                                                                   |
| `can_copy()`                                | `copy`                  |                                                                   |
| `can_move_to(destination)`                  | `move_to`               |                                                                   |
| `can_copy_to(destination, recursive=False)` | `copy_to` ❓            | split into two actions for the recursive and non-recursive cases? |
| `can_view_revisions()`                      | `view_revisions`        |                                                                   |

After creating a tester class for each action, we will register them with the base `Page` model. Then, we will refactor all usages of `PagePermissionTester`. A bulk of the work is in replacing `Page.permissions_for_user()` calls (and subsequently the methods of `PagePermissionTester`) with individual permission checks using `permission_registry.test()`. Then, the `{% page_permissions %}` template tag should be replaced with the generic `{% permissions %}` tag. After all internal usages have been replaced, the `Page.permissions_for_user()` method and the `{% page_permissions %}` template tag will be deprecated. This will be done as part of the second milestone.

### Generic permission tester classes

The generic `BasePermissionTester` is a new concept. Non-page models currently only have the permission policies – mostly used in views via the `PermissionCheckedMixin`, and sometimes with additional ad-hoc permission logic in the view code itself. The mixin only has support for the model-based methods of the permission policy, i.e. it does not check on the instance level. To fully get the benefits of adding the generic permission tester interface, we need to refactor the views to use permission testers instead of the policies.

Before we can use permission testers in generic views, we need to implement the generic permission tester classes for actions that are not page-specific. In simple cases, these testers will make use of the permission policy's `user_has_permission()` and `user_has_permission_for_instance()` methods. These testers will be registered with Django's base `Model` class, so it applies to all models by default (unless there's a more specific registration, e.g. with `Page`).

For actions that require specific model mixins, they will be registered with the mixin class (e.g. the tester class for the `publish` action will be registered with `DraftStateMixin`). In some cases, the basic actions may be tested differently if the model has a specific mixin. For example, the `delete` action requires the user to also have `publish` permission if the object is currently live. This example can be implemented by having a subclass of the delete tester class that also checks for the `publish` permission and registering the class with `DraftStateMixin`, or by adding the mixin check in the generic delete tester class (TBD).

After creating and registering the tester classes, we will refactor the views to use the generic permission testers. We may be able to do this by refactoring `PermissionCheckedMixin`, but we need to consider the potential breaking changes for developers who have been using custom permission policies.

This will be done as part of the third milestone. Ideally, this milestone will be done in the same release as the second milestone. However, it may need to be done in a separate release if the work is too large, considering the number of views that need to be refactored.

## Open Questions

1. Naming. Permission policy and permission tester both use the term "action" to refer to the action being tested. However, they are different concepts: "action" in the permission policy relates directly to the Django `Permission` object's `codename` (e.g. `"delete"` action for `"delete_advert"` codename), while "action" in the permission tester is a more abstract concept that can be anything (e.g. `"add_subpage"`, `"move_to"`). Should we use different terms to avoid confusion?
   - Also note that the `BasePermissionPolicy` interface's initial design doesn't strictly require the use of Django's `Permission` model. This means technically the "action" can be as abstract as it is in permission testers. However, in practice, concrete implementations of `BasePermissionPolicy` that are in use within Wagtail always use Django's `Permission` model.
2. Should we use the same registry for both permission policies and permission testers?
   - They do not need to be in the same registry, but having two separate registries may be confusing.
