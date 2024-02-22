# RFC 92: Permissions registry

* RFC: 92
* Author: Sage Abdullah
* Created: 2024-02-20
* Last Modified: 2024-02-22

## Abstract

This RFC proposes a new permissions registry that maps a model class to a permission configuration. Everywhere a permission check is performed in Wagtail, the registry will be consulted to determine the configuration to use for the model or model instance in question. The registry will make it easier for developers to access the permission configuration of a model from anywhere in the code. It will also allow developers to define custom permission logic for both custom models and Wagtail's built-in models, to be used in place of the default.

## Background

### Basics

Wagtail's permission system is primarily based on Django's `Permission` and `Group` models. Managing permissions in Wagtail is done by assigning permissions to groups, and then adding users to those groups.

To support page permissions based on the page tree structure, Wagtail has a `GroupPagePermission` model that acts as a many-to-many relationship between the `Group`, `Page`, and `Permission` models. Permissions assigned to a `Group` and a `Page` instance are inherited by the page's descendants. Historically, the `GroupPagePermission` model used a `permission_type` `CharField` instead of a `ForeignKey` to the `Permission` model, but this was changed to use the latter in Wagtail 5.1 for consistency ([#10569][pr-10569]).

For models that are managed through collections (e.g. `Document` and `Image`), Wagtail has a `GroupCollectionPermission` model that acts as a many-to-many relationship between the `Group`, `Collection`, and `Permission` models. Permissions assigned to a `Group` and a `Collection` instance are inherited by the collection's descendants.

For all other models, Wagtail uses Django's built-in permissions system. Permissions are managed by assigning `Permission` instances (for that model's `ContentType`) to a `Group` instance. Wagtail currently does not have instance-based permissions for these models. This means that for a given model, a user either has permissions for all of the model's instances or none at all.

Django also allows `Permission` instances to be assigned to a `User` instance directly. Wagtail tries to honour this when checking model permissions, but this is undocumented and there is no way to assign permissions to a `User` instance through the Wagtail admin interface. This is not supported for page and collection-based permissions.

### Historical background

Initially, Wagtail used Django's `permission_required` decorator to do basic permission checks in the admin. For page permissions, Wagtail needs a more fine-grained permission system that takes the page tree structure into account. To accommodate this, Wagtail had the following classes:

- `UserPagePermissionsProxy` to query the database for page permissions, and
- `PagePermissionTester` to perform permission checks against a specific page instance. This used `UserPagePermissionsProxy` under the hood.

When collections were implemented, Wagtail added the concept of "permission policies" in [#2102][pr-2102]. Quoting the PR,

> These are objects that provide a central point for making permission queries for a particular model - such as "can user X perform action Y on instance Z?" or "which instances can user X perform action Y on?". By collecting this logic into a single module, rather than having it distributed across templates, views and models, it's possible to redefine the permission rules for an app by swapping in a new policy object.

The permission policies are similar to the `UserPagePermissionsProxy`, and to some extent, the `PagePermissionTester`. These policies follow a standardised interface defined by the [`BasePermissionPolicy`][base-permission-policy] class. The class constructor takes a model class (or name) and it has methods to:

- check if a `user` has permission to perform `actions` on the model,
- check if a `user` has permission to perform `actions` on an `instance` of the model,
- query users that have permission to perform `actions` on the model,
- query users that have permission to perform `actions` on an `instance` of the  model, and
- query instances that a `user` has permission to perform `actions` on.

Permission policies do not necessarily use Django's `Permission` model. Provided they implement the same interface, they can use any method to determine if a user has permission to perform actions on a model or an instance of the model. For example, there are built-in `BlanketPermissionPolicy` that gives everyone (including anonymous users) full access to the model, and `AuthenticationOnlyPermissionPolicy` that gives authenticated users full access to the model. These are only examples in the Wagtail codebase and are not used by default. For permission policies that Wagtail use by default, they all work with Django's `Group` and/or `Permission` models under the hood.

Since its introduction, permission policies have become the primary way to perform permission checks for non-page models in Wagtail. This is typically done using a view decorator for function-based views, or more recently by setting a `permission_policy` attribute on a view class that extends `wagtail.admin.views.generic.PermissionCheckedMixin` and specifying the required permissions.

The implementation of permission policies live in the [`wagtail.permission_policies`][wagtail.permission_policies] module. For most built-in models, Wagtail has static instances of permission policies that live in the [`wagtail.permissions`][wagtail.permissions] module. For optional models such as images and documents, they live in a `permissions` module inside the app, e.g. `wagtail.images.permissions`. For custom models, developers can define their own permission policies and set them as the `permission_policy` attribute of the view or viewset classes.

For page models, it was not until Wagtail 5.1 did we implement the `PagePermissionPolicy` class. Prior to Wagtail 5.1, permission checks for pages were done in the following ways:

1. Instantiate `user_perms = UserPagePermissionsProxy(user)`.
   - The instance stores a queryset of the user's `GroupPagePermission` instances.
   - If you need to make permission-related queries that are not specific to a page instance, you would use this instance directly. For example, `user_perms.editable_pages()`, `user_perms.publishable_pages()`, etc.
2. Call `user_perms.for_page(page)` to get a `PagePermissionTester` instance.
   - This instantiates `PagePermissionTester(user_perms, page)` and the `user_perms` instance is stored as an attribute of the `PagePermissionTester` instance. This creates a tight coupling between the two classes (both classes are aware of each other).
   - The instance stores a set of the `permission_type` field of the `GroupPagePermission` instances that apply to the user and the page. For example, `{"add", "edit", "publish"}`.
   - You can use this instance to perform permission checks against the page instance. For example, `can_edit()`, `can_publish()`, etc.
   - There are also methods that take **additional arguments**, e.g. `can_delete(self, ignore_bulk=False)`, `can_move_to(self, destination)`, and `can_copy_to(self, destination, recursive=False)`.

Using an instance of `PagePermissionTester` is the primary way to perform permission checks for pages in Wagtail. This is usually done in templates using the `{% page_permissions %}` template tag. Each instance of `UserPagePermissionsProxy` would perform a database query to get the user's `GroupPagePermission` instances. To optimise this, the `UserPagePermissionsProxy` instance is cached in the template context and is reused by the `{% page_permissions %}` template tag.

However, there are cases where permission checks are done in the Python code without access to the template context. This is usually done either by creating a new instance of `UserPagePermissionsProxy` and calling `.for_page(page)`, or using the `page.permissions_for_user(user)` shorthand which did the same thing. Unfortunately, this meant that a new `UserPagePermissionsProxy` instance was created for each permission check outside the templates, which led to a large number of duplicated database queries.

Wagtail 5.1 added a caching mechanism in the permission policies to reduce the number of database queries. The newly-implemented `PagePermissionPolicy` class uses the caching mechanism to store the user's `GroupPagePermission` instances on the user object to be reused for the duration of the request. The new class replaced the `UserPagePermissionsProxy` class to perform the database queries for page permissions.

The `PagePermissionTester` class was modified to make use of the `PagePermissionPolicy` and its caching mechanism under the hood, while still providing the same interface for permission checks. The `page.permissions_for_user(user)` instantiates the `PagePermissionTester` directly and is now the recommended method to get a `PagePermissionTester` instance.

For more details on the changes, read the blog post about [Permissions and performance improvements in Wagtail 5.1][51-blog].

### Current state

Despite having the `PagePermissionPolicy` implemented, we still kept the `PagePermissionTester` class to perform permission checks against a specific page instance. The main reason is that the permission policies interface is not completely compatible with how `PagePermissionTester` works.

The `BasePermissionPolicy` has the following interface for instance-specific permission checks:
  - `user_has_permission_for_instance(user, action, instance)`
  - `user_has_any_permission_for_instance(user, actions, instance)`

For example, to check if a user has permission to edit an image, you would call `image_permission_policy.user_has_permission_for_instance(user, "change", image)`. In Wagtail, such checks are done in the Python code (e.g. in the view) and the appropriate behaviour is displayed to the user.

While not exactly enforced by the `BasePermissionPolicy`, the `action` parameter in all of Wagtail's default permission policies corresponds to the "action" part of Django's `Permission.codename`. For default permissions, the `codename` is in the format `"<action>_<model_name>"`. As an example, checking for the `"change"` action means checking for a `Permission` with the `codename` of `"change_image"` for the `wagtailimages.Image` model.

Meanwhile, the `PagePermissionTester` class constructor takes the user and the page instance. Rather than instantiating the `PagePermissionTester` class directly, permission checks are typically done either by:

- calling `page.permissions_for_user(user)` to get a `PagePermissionTester` instance, or
- using the `{% page_permissions %}` template tag to get a `PagePermissionTester` instance.

In any case, the `PagePermissionTester` uses dedicated methods to perform permission checks against the page instance. For example, `can_edit()`, `can_publish()`, etc. These method names do not always correspond to the "action" part of Django's `Permission.codename`. They may also perform additional checks that are not related to the `Permission` model, such as checking for `Page.subpage_types` in `can_add_subpage()`. Incorporating these checks into `PagePermissionPolicy` would make the implementation more complex and less consistent with the other permission policies.

There are also methods that take **additional arguments**, e.g. `can_delete(self, ignore_bulk=False)`, `can_move_to(self, destination)`, and `can_copy_to(self, destination, recursive=False)`.

For the above reasons, the `PagePermissionPolicy` class does not completely replace the `PagePermissionTester` class. It is unlikely that we will consolidate `PagePermissionTester` into `PagePermissionPolicy` in the future, at least not without modifications to the permission policies interface.

### Current pain points

There are a couple of major pain points with the current implementation of permissions in Wagtail, which this RFC aims to address.

#### Different permission checks for pages and snippets

Features from the `Page` model have been made available to other models via model mixins for snippets. When these features were implemented, we had to do some manoeuvres around the differences in the permission checks. In some cases, we share the code between pages and snippets, but we had to check whether the object is a page, and do the permission checks accordingly. In other cases, we duplicate the code with different permission checks that use the model's permission policy instead of the `PagePermissionTester`.

Doing permission checks for page models is quite straightforward and can be done anywhere provided you have access to the user object and the page instance. This is thanks to the `page.permissions_for_user` method and the `{% page_permissions %}` template tag (both of which use the `PagePermissionTester` class under the hood).

Doing permission checks for custom models is much more difficult in code that is not aware of the model's permission policy. For example, in our `Component` classes and templates, in hooks, and in the `wagtail.actions` module.

As mentioned before, permission policies for custom models are specified via the view or viewset classes' `permissions_policy` attribute. This means we have to "drill down" the model's permission policy (or the permission check results) from the view all the way down to the code that needs it, which is not always possible. It also means that there is no way for third-party code (or e.g. background jobs) to perform permission checks for custom models without knowing about the view or viewset classes. It introduces tight coupling between the view and the permission policy, when it should not be necessary.

To fix this problem, we need to have an easy way to do permission checks for any model, from anywhere in the code, provided that you have the user object and the model or model instance. This would be possible if we have a global registry that can be consulted to get the permission configuration for a model.

#### Hard to customise permissions for built-in models

Wagtail's built-in models have static instances of permission policies that live in the `wagtail.permissions` module (or a `permissions` module inside optional apps). These are defined at the module level (at import-time), which means there is no way to override them without monkey-patching. This is not a problem for most users, but it is a problem for developers who want to use custom permission policies for built-in models.

Custom permission behaviours have been a long-standing feature request ([#2907][issue-2907], [#4305][pr-4305], [#5600][pr-5600], [#5671][issue-5671], [#10702][pr-10702]). If we want to allow custom permission policies for built-in models, we need to have a way to override the default permission policy for a model. For this to be possible, the "canonical" permission policy instance for a model should not be set in stone statically at module level/import-time, but rather can be dynamically changed at run-time. Likewise, any code that uses a model's permission policy should retrieve it at run-time to ensure it is using the overridden policy.

Based on the above pain points, we need a global registry that maps a model class to its permission configuration. This registry will be consulted to determine the policy to use for the model and the permission tester for the model instance in question. The registry will allow developers to define custom permission policies and testers for both custom models and Wagtail's built-in models, to be used in place of the default ones.

### Permission policy vs. permission tester

Implementing a permissions registry still does not solve the discrepancy between the permission policies interface and the `PagePermissionTester` class. For this issue, we need to either:

1. incorporate `PagePermissionTester`'s logic into `PagePermissionPolicy`'s methods, or
2. accept that they are different things, and extend the "permission tester" concept to other models, e.g. creating a `ModelPermissionTester` for custom models.

The first option would make the `PagePermissionPolicy` implementation more complex and less consistent with the other permission policies. It would also muddy the waters between the two concepts, as the `PagePermissionPolicy` would be doing checks based on the `Permission` model and additional checks that are very specific to the `Page` model behaviour.

As mentioned before, there are also methods in `PagePermissionTester` that take additional arguments. While we can keep these methods as being specific to the `PagePermissionPolicy`, this will make it deviate further from the `BasePermissionPolicy` interface. Not to mention that the policy is not bound to the user and the page instance, so they will always need to be passed as parameters, e.g. `page_permission_policy.user_can_copy_page_to(user, page, destination, recursive=False)`.

With how the other default permission policies are implemented, we can establish (at least for now) that the permission policies are **not** meant to do the same things as the permission testers. The permission policies are meant to do basic permission checks according to the `Permission` model, while the permission testers are meant for more complex, higher-level permission checks that are specific to the model's behaviour.

The first option would also be a breaking change and would require major changes to how permission checks are implemented in Wagtail. As such, it is not a viable option at this time.

The second option, on the other hand, would be a new feature that would be a good complement to the permission policies. It would allow us to share the same permission check logic between pages and snippets, e.g. with a generic `{% permissions %}` template tag that works for any model. It would also avoid breaking changes and would not require major changes to how permission checks are implemented in Wagtail.

That said, most models in Wagtail do not really need a permission tester. The reason is that permission tester is useful for models that need custom logic for each instance. Built-in models in Wagtail do not need such custom logic, i.e. it is sufficient to check if the user has a certain permission for the model. A slightly more complex example is the collection-based models i.e. documents, images, and collections, which take into account the ownership of the model instance as well as the collection's tree structure. However, these can still be modelled at query-time using the permission policy (the current implementation is a proof of that). This does not mean adding permission testers for these models would be bad, it would just not be very useful in the default case, other than to provide a consistent interface for all models.

Permission tester would be most useful for snippets that have the optional features enabled, as they behave more like a page. For example, you would want to prevent a user that has the `delete` permission from deleting a live snippet if the user does not have the `publish` permission. With the generic nature of snippets (i.e. each feature may be enabled or disabled for each model), such permission checks are best done at instance-level. The built-in `ModelPermissionPolicy` for all snippet models would just query the `Permission` objects that the user has for the model, and the permission tester would do the rest.

The addition of a generic permission tester does not mean all instance-based permission checks have to be fully implemented within the tester. For example, if developers have their own model to implement instance-based permissions, they can still implement a custom permission policy to do the database queries and either do the checks directly in the policy or use the permission tester to implement the logic.

In conclusion, it does not look like we are getting rid of `PagePermissionTester` any time soon, and it would be useful to have a generic permission tester for snippets. To justify having two different things for permissions, we can use the following as a rule of thumb:

- Permission policies are where permission-related database queries are done, and where the basic permission checks (solely based on the query results) are done. They cache the query results on the user object to be reused for the duration of a request.
- Permission testers are where the higher-level, instance-specific permission checks are done, and where the additional checks that are not related to the `Permission` model are done. Ideally, they should not perform any database queries, but rather use the results from the permission policy.

## Proposal

### The registry

Wagtail already implements registries for various things, such as form field widget overrides and log entry models. They are implemented using the base [`wagtail.utils.registry.ObjectTypeRegistry`][ObjectTypeRegistry] class. This class considers the inheritance chain of the object type, so that if a subclass is not registered, the registry will look for a registration of the superclass. This means developers can also register a custom permission configuration for certain few specific subclasses of the `Page` model, or override the default configuration for all `Page` models. We will use this class to implement the permissions registry.

The registry will live in the `wagtail.permissions` module, where the built-in permission policies are currently defined. It will be a global instance of `ObjectTypeRegistry` that maps a model class to the permission configuration for the model. The `ObjectTypeRegistry` class has the following methods:

- `register(cls, value)` to register a value in the registry
- `get_by_type(cls)` to get the value for a given type
- `get_for_instance(instance)` to get the value for a given instance (using `get_by_type` under the hood)

To register a permission configuration, you would do the following:

```python
from wagtail.permissions import permission_registry

permission_registry.register(model, permission_configuration)
```

For custom apps, this would typically be done in the `AppConfig.ready` method.

When you need to use the permission configuration for a model, you would do the following:

```python
from wagtail.permissions import permission_registry

permission_configuration = permission_registry.get_by_type(model)
```

Retrieving a permission configuration should **never** be done at the module level, as it would be evaluated at import-time. Instead, it should be done inside a function or method that is called at run-time. This is to ensure any overrides are taken into account.

Notice that the above examples uses the term "permission configuration" instead of "permission policy" or "permission tester". This is because we have not decided what gets registered in the registry. **This will be the main question in this RFC.**

We have established that permission policies and permission testers are different things that we want to keep. As of now, they serve different purposes and we still need both to be easily accessible. Whatever gets registered in the registry, we need to have the following conditions met:

1. Given a model class, we need to be able to get the permission policy for the model.
2. Given a user instance and a model instance, we need to be able to get the permission tester for that user and model instance.

Before deciding what gets registered in the registry, let us look at how the `PagePermissionTester` is currently implemented.

For reference, the following is how the `UserPagePermissionsProxy` and `PagePermissionTester` were implemented before Wagtail 5.0:

<details>

  <summary>Source code</summary>

  ```python
  class UserPagePermissionsProxy:
      """Helper object that encapsulates all the page permission rules that this user has
      across the page hierarchy."""

      def __init__(self, user):
          self.user = user

          if user.is_active and not user.is_superuser:
              self.permissions = GroupPagePermission.objects.filter(
                  group__user=self.user
              ).select_related("page")

      def for_page(self, page):
          """Return a PagePermissionTester object that can be used to query whether this user has
          permission to perform specific tasks on the given page"""
          return PagePermissionTester(self, page)

      ...


  class PagePermissionTester:
      def __init__(self, user_perms, page):
          self.user = user_perms.user
          self.user_perms = user_perms
          self.page = page
          self.page_is_root = page.depth == 1  # Equivalent to page.is_root()

          if self.user.is_active and not self.user.is_superuser:
              self.permissions = {
                  perm.permission_type
                  for perm in user_perms.permissions
                  if self.page.path.startswith(perm.page.path)
              }

      ...
  ```

</details>

With `UserPagePermissionProxy` now replaced by `PagePermissionPolicy`, here is how `PagePermissionTester` is now implemented:

<details open>
  <summary>Source code</summary>

  ```python
  class PagePermissionTester:
      def __init__(self, user, page):
          from wagtail.permissions import page_permission_policy

          self.user = user
          self.permission_policy = page_permission_policy
          self.page = page
          self.page_is_root = page.depth == 1  # Equivalent to page.is_root()

          if self.user.is_active and not self.user.is_superuser:
              self.permissions = {
                  # Get the 'action' part of the permission codename, e.g.
                  # 'add' instead of 'add_page'
                  perm.permission.codename.rsplit("_", maxsplit=1)[0]
                  for perm in self.permission_policy.get_cached_permissions_for_user(user)
                  if self.page.path.startswith(perm.page.path)
              }
      ...
  ```

</details>

As a reminder:

- the `PagePermissionTester` lives in `wagtail/models/__init__.py`, along with models like `Page` and `GroupPagePermission`,
- the `PagePermissionPolicy` lives in `wagtail/permission_policies/pages.py`, and
- the `page_permission_policy` is a static instance of `PagePermissionPolicy(Page)` that lives in `wagtail/permissions.py`.

With the current situation in mind, we have a few options for what gets registered in the registry.

### Option A: The permission policy

For models other than pages, permission policies are currently the only thing that is used to perform permission checks. This means in most cases, what we will need to do is to register the default permission policies for the built-in models, and refactor existing code to get the policy from the registry at run-time instead of importing the default instances directly.

However, we still need to implement how to get a permission tester for a model instance. If we use the registry to store permission policies, then we will need to have a way to get the permission tester for a model instance from the policy. For example, we can add a `get_permission_tester` method to the `PagePermissionPolicy` class that looks like the following:

```python
class PagePermissionPolicy(OwnershipPermissionPolicy):
    tester_class = PagePermissionTester

    def get_permission_tester(self, user, instance):
        return self.tester_class(user, instance)
    ...
```

To customise the permission policy and the permission tester for a model, you would subclass the default permission policy and override the `tester_class` or `get_permission_tester` method. Then, register the custom permission policy in the registry. For example:

```python
# myapp/permissions.py

from wagtail.models import PagePermissionTester
from wagtail.permission_policies.pages import PagePermissionPolicy
from .models import CustomPage


class CustomPagePermissionTester(PagePermissionTester):
    # Customise here
    ...


class CustomPagePermissionPolicy(PagePermissionPolicy):
    tester_class = CustomPagePermissionTester

    # More customisation here

custom_page_permission_policy = CustomPagePermissionPolicy(CustomPage)
# or if you want to override the default for all Page models, do:
# custom_page_permission_policy = CustomPagePermissionPolicy(Page)
```

```python
# myapp/apps.py

class MyAppConfig(AppConfig):

    def ready(self):
        from wagtail.permissions import permission_registry
        from .models import CustomPage
        from .permissions import custom_page_permission_policy

        permission_registry.register(CustomPage, custom_page_permission_policy)

        # or if you want to override the default for all Page models, do:
        # permission_registry.register(Page, custom_page_permission_policy)
```

To use it, you would do:

```python
from wagtail.permissions import permission_registry

permission_policy = permission_registry.get_by_type(model)
permission_tester = permission_policy.get_permission_tester(user, instance)
```

Since the model can be derived from the instance, we can also add a `get_tester` convenience method that does the above under the hood, and perhaps rename the default `get_by_type` method to `get_policy`:

```python
from wagtail.permissions import permission_registry

permission_policy = permission_registry.get_policy(model)
permission_tester = permission_registry.get_tester(user, instance)
```

For non-page permission policies, the method would return an instance of a generic `ModelPermissionTester` or a more suitable tester instance (or perhaps left unimplemented).

However, this means that the policy is now aware about the tester. The `PagePermissionTester` is currently already aware about the policy to get the cached permissions. It is not ideal to have both classes be aware about each other.

Note that while the rule of thumb says that permission tester needs to use the query results from the permission policy, it does not necessarily mean the permission tester needs to know about the permission policy as it currently does. For example, we can make it so that the permission tester accepts the permission objects in its constructor, and use them to perform the checks:

```python
# wagtail/models/__init__.py
class PagePermissionTester:
    def __init__(self, user, page, permissions):
        self.user = user
        self.page = page
        self.page_is_root = page.depth == 1  # Equivalent to page.is_root()
        self.permissions = {
            perm.permission.codename.rsplit("_", maxsplit=1)[0]
            for perm in permissions
            if self.page.path.startswith(perm.page.path)
        }

    ...
```

```python
# wagtail/permission_policies/pages.py
class PagePermissionPolicy(OwnershipPermissionPolicy):
    ...

    def get_permission_tester(self, user, page):
        # Get cached GroupPagePermission instances for the user,
        # and pass them to the permission tester
        permissions = self.get_cached_permissions_for_user(user)
        return PagePermissionTester(user, page, permissions)
```

This effectively reverses the relationship between the two. The permission policy is now aware about the permission tester, but the permission tester is not aware about the policy.

One potential downside of this approach is that the permission tester is now stuck with whatever permissions were given at the time of instantiation. It cannot make use of the permission policy to make additional permission-related queries.

This is in line with the rule of thumb that the permission tester should not perform any database queries, but completely removing the ability to do so may be too restrictive. For example, you might want to use the permission policy to get the permissions of a different user, or to get the cached result of a different query (e.g. the "explorable root instance").

Technically we can still access the permission policy from inside the tester using the registry, but that reintroduces the tight coupling between the two classes that we are trying to avoid.

### Option B: The permission tester

Registering the permission tester instead of the policy would allow us to keep the concept that the permission tester is a higher-level abstraction that uses the permission policy under the hood. However, we need to be mindful that permission tester instances are bound to the user and the model instance. Since we also need to be able to get the permission policy for a model, we need to have a way to get the policy from the tester without instantiating the tester. This means that the registry needs to store the permission tester class, not the instance.

To access the permission policy from a permission tester class, we can add a `get_permission_policy` class method to the `PagePermissionTester` class that looks like the following:

```python
class PagePermissionTester:

    @classmethod
    def get_permission_policy(cls):
        # This import was originally in the constructor
        from wagtail.permissions import page_permission_policy
        return page_permission_policy

    def __init__(self, user, page):
        self.permission_policy = self.get_permission_policy()
        # The rest of the implementation stays the same
```

To customise the permission policy and the permission tester for a model, you would subclass the default permission tester and override the `get_permission_policy` method. Then, register the custom permission tester in the registry. For example:

```python
# myapp/permissions.py

from wagtail.models import PagePermissionTester
from .models import CustomPage

class CustomPagePermissionPolicy(PagePermissionPolicy):
    # Customise here
    ...


custom_page_permission_policy = CustomPagePermissionPolicy(CustomPage)


class CustomPagePermissionTester(PagePermissionTester):

    @classmethod
    def get_permission_policy(self):
        return custom_page_permission_policy

    # More customisation here
```

```python
# myapp/apps.py

class MyAppConfig(AppConfig):

    def ready(self):
        from wagtail.permissions import permission_registry
        from .models import CustomPage
        from .permissions import custom_page_permission_tester

        permission_registry.register(CustomPage, custom_page_permission_tester)
```

To use it, you would do:

```python
from wagtail.permissions import permission_registry

permission_tester_class = permission_registry.get_by_type(model)
permission_tester = permission_tester_class(user, instance)
permission_policy = permission_tester_class.get_permission_policy()
```

As with option A, we can add a `get_policy(model)` and `get_tester(user, instance)` convenience methods to the registry that do the above under the hood.

The main consideration for this option is that all models now need to have a permission tester. It may not be a big deal as we will likely add a generic `ModelPermissionTester` in any case. However, as mentioned before, it would not be useful for most built-in models, as the basic checks in the permission policy already suffice. To make the tester useful, we would need to refactor existing code to use it in place of the policy wherever we check for permissions against a specific instance. This is in addition to refactoring the existing code to get the policies from the registry at run-time instead of importing the default instances directly.

### Option C: Both

We can register both the permission policy and the permission tester, but we will need to either:

- create separate registries for the policy and the tester, or
- modify the registry to add methods to set and get the policy and the tester class, or
- create a wrapper that stores both the policy and the tester.

The wrapper approach would be something like the following:

```python
class PermissionConfiguration:
    permission_policy = CustomPagePermissionPolicy(CustomPage)
    permission_tester_class = CustomPagePermissionTester


permission_registry.register(CustomPage, PermissionConfiguration)
```

```python
permission_configuration = permission_registry.get_by_type(model)

permission_policy = permission_configuration.permission_policy
permission_tester_class = permission_configuration.permission_tester_class
permission_tester = permission_tester_class(user, instance)
```

As with the other options, we can add convenience methods to the registry to get the policy and the tester directly.

Keep in mind that the tester would still need to use the policy to get the cached permissions, so it would need to consult the registry, or the registry has to pass the policy to the tester using the convenience method. This does not sound like a good idea, as it means the tester would need to be aware of the registry in addition to the policy. Not to mention that without the convenience methods, the tester would also need to be aware of the wrapper.

## Open Questions

- Which option should we take?
- What is option D?

[51-blog]: https://wagtail.org/blog/benchmarking-wagtail-51/
[base-permission-policy]: https://github.com/wagtail/wagtail/blob/c54d9aa64fd7a5030de878f764cd933181e7d938/wagtail/permission_policies/base.py#L11
[ObjectTypeRegistry]: https://github.com/wagtail/wagtail/blob/cadc40e6a2f81578cb03da3ed085fef475b94035/wagtail/utils/registry.py#L7
[wagtail.permission_policies]: https://github.com/wagtail/wagtail/tree/stable/6.0.x/wagtail/permission_policies
[wagtail.permissions]: https://github.com/wagtail/wagtail/tree/stable/6.0.x/wagtail/permissions.py
[issue-2907]: https://github.com/wagtail/wagtail/issues/2907
[issue-5671]: https://github.com/wagtail/wagtail/issues/5671
[pr-2102]: https://github.com/wagtail/wagtail/pull/2102
[pr-4305]: https://github.com/wagtail/wagtail/pull/4326
[pr-5600]: https://github.com/wagtail/wagtail/pull/5600
[pr-10569]: https://github.com/wagtail/wagtail/pull/10569
[pr-10702]: https://github.com/wagtail/wagtail/pull/10702
