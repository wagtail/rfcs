# RFC 45: Workflow

* RFC: 45
* Authors: Karl Hobley
* Created: 2019-11-12
* Last Modified: 2019-11-12

## Abstract

This RFC proposes how we can implement multi-stage configurable workflow in Wagtail core.

## Overview

### Workflows

Workflows are configured in Wagtail by an administrator. Each workflow has a name and a list of tasks arranged in a sequence.

Workflows are triggered when a user clicks the "Submit for Review" action in the page editor. This action creates a "workflow state" object and begins the first task in the workflow.

The workflow that gets used depends on the position of the page in the tree. Wagtail finds the workflow to use from the closest ancestor page that has a workflow associated with it. Out of the box, the "Root" page in Wagtail has the "Default" workflow linked with it, so pages use this workflow if none of their ancestor pages has overridden it.

The "Submit for Review" button is not shown when there isn't a workflow configured.

### Tasks

Tasks represent the checks that must pass for a page to be published. Each task has a name, a type, and some additional configuration dependant on the chosen type.

Tasks are defined separately to workflows, allowing workflows to share tasks.

Tasks are started when the previous task completes successfully, or when it's the first task in the workflow. Starting a task creates a "task state" object and executes any actions that the task type has defined (For example, send an email).

The "task state" object tracks the current progress for completing a task; these are associated with a workflow state and a page revision. Depending on the task type, The "task state" may be an instance of TaskState or an instance of a custom model that inherits TaskState.

If the page is changed while a review is taking place, the existing "task state" objects will no longer be valid because they are associated with the previous revision. A new set of task state objects are created, and the review starts again at the first task.

"task state" objects from previous successful reviews of the page would be available for uses such as generating diffs from the last reviewed version.

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

Returns ``True`` if the user is allowed to access the page editor. By default, this queries ``UserPagePermissionsProxy``.

Note that this can only grant access to a user who doesn't usually have access to the editor and not take this away. To prevent users from editing pages, use ``page_locked_for_user`` instead.

Also, this does not make the page appear in the explorer.

#### ``page_locked_for_user(workflow_state, task_state, user)``

Returns ``True`` if the page should be locked for the specified user. By default, this queries ``UserPagePermissionsProxy``.

#### ``page_lockable_by_user(workflow_state, task_state, user)``

Returns ``True`` if the page can be locked by the specified user. By default, this queries ``UserPagePermissionsProxy``.

#### ``page_unlockable_by_user(workflow_state, task_state, user)``

Returns ``True`` if the page can be unlocked by the specified user. By default, this queries ``UserPagePermissionsProxy``.

### Workflow

The ``Workflow`` model has a Name and an ordered list of tasks. It is not customisable.

The ``Page`` model has a ``ForeignKey`` to ``Workflow``. This specifies which workflow to use on the page and its descendants.

### WorkflowState and TaskState

These two models store the current status of active workflows and keep a history of all past workflows.

``WorkflowState`` has the following fields:

 - ``id``
 - ``workflow`` - A ``ForeignKey`` to ``Workflow``
 - ``page`` - A ``ForeignKey`` to ``Page``
 - ``status`` - Either "In progress", "Approved", "Rejected" or "Cancelled"
 - ``created_at``
 - ``requested_by``
 - ``current_task_state`` - A ``ForeignKey`` to the most recent ``TaskState``

``TaskState`` has the following fields:

 - ``id``
 - ``workflow_state`` - A ``ForeignKey`` to ``WorkflowState``
 - ``page_revision`` - A ``ForeignKey`` to ``PageRevision``
 - ``status`` - Either "In progress", "Approved", "Rejected" or "Skipped""
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
