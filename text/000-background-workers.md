# RFC 1: Background workers

* RFC: 1
* Author: Jake Howard, with help from the Performance sub-team
* Created: 2021-09-17
* Last Modified: 2021-09-17

## Abstract

Wagtail currently doesn't have a first-party solution for long-running tasks. Other CMSs in the ecosystem such as WordPress and Drupal have background workers, allowing them to push tasks into the background to be processed at a later date, without requiring the end user to wait for them to occur.

One of the key goals behind this proposal is removing the requirement for the user to wait for tasks they don't need to.

This proposed implementation specifically doesn't assume anything about the user's setup. This not only reduces the chances of Wagtail conflicting with any existing task system implemented by applications, but also allows it to work with almost any hosting environment a user might be using.

## Background

Some tasks done as part of certain Wagtail requests don't need to block the user, and could instead be pushed to the background, improving the perceived responsiveness of the application. Having a first-party solution would also remove the need for downstream users to build a background worker pipeline themselves.

A prime example of this kind of improvement is re-indexing pages. Currently, when a user publishes a page, the "Publish" action also re-indexes the page, which slows down the request unnecessarily. The user doesn't need to wait for the indexes to be updated, meaning they could continue with whatever they need to do next faster. By moving tasks into the background, it also means longer tasks don't tie up the application server, meaning it should be able to handle more editor traffic.

Other CMSs such as WordPress and Drupal have background workers to accelerate these kinds of non-blocking tasks. These APIs allow both for the tools themselves to push tasks to the background, but also for users to submit tasks themselves.

## Requirements

This feature has some basic requirements for it to be considered "complete":

- Wagtail's background tasks should be opt-in, and Wagtail should function as it does now without it.
- Users should be able to choose from either running a persistent background process, or periodic execution with cron
- Users should have multiple options for task backends, depending on their scale and hosting environment. By default, Redis and Django's ORM should be supported.
- Users should be able to easily add their own tasks to be executed, whether through Wagtail hooks or entirely manually.
- Tasks should be able to specify a priority, so they can be executed sooner, regardless of when they were submitted.
- Users should need to neither know nor care about the specific implementation details. This includes both the implementation details, and which backend is being used (mostly applicable to library authors)

## Implementation

The proposed implementation is powered by [Huey](https://huey.readthedocs.io), a Python task scheduler which is both composable enough for our needs, and appears stable enough to be comfortable relying on it. Huey provides the building blocks for task storage, execution and submission.

By default, Huey has no integrations with Django, allowing it to work in isolation, reducing any potential impact on people's existing sites. Huey does ship with an [optional Django integration](https://huey.readthedocs.io/en/latest/contrib.html#django), but it makes several assumptions about how Huey is used, and would conflict with users already using Huey or with any extended customization we may need.

Whilst this proposal also covers scheduled tasks for Wagtail, enabling those should be opted in separately to task scheduling.

### Why Huey?

The decision to use Huey was not taken lightly. A number of different queuing and worker libraries were reviewed, including but not limited to [Celery](https://docs.celeryproject.org), [`django-db-queue`](https://github.com/dabapps/django-db-queue), [`django-lightweight-queue`](https://github.com/thread/django-lightweight-queue/), [RQ](https://python-rq.org/), and [APScheduler](https://apscheduler.readthedocs.io/).

Probably the largest package in this space, Celery, would feature-wise do everything we'd need it to and more, however has one key drawback: Complexity. This background worker needed to both be sufficiently distinct from anything else the user might be doing around background workers, and also simple enough to get up and running. Celery is neither of those things.

There are a few critical features which Huey does well which weren't all available in the other offerings:

Wagtail's background workers needed to be sufficiently separate from any other background workers used on the project. DBQ and DLQ both deeply integrate with Django, and whilst it's possible to have a dedicated queue just for wagtail, it may cause issues and concerns for consumers. Huey doesn't know or care about Django, and so even if a user were already using Huey, it could still be used within Wagtail without conflicts.

Wagtail's background worker also needed to assume as little as possible about the environment it was being run in, especially around what to use as a job store. This meant any which didn't support multiple backends were immediately discarded.

Wagtail is used by a number of large and high-profile companies, meaning using smaller libraries was less desirable from a maintenance and supply-chain security perspective.

APScheduler and Huey made the short list, but at time of writing support for out-of-process workers in APScheduler is still work-in-progress and unreleased. Huey not only ticked all the boxes in terms of features, but was also sufficiently simple to work with and integrate, and popular enough to feel confident adding it as a dependency for Wagtail.

## Proposed API

Similarly to Django's caching framework, a global "background" object can be imported, which is used to add tasks. This will be a "Huey" instance, potentially subclassed with some Wagtail sugar. This global will be pre-configured based on the application's settings such as backend, immediate mode, and connection details.

```python
from wagtail.contrib.tasks import background

@background.task()
def do_a_task():
    pass

# And now, run the task
do_a_task()
```

Using this object, tasks are submitted using the existing Huey constructs and patterns.

For Wagtail [hooks](https://docs.wagtail.io/en/stable/reference/hooks.html), there will be an additional property passed when registering the hook, which will transparently convert the hook to a task, and ensure it's submitted as a task when the hook should be called. Only certain hooks will support background tasks. Others, such as those for registering URLs or menu items must be run synchronously, and so will ignore the background argument. This will only be applicable for certain hooks, and will do nothing when passed to these hooks.

```python
from wagtail.core import hooks

@hooks.register('name_of_hook', background=True)
def my_hook_function(arg1, arg2...)
    pass
```

Whilst it's possible to define tasks anywhere in an application, the convention will be to put them alongside hooks in the `wagtail_hooks.py` file, to ensure they're imported at the right times.

### Settings

```python
WAGTAIL_TASKS = {
    BACKEND: str  # Module path to the backend to use for tasks (default empty)
    BACKEND_OPTIONS: dict  # Any additional options the backend may take (eg connection parameters) (default empty)
}
```

Most of the settings are passed through to Huey as [its configuration](https://huey.readthedocs.io/en/latest/api.html) with little to no modification.

Because Huey doesn't handle "Immediate" tasks as a distinct backend, a blank `BACKEND` is synonymous with immediate mode.

## Current workarounds

For Wagtail's internals, it's currently not possible to control how these are executed. For user-controlled code, it's possible to implement a task queueing system separate from Wagtail, and manually submit tasks to it as needed.

For scheduled tasks, Wagtail currently relies on Django's management commands, which requires the user to use a tool such as cron or Heroku's Scheduler to execute them.

## Implementation plan

1. Contribute cron-style consumer to Huey
2. Contribute a Django ORM based storage backend for Huey
3. Create the basic plumbing and configuration required to get a worker running as a part of the Wagtail codebase, based off the existing Huey constructs
4. Enable creating wagtail hooks as tasks
5. Documentation
6. Begin migrating background-compatible bits of Wagtail to tasks
7. Documentation
8. Initial release?
9. Complete migrating background-compatible bits of Wagtail to tasks

## Open Questions

- How will / should this interact with the ongoing "Bulk Actions" work?
- Should the executors be a unified management command with an additional flag, or 2 distinct commands?
- Should we not use Huey and write something ourselves?
- Is this _contrib_?
- Should Huey be an optional dependency (thus requiring us to implement "immediate mode" ourselves) or a required dependency.
- The background runner should probably have a name of some sort (and be consistent around terminology eg tasks)
- Should Django ORM support be a day-1 feature? (Wider support base, but delays release)
