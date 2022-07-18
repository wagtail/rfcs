# RFC XXX: Public roadmap

* RFC: XXX
* Author: Phil Dexter
* Created: 2022-07-18
* Last Modified: 2022-07-18

## Abstract
A few years ago Wagtail had a couple of attempts at a [public roadmap]([url](https://github.com/wagtail/wagtail/projects?query=is%3Aclosed)), using Github’s projects feature. This was stopped because at that time there was a general feeling that in most cases the roadmap was outside our control - features came in because they were unexpectedly contributed or a client needed them, or because it made sense to tackle issue x before merging PR y.

However, things have changed. We now have more investment in Wagtail and more infrastructure around its development, be that through the community, open source projects, or sponsorship.

I’d like to propose that we start using this again as a public roadmap for all things Wagtail; **a single place where the community can go to see an accurate list of upcoming changes, as well as some of those we’d like to see happen in the future.**

## Specification
We will host the roadmap on Github. It's where most of the current and future Wagtail evangilists already are, and it avoids the spread of sproadic tooling choices. 

The Github project will follow the pattern that [Github themselves use]([url](https://github.com/orgs/github/projects/4247/views/1)). Cards will follow the Summary, _Intended Outcome_, _How will it work_ pattern. They will be a high level overview of the change with links to more documentation, rather than anything detailed. The public could comment on or emoji like certain items.

Items only go onto the roadmap once they have been considered (by the core team). These items might, or might not need an RFC for implementation details before they get on the roadmap.

Items that have a hopeful reelase will be put into the column for that release. Items without any agreed delivery mechanism will go into the Future column. This Future column may act as a way to attract contributions and/or sponsorship. We could also make use of labels to help guide this (e.g. Needs sponsorship, Needs contribution).

Once an item is delivered we’ll remove it from the board - the release notes will act as the record of its delivery.

Only items of significant interest to the community will go onto the roadmap. This definition is purposefully vague at the moment, and should be reviewed on a regular basis.

## Resolved questions
### What will we do with the old pages?
Any we can control redirects from will point to the new roadmap. The rest we'll change the content on the point at the new roadmap.

## Unresolved questions
Where should the roadmap live? In the main Wagtail repository or some sub-repo?

## What should the roadmap cover?
Should the roadmap cover only Wagtail core, or also things like wagtail-localise, or any other larger scale third party packages or distributions?

