# RFC 72: Background workers

* RFC: 72
* Author: Jake Howard, with help from the Performance sub-team
* Created: 2021-09-17
* Last Modified: 2024-02-05

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

The proposed implementation will be in the form of an application wide "Job backend". This backend will be what connects Wagtail to the task runners. The job backend will provide an interface for either 3rd party libraries, or application developers to specify how jobs should be created and pushed into the background

The default backend will not push jobs into the background, instead running them in-band. This means the change is backwards compatible with current wagtail.

Wagtail will also ship with an ORM-powered backend for users to transition too easy should they want the powers of a background worker without a large infrastructure migration or commitment. Whilst this backend will be designed to be performant enough, and considered fine for production use, it won't be designed to fully scale to all needs, and so tools such as Redis still have a place.

Whilst this proposal also covers scheduled tasks for Wagtail, enabling those should be opted in separately to task scheduling.

## Proposed API

### Backends

A backend will be a class which extends a wagtail-defined base class.

```python
from datetime import datetime
from typing import Callable, Dict, List

from wagtail.contrib.tasks import BaseJobBackend

class MyBackend(BaseJobBackend):
    def __init__(self, options: Dict):
        """
        Any connections which need to be setup can be done here
        """
        super().__init__(options)

    def enqueue(self, func: Callable, args: List, kwargs: Dict) -> None:
        """
        Queue up a job function to be executed
        """
        ...

    def defer(self, func: Callable, when: datetime, args: List, kwargs: Dict) -> None:
        """
        Add a job to be completed at a specific time
        """
        ...

    async def aenqueue(self, func: Callable, args: List, kwargs: Dict) -> None:
        """
        Queue up a job function (or coroutine) to be executed
        """
        ...

    async def adefer(self, func: Callable, when: datetime, args: List, kwargs: Dict) -> None:
        """
        Add a job function (or coroutine) to be completed at a specific time
        """
        ...
```

If a backend doesn't support a particular scheduling mode, it simply does not define the method.

### Running tasks

Similarly to Django's caching framework, a global "background" object can be imported, which is used to add tasks.

```python
from wagtail.contrib.tasks import background

def do_a_task(*args, **kwargs):
    pass

# Submit the task to be run
background.enqueue(do_a_task, args=[], kwargs={})
```

In the initial implementation, `enqueue` will not return anything. A response object which tracks in-flight tasks may be possible in a future iteration.

`args` and `kwargs` are intentionally their own dedicated arguments to make the API simpler and backwards-compatible should other attributes be added in future.

#### Deferring tasks

Tasks may also be "deferred" to run at a specific time:

```python
background.defer(do_a_task, args=[], kwargs={}, when=datetime.datetime.now() + datetime.timedelta(minutes=5))
```

When scheduling a task, it may not be exactly that time a task is executed, however it should be accurate to within a few seconds.

#### Background Hooks

For Wagtail [hooks](https://docs.wagtail.io/en/stable/reference/hooks.html), there will be an additional property passed when registering the hook, which will transparently convert the hook to a task, and ensure it's submitted as a task when the hook should be called. Only certain hooks will support background tasks. Others, such as those for registering URLs or menu items must be run synchronously, and so will raise an error. This will only be applicable for certain hooks, and will do nothing when passed to these hooks.

```python
from wagtail.core import hooks

@hooks.register('name_of_hook', background=True)
def my_hook_function(arg1, arg2...)
    pass
```

It will still be possible to manually run `background.enqueue` etc inside a hook method, this is merely a convenience wrapper.

Whilst it's possible to define tasks anywhere in an application, the convention will be to put them alongside hooks in the `wagtail_hooks.py` file, to ensure they're imported at the right times.

#### Async tasks

Where the underlying implementation supports it, backends may also provide an `async`-compatible interface for scheduling tasks, using `a`-prefixed methods:

```python
await background.aenqueue(do_a_task, args=[], kwargs={})
```

### Settings

Values shown below are the default.

```python
WAGTAIL_JOBS = {
    "BACKEND": "wagtail.contrib.tasks.backends.ImmediateBackend",
    "OPTIONS": {}  # To pass to the backend's constructor
}
```

Note that only a single backend may be defined for an application. Should multiple task runners be required, this should be implemented in a custom backend.

### Checks

To ensure the configuration is valid, a check will be added to validate:

- If `SCHEDULE` is set, does the backend support scheduling?

## Current workarounds

For Wagtail's internals, it's currently not possible to control how these are executed. For user-controlled code, it's possible to implement a task queueing system separate from Wagtail, and manually submit tasks to it as needed.

## Implementation plan

1. Create the basic plumbing and base classes required
2. Implement (`async`-compatible) `ImmediateBackend`
3. Enable creating wagtail hooks as tasks
4. Implement an ORM backend
6. Begin migrating background-compatible bits of Wagtail to tasks
7. Documentation
8. Initial release?
9. Complete migrating background-compatible bits of Wagtail to tasks

## Open Questions

- How will / should this interact with the ongoing "Bulk Actions" work?
- Is this _contrib_?
- The background runner should probably have a name of some sort (and be consistent around terminology eg tasks)
- Should Wagtail contain some additional backends for commonly used task runners?
  - Or, should these be separate packages under the Wagtail organisation?
- What should the argument for scheduling a task be? Should Wagtail parse it into a format more consumable for the backend?
- Should the ORM backend use a custom task runner, or a 3rd-party package?
