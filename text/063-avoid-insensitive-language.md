# RFC 63: Avoid insensitive language

* RFC: 63
* Author: Naomi Morduch Toubman
* Created: 2020-11-02
* Last Modified: 2020-11-02

## Abstract

Aside from maintaining the status quo, there is no benefit to using insensitive language.
Even if we disagree about whether a given language choice is acceptable, the benefits of avoiding language that may hurt people outweigh the inconvenience of changing it.

## Specification

### Renaming our `master` branches to `main`

As per [github/renaming/README](https://github.com/github/renaming), later this year (2020), GitHub plans to remove some of the last remaining hurdles to changing the default branch name.
The remaining change that most affects Wagtail is retargeting open pull requests to the new branch.

Our plan will be to change to using `main` for all Wagtail repos as soon as GitHub offers automatic retargeting of open pull requests, or during January 2021, whichever comes sooner.

### Linting for insensitive language

We will run [alex](https://alexjs.com/), make changes based on what it finds, and consider whether we want to add it to our checks.
Adding it may require adding some comments telling it to ignore lines of certain files, or configuration telling it to ignore specific rules or file types.
If running it automatically seems like it will cause a lot of problems, we will make running alex, and addressing issues it raises, part of the release process.

## Questions

### Open questions

* Is this the appropriate timing for the branch name change?
* Is alex the best tool for linting language?
* What will alex check other than text files?
Are there types of files it will miss when run automatically?

### Resolved questions

* Is `main` the best option for the new branch name?
  * Yes, this is what GitHub has adopted as the default.