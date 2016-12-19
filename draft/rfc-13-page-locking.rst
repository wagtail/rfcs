=====================================
RFC 13: Page locking
=====================================

:RFC: 13
:Author: Roel Bruggink
:Status: Draft
:Created: 2019-12-19
:Last-Modified: 2019-12-19

.. contents:: Table of Contents
   :depth: 3
   :local:

Abstract
========
Provide a way to prevent user overriding changes when multiple users open a page for editing.

Specification
=============
a. anyone who can edit a page can acquire a lock.
b. anyone who can edit a page can take the lock.
c. the lock must result in a validation error if user is not the lock owner.

Acquiring a lock
----------------
1. On opening the editing interface for a unlocked page, a lock on that page is acquired.
2. On opening the editing interface for a lockd page, an intermediate page is shown. The page allows to take the lock.
3. The lock is released using the actions at the bottom of the screen.

Locking mechanism
-----------------
Two field will be added to the Page schema: lock_userid and lock_datetime.
a. The field lock_ownerid is a FK to user, alike the owner_id field.
b. The lock_datetime contains the datetime on which the lock has been acquired.
c. The fields should NOT be part of the editing interface.

Open Questions
==============
Should a lock automatically be released?
- This is possible, but only wanted when the lock can automatically be prolonged, ie by using websockets or regural ajax calls.
