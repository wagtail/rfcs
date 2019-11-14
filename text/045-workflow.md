# RFC 45: Workflow

* RFC: 45
* Authors: Karl Hobley
* Created: 2019-11-12
* Last Modified: 2019-11-12

## Abstract

This RFC proposes how we can implement multi-stage configurable workflow in Wagtail core.

## Overview

### Workflows

Workflows are configured in Wagtail by an administrator. Each workflow has a name and a sequence of steps.

Workflows are triggered when a user clicks the "Submit for Review" action in the page editor. This creates a "workflow state" object and starts the first step in the workflow.
The steps in the workflow must be completed in the same order as defined by the administrator. When the workflow is completed, the page is published.

Wagtail finds the workflow to use from the closest ancestor page that has a workflow associated with it. Out of the box, the "Root" page in Wagtail has the "Default" workflow linked with it, so this will be used in any section that hasn't got a more specific workflow assigned to it.

The "Submit for Review" button is not shown when there isn't a workflow configured.

### Tasks

Each step in a workflow has a task that must be completed in order to go to the next step. Tasks are defined separately to workflows, allowing them to be to shared.

Examples of tasks include reviews by members of specific user groups, examples for this could be "Editorial Review" or "Legal Review".
Different task types can be defined by developers using models. This allows custom tasks to be defined. For example, this could be used to implement automated grammar and spelling checks.

When a task starts, it creates a "task state" object. This object tracks the current progress for completing a task in a workflow for an individual revision. The "task state" object may also be customised
for each task type allowing fields like "review comment" to be implemented, for example.

If the page changes while a step task is taking place, the workflow will return to the first step of the workflow and any "in progress" tasks will be cancelled. Any "task state" objects from previous steps will be kept indefinitely, and will be available for uses such as generating diffs between versions. All "task state" objects can be viewed by users in the "Workflow history" interface.

## Models

### Task

The ``Task`` model is the base that all task types inherit. It is a concrete model, so tasks use multi-table-inheritance.

This model has an ID and Name fields. It also has the following methods that are overridable:

#### ``start(workflow_state)``

This is called when the task is started in a workflow.

This is responsible for creating an instance of ``TaskState``, which itself could be customised per task type.

#### ``get_action_menu_items(workflow_state, task_state)``

This returns a list of ``ActionMenuItem`` instances to replace the defaults. This could be used to implement "Accept" and "Reject" actions.

#### ``on_action(workflow_state, task_state, action_name)``

If the edit view receives a ``POST`` request with a field called ``wagtail-moderation-action``, then this method is called
with the action name passed through.

#### ``user_can_access_editor(workflow_state, task_state, user)``

Returns ``True`` if the user is allowed to access the page editor. By default, this returns ``False``.

If this returns ``False``, Wagtail will fall back to querying ``UserPagePermissionsProxy`` so this can't be used to revoke editor permissions for a user that would usually have access to this.
This is to prevent broken links in the admin as areas of the admin which link to the edit view (such as the explorer) won't be aware of workflows.

To prevent users from editing pages while the task is active, use ``page_locked_for_user`` instead.

Also, this does not make the page appear in the explorer if the user wouldn't normally have permission to edit the page.

#### ``page_locked_for_user(workflow_state, task_state, user)``

Returns ``True`` if the page should be locked for the specified user. By default, this returns ``False``.

Note that this method won't unlock the page if it is locked to a specific user using Wagtail's edit lock feature.

#### ``page_lockable_by_user(workflow_state, task_state, user)``

Returns ``True`` if the page can be locked by the specified user. By default, this queries ``UserPagePermissionsProxy``.

#### ``page_unlockable_by_user(workflow_state, task_state, user)``

Returns ``True`` if the page can be unlocked by the specified user. By default, this queries ``UserPagePermissionsProxy``.

### Workflow

The ``Workflow`` model has a ``name`` field and an orderable link model to ``Task`` called ``WorkflowStep``. It is not customisable.

The ``Page`` model has a ``ForeignKey`` to ``Workflow``. This specifies which workflow to use on the page and its descendants.

### WorkflowState and TaskState

These two models store the current status of active workflows and keep a history of all past workflows.

``WorkflowState`` has the following fields:

 - ``id``
 - ``workflow`` - A ``ForeignKey`` to ``Workflow``
 - ``page`` - A ``ForeignKey`` to ``Page``
 - ``status`` - Either "In progress" (default), "Approved", "Rejected" or "Cancelled"
 - ``created_at``
 - ``requested_by``
 - ``current_task_state`` - A ``ForeignKey`` to the most recent ``TaskState``

``TaskState`` has the following fields:

 - ``id``
 - ``workflow_state`` - A ``ForeignKey`` to ``WorkflowState``
 - ``page_revision`` - A ``ForeignKey`` to ``PageRevision``
 - ``status`` - Either "In progress" (default), "Approved", "Rejected" or "Skipped"
 - ``started_at``
 - ``finished_at``

``TaskState`` is overridable per task type. This is done by overriding the ``start`` method on ``Task`` to return a different ``TaskState`` model.

#### Adding/removing/reordering tasks when they are in use

If an admin adds a new task or reorders tasks in the workflow management interface, the current task always needs to be completed before going back to any task that was just added, even if the new task is before the current task.

For example, say we have the following workflow configured:

 Editor Approval => Clinical Approval

If a new "Grammar Check" task is added in between, any pages that are currently on "Clinical Approval" will move to "Grammar Check" after they complete "Clinical Approval", even though "Clinical Approval" is last.

If a task is removed from a workflow, any pages that are currently on that task will have to either complete it or cancel the workflow and start again.

Administrators can't permanently delete tasks if any page is currently on that task.

## UI Changes

### Page editor

When the user visits the page editor, the view checks for an active workflow on the page. If there is, certain behaviours can be overridden by the implementation of the current task in the workflow:

 - What action buttons appear at the bottom
 - Whether the page is locked/lockable/unlockable
 - Whether the edit view is accessible to a user who hasn't got edit permission on the page (to allow non-editors to review pages)

There will also be a section in the header which shows the current workflow that is going on and what tasks have been completed.

#### Submit for moderation action button

Currently, the “Submit for moderation" button saves a revision with the “submitted for moderation” field set to True.

This is changed to use the new workflow models instead. Clicking “Submit for moderation” creates an instance of “workflow state” on the page and then starts the first task of that workflow for the current page revision.

The workflow chosen is based on the position of the page in the tree. The “Submit for moderation” button is not displayed when there is no workflow configured for the section of the page tree or when there is a workflow currently in progress.

### The built-in workflow

The existing “moderation” feature of Wagtail is replaced with a workflow called “Default”. The existing “submitted for moderation” field will still exist, and revisions submitted in this way will still exist in the moderation queue. But all new pages submitted for moderation will use the new moderation workflow described by this RFC.

This workflow will work mostly the same as before; the main difference is that when a page is in review, it will no longer be editable by editors who do not have publish permission. They are able to edit again once the page has been published or the moderation rejected. If they need to edit the page, they can “Cancel” the workflow (if they created), then edit the page, then resubmit. Administrators are allowed to cancel any workflow.

Another difference is the user interface that the moderators will use. The page can be “Accepted” of “Rejected” using buttons in the page editor action menu or the “Edit bird” on the frontend. Users can no longer accept or reject pages in moderation from the dashboard unless those pages were submitted using the previous method (these won't be overridable by the workflow).

### New workflow management interface

This interface allows a user to configure workflows and tasks from the admin interface. It’s open to anyone who has the “edit” permission on the Workflow model (by default, this would be administrators only).

The task types that the administrator can choose from are implemented in code as a model. This interface is used to manage instances of the tasks, add them to workflows and hook them into the tree.

### New Workflow history interface

This interface allows editors, moderators and administrators to see the results of all past workflows on a page.

Custom ``Task`` models are able to override how their results display in this interface to allow information like review comments to be displayed.

### Reports

The "Workflow" report is accessible from under the new "Reports" top-level menu item. This shows all active workflows on pages that the user has permission to edit. The list is filterable by workflow and current task.
