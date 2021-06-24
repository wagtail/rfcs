# RFC 69: Model audit logging

* RFC: 69
* Author: Matthew Westcott
* Created: 2021-06-23
* Last Modified: 2021-06-23

## Abstract

Wagtail 2.10 introduced logging and reporting for changes made to page models. We now wish to extend this logging mechanism to capture all actions performed within the admin (and related processes such as scheduled tasks). Our design goals are as follows:

* Logged data must include the user performing the action (which would not be possible with a solution just using Django model signals, for example)
* It must be possible to generate reports that span multiple model types, e.g. "which actions did user X perform today"
* The amount of 'boilerplate' code needed to add logging to Wagtail admin views should be minimised (since there are many such views: snippets, ModelAdmin, images, documents and so on).

A proof-of-concept implementation of the feature described here can be found at: https://github.com/gasman/wagtail/tree/feature/model-audit-log

## Specification

### Data model

[A `ModelLogEntry` model](https://github.com/gasman/wagtail/blob/3f27f38be51eab5c421c05bdfb5ede77bb9e2ecc/wagtail/core/models/audit_log.py#L194-L210), extending the existing abstract `BaseLogEntry` model, will be added to `wagtailcore` alongside `PageLogEntry`. This differs from `PageLogEntry` in that the object ID is stored as a string, allowing it to refer generically to any object, rather than a foreign key.

During the development of page logging, we found that a general-purpose log entry model, with a generic foreign key, made it difficult to write efficient report queries that correctly applied the page tree's permission logic, and so the current PageLogEntry model, with a foreign key to Page, was chosen. While it may be possible to make Page work well with a generic logging model, there's no guarantee that this will be the case for all models that we might wish to log against, so continuing to support model-specific log models is the pragmatic option.

At the other extreme, we could consider a setup where each model has its own distinct log model. However, this would create an unacceptably large maintenance burden (for example, needing a migration to create the log model for each new snippet model introduced) and would make it inefficient to construct report queries that span many model types (since they would potentially need to combine results from dozens of log tables).

We have therefore chosen a middle ground, where an object type can be configured to log to either the generic ModelLogEntry model or its own purpose-built model such as PageLogEntry. It's expected that most models will use ModelLogEntry, and at least initially, PageLogEntry will be the only special case - but as and when other models grow in complexity to the point where ModelLogEntry's generic foreign key is unsuitable, they can be migrated to a purpose-built log model.

In this approach, the logging framework will need to support multiple LogEntry models, but in practice only a few (initially, two) will be in use, so reports that span multiple models should remain relatively efficient.


### Registry

In current Wagtail versions, code that intends to log page-related actions will call `PageLogEntry.objects.log_action`. If we were to extend this pattern to other object types, we would need to call `ModelLogEntry.objects.log_action` or potentially some other handler method specific to that object type. This is undesirable - the choice of log model should be considered an internal implementation detail, and code that intends to log an action should not have to care which particular log model an object is using.

Instead, we will provide [an all-purpose 'log' method](https://github.com/gasman/wagtail/blob/3f27f38be51eab5c421c05bdfb5ede77bb9e2ecc/wagtail/core/log_actions.py#L158-L159) which directs the log entry to the appropriate log model. This can be invoked as follows:

    from wagtail.core.log_actions import log

    def create(request):
        # ...
        my_object = form.save()
        log(instance=my_object, action='wagtail.create', user=request.user)

To achieve this, we extend the existing concept of the "log action registry"; in addition to registering new action types through `register_action`, new object types and their associated log models can be registered through `register_model`:

    @hooks.register('register_log_actions')
    def register_core_log_actions(actions):
        actions.register_model(models.Model, ModelLogEntry)
        actions.register_model(Page, PageLogEntry)

When `log` is called, the registry will look for the registered model that most closely matches the type of the passed instance (according to its Method Resolution Order), and will then call `<log_model>.objects.log_action` on the resulting log model.

Note: Some refactoring was done in the 2.13 release cycle, where it was anticipated that [each object type would have its own registry of log actions](https://github.com/wagtail/wagtail/commit/f2f6167f5d0516a41d33ad73433f469b93dc77b5). This will be rolled back as it's inconsistent with this new architecture; it would entail having to consult a "registry of registries" to find out which registry is relevant to the object being logged. Instead, there will be a single registry handling all object types.


### Logging context

In the above example, we pass `user` to the `log` method. This can be simplified further by introducing a 'logging context', which [stores a set of common properties in a thread-local variable](https://github.com/gasman/wagtail/blob/3f27f38be51eab5c421c05bdfb5ede77bb9e2ecc/wagtail/core/log_actions.py#L8-L43) that will be attached to all logged actions within that context:

    from wagtail.core.log_actions import LogContext, log

    with LogContext(user=request.user):
        log(instance=instance, action='wagtail.create')

By [activating a log context within the `require_admin_access` decorator](https://github.com/gasman/wagtail/blob/3f27f38be51eab5c421c05bdfb5ede77bb9e2ecc/wagtail/admin/auth.py#L171) that is applied to all admin views, we can eliminate the need for view code to explicitly pass the user.

Initially the logging context will just store the user, but this could be extended to include the request URL or view name, and generate a UUID that can be used to group together actions that happen in the same request (for example, bulk-deleting snippets is notionally a single action and should be shown as such in reports, but needs one log item per deleted object).


### Reporting

Reports such as the 'site history' report will need to take data from multiple log models. These reports need to be agnostic of the log models and permission logic in use, since specific object types should be able to customise these (and we don't want to have to rewrite our report views in that situation). To make this possible:

* The log registry provides a `get_log_entry_models()` method, which returns the set of all distinct log models that have been registered with `register_model`;
* The manager for a log model is expected to implement a `viewable_by_user` method, returning a queryset of log entries that are viewable to a given user;
* `ReportView` is refactored to provide an overrideable `get_filtered_queryset` method. Currently report views are expected to define a `get_queryset` method returning a query that can be filtered according to the filter controls on the report, but this doesn't work on `union` querysets (where each subquery needs to be filtered individually).

This can be seen in action on [`LogEntriesView`](https://github.com/gasman/wagtail/blob/3f27f38be51eab5c421c05bdfb5ede77bb9e2ecc/wagtail/admin/views/reports/audit_logging.py#L75-L91).


## Open Questions

### Model-level logging

In the design outlined above, view-level code must explicitly call `log` in order for actions to be logged, and it is still possible for un-logged changes to occur (e.g. through a third-party app that has not yet been updated to include `log` calls). However, with the introduction of logging contexts, it would be possible to perform meaningful logging (including capturing the current user) at the model level, e.g. through model `save` methods or signals. Is this desirable, and if so:

* Which models do we enable this on? "All models" seems a bad idea, as this would include e.g. inline child models. Maybe this is functionality that models "opt into", and is enabled by default for models being registered through snippets or ModelAdmin?
* If logging exists at both view and model level, how do we avoid duplicate log entries referring to the same thing? For example, if the Wagtail admin provides a "publish newsletter" interface, and the publication process involves a call to `newsletter.save()`, then we're likely to end up logging a 'newsletter.publish' action from the view code, and a 'newsletter.save' action from the model level. Do we have some kind of hierarchy where the more specific action type (newsletter.publish) supersedes the less specific one?

### Discovery of models / actions for report filters

In the "site history" report, it would be useful to provide a dropdown to filter the report by object type. However, since actions can be potentially logged for _any_ model (not just ones explicitly registered with the log action registry, given that we're registering `actions.register_model(models.Model, ModelLogEntry)` as a catch-all), it's not clear what this dropdown should contain - listing all Django models would contain a lot of unwanted junk.

* Do we require models to be explicitly registered as 'loggable' to be shown in that list? Or do we do a "preflight" query against the log tables to find out which object types have actions logged against them, and just show those?
* Do we need to take permissions into account - e.g. only allowing users to view logs for a model if they have an explicit add/edit permission for it? Does this justify a distinct 'can view logs' permission level?
* Currently, when actions are registered with `register_action`, they are not associated with a specific object type - for example, there's nothing to specify that the 'wagtail.publish' action type only applies to pages and not snippets. This means that the 'filter by action' dropdown for a report that's only reporting on snippets is likely to include a bunch of useless actions. Can / should we prevent that, by linking actions to specific object types?
