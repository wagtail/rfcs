# RFC 079: Public roadmap

- RFC: 079
- Author: Phil Dexter, Thibaud Colas
- Created: 2022-07-18
- Last Modified: 2022-10-19

## Abstract

A few years ago Wagtail had a couple of attempts at a [public roadmap](https://github.com/wagtail/wagtail/projects?query=is%3Aclosed), using GitHub’s Projects feature. This was stopped because at that time there was a general feeling that in most cases the roadmap was outside our control - features came in because they were unexpectedly contributed or a client needed them, or because it made sense to tackle issue x before merging PR y.

However, things have changed. We now have more investment in Wagtail and more infrastructure around its development, be that through the community, open source projects, or sponsorship.

I’d like to propose that we start using this again as a public roadmap for all things Wagtail; **a single place where the community can go to see an accurate list of upcoming changes, as well as some of those we’d like to see happen in the future**.

## Specification

### What goes on the roadmap

Only items of significant interest to the community will go onto the roadmap. This definition is purposefully vague at the moment, and should be reviewed on a regular basis.

Items only go onto the roadmap once they have been considered by the core team. These items might, or might not need an RFC for implementation details before they get on the roadmap.

### Where the roadmap lives

We will host the roadmap on GitHub. It's where most of the current and future Wagtail evangelists already are, and it avoids the spread of sporadic tooling choices.

The GitHub project will follow the pattern that [GitHub themselves use](https://github.com/orgs/github/projects/4247/views/1). Cards will follow the Summary, _Intended Outcome_, _How will it work_ pattern. They will be a high level overview of the change with links to more documentation, rather than anything detailed. The public could comment on or emoji-react to certain items.

High-level items will be created as issues on a new `roadmap` repository, where the README will document our roadmap process. Just like GitHub, the contents of the roadmap will be visible as an organisation-level Projects board, so we can pull items from the `roadmap` repository as well as any organisation repository if relevant.

Release column names on the roadmap will follow the time-based release schedule with an LTS yes/no indicator, as our [version numbering scheme](https://github.com/wagtail/wagtail/discussions/7732) means we don’t know release numbers ahead of time.

### Roadmap lifecycle

Items that have a hopeful release will be put into the column for that release. Items spanning multiple releases / long-term architecture changes will have a dedicated column. Items without any agreed delivery mechanism will go into a "Future" column. This Future column will act as a way to attract contributions and/or sponsorship on items that align with our long-term plans. We will make use of labels to help guide this: "Needs sponsorship", "Needs contribution" to start with.

Once an item is delivered we’ll remove it from the board - the release notes will act as the record of its delivery.

### Overlap with milestones and backlog refinement

The roadmap will replace the already-removed "some day" and "real soon now" milestones used in GitHub, though only for large areas of improvements. It will help for high-level release planning of future "n + 1" releases, while we will keep using the milestones for detailed planning.

## Resolved questions

### What will we do with the old pages?

Any pages we can control redirects from will point to the new roadmap. The rest we'll change the content on the point at the new roadmap.

### Where should the roadmap live?

The majority of roadmap items will be GitHub issues created on a dedicated `wagtail/roadmap` repository. They will be visible on an org-wide GitHub Projects board.

### What should the roadmap cover?

The roadmap is primarily meant to cover Wagtail core improvements of all types, but it can also be used for any substantial effort within the Wagtail ecosystem. Whether something is included on the roadmap is up to the core team.

## Unresolved questions

None!

## Implementation plan

This section isn’t binding as part of the RFC but provided here to foster relevant feedback.

- [ ] Create the new `roadmap` repository with high-level documentation of the roadmap process as presented in this RFC.
- [ ] Create a new GitHub project in the Wagtail organisation, with columns as presented in this RFC.
- [ ] Populate the roadmap for the next release with significant items tracked on the existing milestone.
- [ ] Populate the "Future" roadmap with items noted on the [the outdated roadmap](https://github.com/wagtail/wagtail/wiki/Roadmap)
- [ ] Add [Page editor areas earmarked for future contributions](https://wagtail.org/blog/build-update-on-new-page-editor/) to the Future column.
- [ ] Redirect the [the outdated roadmap](https://github.com/wagtail/wagtail/wiki/Roadmap) to the new roadmap.
- [ ] Add Roadmap links on wagtail.org, docs.wagtail.org Slack, and any other relevant documents.
- [ ] Share new roadmap via a blog post
- [ ] Core team discussion on process to manage roadmap additions (exact process TBC outside the RFC).
- [ ] Liaise with marketing team / What’s New in Wagtail to ensure alignment between showcased work and the roadmap
