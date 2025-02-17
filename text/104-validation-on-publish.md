# RFC 104: Validation on publish

* RFC: 104
* Author: Matthew Westcott
* Created: 2024-11-07
* Last Modified: 2024-11-07

## Abstract

A frequently-requested feature (see [Wagtail issue #12505](https://github.com/wagtail/wagtail/issues/12505)) is the ability to save draft versions of pages in an incomplete state that would not pass validation, but still enforce validation at the point that the page is published or submitted for approval. This is also a prerequisite for an effective auto-save implementation, as noted in [RFC 99](https://github.com/wagtail/rfcs/pull/99). This RFC sets out the requirements for such a feature, and an approach to implementing it.

## Overview of existing validation behaviour

Validation is primarily handled through the edit view's ModelForm instance. This picks up any validation rules defined at the model level, such as `blank=False` or `validators` arguments on fields. Additional validation logic can be added by overriding the `clean` method on the form (as the documentation on [customizing generated forms](https://docs.wagtail.org/en/stable/advanced_topics/customisation/page_editing_interface.html#customizing-generated-forms) recommends) or on the model. In all cases, validation runs when the form is submitted, regardless of whether the page is being saved as a draft, published, or submitted for moderation, and validation errors are displayed to the user in the form.

Separately, the Page model's `full_clean` method is called from `Page.save` and `Page.save_revision`. This is a Wagtail design decision, not one enforced by Django, and is mainly for the benefit of operations on pages that do not use forms, such as batch import tasks, as `full_clean` performs housekeeping such as setting a unique slug if one is not provided. Calling code can skip this validation by passing `clean=False` to `save` or `save_revision`, but this is generally only done when making narrow non-content-related changes to the page record, such as setting `live` to False to unpublish it, or updating `latest_revision` after a revision has been saved. Calls to `Page.save` that exist within Django's code, such as when calling `ModelForm.save`, do not pass `clean=False`.

In the context of the create / edit form, this validation step on save just duplicates the validation that has already been done by the form, since ModelForm's validation calls `full_clean`. However, this presents an extra 'gotcha' for us if we intend to bypass the form's validation step, as this second round of validation will still be performed - and this time any `ValidationError` raised will be left uncaught.

## Applicability to snippets

The above discussion has focused on pages, but the changes of behaviour proposed here would equally apply to snippets inheriting from `DraftStateMixin`.

## Autosave user and developer experience

In a non-autosaving environment, the inability to save an incomplete draft may be inconvenient, but will generally not be seen as "broken", in the sense that the editor can form a mental model that "I need to fill in the whole form before it will let me save". However, once the editor is introduced to a new mental model where auto-saving is the norm, their user experience will likely become jarring and contradictory - "it's supposed to be saving as I go, but it isn't". This presents additional challenges for us, as it means that validate-on-publish has to be not just a developer-facing feature, but an invisible part of Wagtail's developer and user experience. This has a major bearing on how the feature is designed - for example, if it only came into play on fields where the developer has made an express decision to configure it, then it would be acceptable for this configuration step to require non-standard Wagtail-specific attributes. However, if it's expected to "just work" without the developer having to think about it, then it needs to work with conventional Django validation mechanisms.

## Handling validation rules other than requiredness

A key question when scoping this feature is whether `blank=True` is the only validation rule that needs to be deferred until publish time, or whether there is a use-case for allowing drafts to be saved while other kinds of validation error are present. In this scenario, validation errors would behave more like warnings, being displayed immediately when validation fails but not preventing the page from being saved.

The main driver for the validate-on-publish feature is the ability for editors to save work-in-progress drafts where not all fields are filled in. Other kinds of validation error, such as malformed email addresses or inconsistent start / end dates, are usually a lesser concern as an editor can easily fix these as soon as they are flagged up. `blank=True` may not be the only rule that is applicable specifically to _incomplete_ pages - for example, given an intro paragraph field with a min_length of 500 characters, an editor may wish to enter something in the field and expand it later - but these are likely to be niche cases.

Supporting arbitrary validation rules may be difficult due to the way Django treats "cleaning" as an integral step in obtaining data from a form. For example, consider an event page type where the `clean` method validates that the start date is before the end date. If this condition is not satisfied, `clean` will raise a `ValidationError` and `form.is_valid()` will return False - but crucially, `form.cleaned_data` will not be populated, so there is no way to ignore the validation error and continue saving regardless. The raw POST data is not sufficient, as we rely on the cleaning step to convert the string value to a date object. (Indeed, with some kinds of validation errors, such as a date of 31st February, there is no well-formed value that it _could_ return.)

Given this difficulty, we propose to only support `blank=True` at this stage, and consider other validation rules as a future enhancement when a clear use-case arises.

## Validate-on-publish as an external step

A "light-touch" approach to this problem, that can be implemented with minimal impact to existing validation logic, is to implement publish-time validation as a separate piece of configuration that exists outside of the standard Django validation mechanism. In this approach, fields to be made required on publish would be defined on the model as `blank=True` to allow the existing validation step to complete successfully on save, and any additional rules to be applied on publish would be defined elsewhere in the model, such as the corresponding `FieldPanel` definitions. One existing implementation of this pattern [introduces a `publish_requires` decorator on the class](https://gist.github.com/thibaudcolas/e1cd014ed0c02b46993b1c4a4e134d9b).

As per "Autosave user and developer experience" above, this would be acceptable for a developer-facing opt-in mechanism, but the need for custom Wagtail-specific configuration would make this unsuitable for the kind of seamless developer experience needed for autosave. On a more philosophical level, it's undesirable to have model fields configured as `blank=True` to serve the needs of the edit view, when the field is not actually allowed to be blank within the site's own business logic.

## Proposed solution

A proof-of-concept of an autosave-compatible implementation has been created [on the `gasman/feature/validate-on-publish branch`](https://github.com/gasman/wagtail/commit/f1ab76d87788a8c52d36b644abe9461b7e3db472).

This extends the capabilities of `WagtailAdminModelForm` (Wagtail's `ModelForm` subclass used for forms within the admin) to accept a new `Meta` option `defer_required_on_fields`, which lists fields that are normally required but can be made non-required by calling the form's `defer_required_fields()` method. This functionality could conceivably be offloaded to an external package similar to `django-permissionedforms`, but given its simplicity (and unclear usefulness outside of Wagtail) it seems reasonable to include it in Wagtail core.

`FieldPanel.get_form_options` has been modified to include this list in the set of `Meta` options passed to the form, alongside the existing ones such as `fields` and `widgets`. Individual fields can be excluded from this list by passing `required_on_save=True` to the `FieldPanel` constructor, meaning that they will continue the current behaviour of being required on draft saves. This is the case for the `title` field (since even draft pages must have a title to show in page listings), and so `TitleFieldPanel` enables this option by default.

`EditView` and `CreateView` for pages have been refactored to check the requested action before validating the form, and if the action is `save` (indicating a save as draft) then `form.defer_required_fields()` is called ahead of validation to bypass the `blank=False` checks. All other validation rules run as normal, and validation errors are displayed in the form as usual. (As a minor point, the `defer_required_fields` operation should be reversed before the form is redisplayed, to ensure that those required fields are displayed with their asterisk as normal.)

In the final implementation, this change to the edit / create views will also be applied on the snippet views, for snippet models that inherit from `DraftStateMixin`.

In the proof of concept, the model-level validation on `save_revision` and `save` has been bypassed by calling `page.save_revision(clean=False)` on the views in question, and flipping the behaviour of `save` to default to `clean=False` so that the call to `form.save` does not trigger `full_clean`. This change will need to be considered carefully to ensure that it does not bypass necessary housekeeping such as setting the slug, including in external code that creates pages programmatically. One possible refinement is to continue defaulting to `clean=True` on `save` when the page's `live` flag is True.

## Handling of non-text fields

This process relies on the assumption that the model instance resulting from the form submission is in a state that can be legally saved to the database, even if some application-level validation rules have been skipped. This assumption holds for text-based fields (`CharField` and `TextField`) that have skipped a `blank=False` check, since databases will still allow empty strings to be saved (unless there is an explicit check constraint preventing this - in which case it is the developer's responsibility to enforce that on draft saves using `required_on_save=True`). However, this is not true for non-text fields such as `IntegerField`, as Django will attempt to save blank values as `NULL`, causing a database-level validation error if the field has not been defined as `null=True`.

For this reason, a developer must define such a field as `IntegerField(null=True)` to allow it to be left blank during drafts but enforce it as required on publish. This is in contrast to `IntegerField(null=True, blank=True)` which will also permit the field to be blank on publish.

To minimise the possibility of uncaught validation errors, `FieldPanel` will incorporate the following check:

* If the field name corresponds to a field on the model, and
* that field is a non-text type, and
* that field does not permit nulls, and
* the FieldPanel has not been passed an explicit `required_on_save` argument, then
* the FieldPanel will behave as if `required_on_save=True` has been passed, i.e. it will not add the field to the model form's `defer_required_on_fields` list.

## Open Questions

// Include any questions until Status is ‘Accepted’
