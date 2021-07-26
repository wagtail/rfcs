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

This feature is not intended to provide an absolute guarantee that any database state change will be traceable to a logged action - for example, if a change is made through custom code or a third-party library, it will not be recorded unless the code in question has incorporated support for logging.

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


## Resolved questions

### Model-level logging

> In the design outlined above, view-level code must explicitly call `log` in order for actions to be logged, and it is still possible for un-logged changes to occur (e.g. through a third-party app that has not yet been updated to include `log` calls). However, with the introduction of logging contexts, it would be possible to perform meaningful logging (including capturing the current user) at the model level, e.g. through model `save` methods or signals. Is this desirable, and if so:
>
> * Which models do we enable this on? "All models" seems a bad idea, as this would include e.g. inline child models. Maybe this is functionality that models "opt into", and is enabled by default for models being registered through snippets or ModelAdmin?
> * If logging exists at both view and model level, how do we avoid duplicate log entries referring to the same thing? For example, if the Wagtail admin provides a "publish newsletter" interface, and the publication process involves a call to `newsletter.save()`, then we're likely to end up logging a 'newsletter.publish' action from the view code, and a 'newsletter.save' action from the model level. Do we have some kind of hierarchy where the more specific action type (newsletter.publish) supersedes the less specific one?

No, we should not globally capture log actions at the model level.

In MVC architecture terms, there is no strict rule about whether the methods exposed by the model level correspond to meaningful loggable actions. Typically that's something that evolves as a model gets more complex: simple models (of the sort that would typically be managed through snippets or ModelAdmin) will often just rely on Django's generic `save()` method and leave it up to the view code to implement any cleverer logic, but as that view code gets more complex it will be refactored into model methods such as the page model's 'publish' and 'copy' methods. Meanwhile, Wagtail's workflow model is inherently based around user actions.

Given this diversity around how model code is approached in the real world, it doesn't make sense for Wagtail's logging framework to make blanket assumptions about whether a `save` signal being fired indicates a meaningful loggable action. Instead, that should be a case-by-case decision for each model.

This does mean that the current development falls short of being an 'audit log' in the strictest sense - it's possible for custom / third-party code to make database state changes that 'fall through the net' and aren't recorded at either the view or model level. We'll consider this out of scope for the current development - there's no proven need for this, and the ease of adding logging support to custom code (essentially just a single line of code per action) should mitigate this. If true low-level audit logging is really needed, this should probably be set up at the database level instead.


### Discovery of models / actions for report filters

> In the "site history" report, it would be useful to provide a dropdown to filter the report by object type. However, since actions can be potentially logged for _any_ model (not just ones explicitly registered with the log action registry, given that we're registering `actions.register_model(models.Model, ModelLogEntry)` as a catch-all), it's not clear what this dropdown should contain - listing all Django models would contain a lot of unwanted junk.

> * Do we require models to be explicitly registered as 'loggable' to be shown in that list? Or do we do a "preflight" query against the log tables to find out which object types have actions logged against them, and just show those?

The 'users' filter on that report already takes the latter approach, only showing users in the dropdown when they have at least one logged action. We can do the same for filtering by model - testing against an instance with ~500K log entries shows that this 'select distinct' query doesn't add unreasonable overhead to the report.

Similarly, any reports that only report on (for example) snippet models and wish to provide an 'action type' filter can build the list by running a 'select distinct' query over the dataset, so that only action types that have been logged against snippets are included in the list.


## Open Questions

There are still some details in reports where we would benefit from contextual information about the models we're reporting on, which goes beyond what the log action registry currently collects:

* Action labels - currently `register_action` associates a label with the action, to be used across all models. Since these labels were chosen with only pages in mind (with the intention that other models would have their own registries), there are quirks like [`wagtail.edit` being labelled "Save draft"](https://github.com/gasman/wagtail/blob/45379032db204bc8af11ce583e6912117fa6db3c/wagtail/core/wagtail_hooks.py#L106) even for models that have no concept of drafts. Ideally, we'd have a generic label that could be overridden for more specific models.
* Permissions - especially in multi-tenant setups, we may want to limit the visibility of models to just the ones a user can 'see', as defined by model-specific logic elsewhere in Wagtail (e.g. 'choose' permission for images and documents). The `BaseLogEntryManager.viewable_by_user` method is a first pass at this, but that would imply every model with non-trivial permission requirements needing its own log model. Can we do better than that?
* Linking to objects from reports - for page objects, the Site History report links to the 'edit' view of the page, but to do that for models in general we'd have to know whether the relevant view is in the snippets app, modeladmin, or something else (images, documents, wagtailmedia...). This problem has come up before - for example, [generating a friendly error page for on_delete=PROTECT that provides links to the objects that are blocking deletion](https://github.com/wagtail/wagtail/pull/3525#issuecomment-308510208) and [`describe_collection_contents`](https://docs.wagtail.io/en/stable/reference/hooks.html#describe-collection-contents) (where a collection management view within wagtailadmin ends up linking out to wagtailimages / wagtaildocs). Does this call for another kind of registry, where apps can declare "I provide an edit view for model X"? If so, what methods does that registry need to provide?
