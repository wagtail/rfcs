# RFC 105: Site setting permissions

* RFC: 105
* Author: Matt Westcott
* Created: 2025-01-24
* Last Modified: 2025-01-24

## Abstract

Currently, permissions on [site settings](https://docs.wagtail.org/en/stable/reference/contrib/settings.html) are governed by a standard global Django permission record on the model, meaning that users have edit permission for the settings of all sites, or none. This does not fit well with organisational structures where a member of staff has overall responsibility for one particular site. This RFC proposes a new permission model where groups can be assigned permissions at the level of individual sites.

## Specification

### Data model

We will introduce a new core model `GroupSitePermission`, with the fields:

* `group` - foreign key to `django.contrib.auth.models.Group`
* `site` - foreign key to `wagtail.models.Site`
* `permission` - foreign key to `django.contrib.auth.models.Permission`

For site settings models, the only meaningful permission type is "change", since there is no user-facing notion of adding or deleting settings. (In reality, a newly-created site will not have a settings record associated with it, and this will be created when the settings are first saved - but this detail is invisible to the user, and thus does not warrant a separate permission type.) It is possible for multiple settings models to exist - for example, a site might have `SocialMediaSettings` and `ThemeSettings`, with distinct permissions for each - and so the `permission` field will identify the model that the permission record applies to.

For example, a permission rule giving members of the "Blog owners" group the ability to edit social media settings for the site `blog.example.com` would look like:

* `group = <Group: Blog owners>`
* `site = <Site: blog.example.com>`
* `permission = <Permission: change_socialmediasettings>`

This follows the pattern of the existing `GroupCollectionPermission` model used for images and documents. Just as the `GroupCollectionPermission` model is not inherently tied to the Image or Document model, `GroupSitePermission` is not specific to settings models, and could conceivably be used for any other kind of model where permissions are partitioned by site. There are no immediate plans to make use of this beyond settings models, though.

We will also introduce a `SitePermissionPolicy` class, similar to `CollectionPermissionPolicy`, which can be used on admin views to perform permission-related queries such as "Does user X have change permission over SocialMediaSettings instance Y?" and "Which sites does user X have change_socialmediasettings permission on?". This will implement the methods common to all permission policy classes, with the following special behaviours:

* The standard "find all instances the user has change permission for" operation won't account for sites that do not yet have a relevant settings record, but should behave as if they do (as noted above). For this, we need a method that returns all applicable Site instances, rather than settings instances.
* The permission policy should check for standard Django permission records (the `User.user_permissions` and `Group.permissions` relations) in addition to `GroupSitePermission`, and treat them as granting permission across all sites. The rationale for this is discussed in "Backwards compatibility" below.

### Backwards compatibility

In some cases it may still be desirable to grant change permissions for a given settings model across all sites - for example, a Communications team managing social media accounts across all of an organisation's sites. Superusers will automatically receive global permissions, but it may not be appropriate to grant these users superuser status.

Since sites are a flat list, rather than a hierarchy like collections and pages, there is no equivalent of assigning permission rules at the "root level" to apply them globally, and so the only alternative is to grant the group a permission for each individual site. This is inconvenient for installations with a large number of sites, and error-prone due to the need to remember to update the permission list whenever a new site is added.

For this reason, we will preserve the ability to set permissions globally for a settings model via standard Django permission records, and respect these as part of the permission-checking logic.

This also avoids the complication of performing a data migration to move existing Django permission records to `GroupSitePermission` as part of the rollout of this feature. This would be decidedly non-trivial, especially if we wish to avoid developers having to write their own data migrations as part of upgrading Wagtail (since this would involve having a central migration somewhere within Wagtail that operates on some set of models that isn't known in advance).

### Changes to settings views

We anticipate that the changes required to the view code of `wagtail.contrib.settings` will be fairly minimal:

* EditView will use either `SitePermissionPolicy` or `ModelPermissionPolicy` for its permission policy, depending on whether the settings model is a `BaseSiteSetting` or `BaseGenericSetting` - or could perhaps use `SitePermissionPolicy` in both cases, given that it will have equivalent behaviour to `ModelPermissionPolicy` when no `GroupSitePermission` records exist.
* The `SettingMenuItem.is_shown` method will check permission using the relevant permission policy, rather than the ad-hoc `user_can_edit_setting_type` function (which only looks at basic Django permissions).
* `SiteSwitchForm` will accept the current user as an argument, and only offer sites for which the user has permission according to `SitePermissionPolicy`.

### Changes to group permission editing view

The permission records associated with each setting model and site will be editable through the Settings -> Groups admin area.

In the proposed implementation, each registered `BaseSiteSetting` model will have its own section of the form, registered through the `register_group_permission_panel` hook. In the case that multiple site settings models exist, each one will appear under its own heading: for example, "Social media settings permissions" and "Theme settings permissions".

Within each section, the permissions will be presented as a list of checkboxes, one per site:

```
Social media settings permissions
---------------------------------

[x] blog.example.com
[x] jobs.example.com
[ ] intranet.example.com

Theme settings permissions
--------------------------

[x] blog.example.com
[ ] jobs.example.com
[ ] intranet.example.com
```

On submitting the form, a `SitePermissionPolicy` record will be created for each item checked, corresponding to the `change_...` permission for the given settings model and the given site.

Additionally, settings models will continue to be listed in the "Object permissions" section of the page, as they are now, to allow granting the permission globally for all sites. We should consider extending the `register_permissions` hook to allow passing help text to display here, to explain the difference between this checkbox and the site-by-site options. For example, this might read: "Allow changing social media settings globally for all sites. To set this on a per-site basis, see 'Social media settings permissions' below."
