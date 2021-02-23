# RFC 63: Avoid insensitive language

* RFC: 63
* Author: Naomi Morduch Toubman
* Created: 2020-11-02
* Last Modified: 2021-02-03

## Abstract

Aside from maintaining the status quo, there is no benefit to using insensitive language.
Even if we disagree about whether a given language choice is acceptable, the benefits of avoiding language that may hurt people outweigh the inconvenience of changing it.

## Specification

### Renaming our `master` branches to `main`

As per [github/renaming/README](https://github.com/github/renaming), GitHub has removed the last remaining hurdles to changing the default branch name,
including retargeting open pull requests to the new branch.
We will change to using `main` for all Wagtail repos.

### Checking for insensitive language

We will do a review of the language used in our codebase, and make changes to insensitive language that we find.
Going forward, we will check for insensitive language as part of the release process.
To avoid adding future instances of insensitive language,
we may add a linter for insensitive language to our checks.

### Adding documentation

We will add official contributors’ documentation on how to write with inclusion in mind, and why it matters.

## Questions

### Open questions

(None)

### Resolved questions

* Is `main` the best option for the new branch name?
  * Yes, this is what GitHub has adopted as the default.
* When will GitHub add automatic retargeting of PRs?
  * This was done in mid-January 2021.
* Should we use a linter such as [alex](https://alexjs.com/)?
  * We will leave this as an implementation detail.
