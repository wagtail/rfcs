# RFC 40: User management improvements

* RFC: 40
* Author: Karl Hobley
* Created: 2019-05-02
* Last Modified: 2019-05-02

## Abstract

This RFC includes three proposals for improvements we can make to user management in Wagtail.

## Task A - Make new users set their own password using a link that is emailed to them

When users are created, currently the admin sets their email address and password. The email address is never verified and passwords set in this way are usually weak and don’t have to be changed which can lead to multiple users having the same password.

We should remove the password fields from the create user form, and instead email the user a link to a form where they can set their password themselves. Similar to the password reset feature.

This has two main advantages:

- Their email address is verified - which is important in case they want notifications or need to use the password reset feature in the future
- Their password is likely to be stronger and unguessable by other users

A disadvantage is this requires email to be configured in order to create new users.  I think we could either rely on the admin finding the change password box on the edit page or keep those fields on the create user form but conceal them a bit to discourage their use.

## Task B - Add setting to require verified email addresses

Right now, admins can create users with any email address and users can change their email address to anything they want.

Some projects might use the user’s email address as their identifier in external systems, so allowing users to change their email address without verification could open up security issues.

We should add a setting `WAGTAIL_REQUIRE_EMAIL_VERIFICATION` which, when set to `True` (it’ll default to `False` for backwards compatibility) requires the user to verify their new email address before the change is made in the database.

## Task C - Disabling user management

Some sites may use SSO for authentication so usernames, emails and passwords (and possibly other basic information) could be managed by an external system.

We should make it possible to easily disable all user management features in Wagtail to prevent updates to fields that are managed by external systems.

Currently it’s not possible to remove `wagtailusers` from `INSTALLED_APPS` as the `UserProfile` model is defined there.

I think we should do the following to allow completely disabling account management in Wagtail:

- Convert the users admin interface into an admin module (Groups is already a module)
- Allow admin modules to be easily unregistered
- Add a setting to allow disabling the “Change email” function for users `WAGTAIL_USER_CHANGE_EMAIL_ENABLED`
