# RFC 35: Sub teams

* Authors: Karl Hobley, Naomi Morduch Toubman
* Created: 2019-04-18
* Last Modified: 2019-09-30

The Wagtail core team is relatively small and largely focused on the development of Wagtail. Wagtail would benefit from the input of a wider variety of community members.

Sub teams will supplement the core team with more diverse teams focused on specific areas, each led by a member of the core team.

The core team will still be responsible for setting the overall goals and vision of Wagtail and will decide on sub team creation, leadership, and disbandment.

The benefits of this approach include:

- More focus, the sub-teams can be more narrowly focused on each area giving it the care needed
- Enable more 'levels' of involvement, volunteers can fill a gap in a sub team which may be slightly more commitment than just a contributor but without the commitment of a core team member
- May help with processing PRs faster, giving a clearer touch point of who to ask for PR review from the core team (as they will be a sub team leader of an area that may match the focus of a PR)

## The sub teams

Sub teams are delegated by the core team and a participating core team member is assigned.

This RFC doesn’t propose to create any yet, since ideas for sub teams will be discussed after this RFC is accepted.

The sub teams that will most strengthen our work will be defined by interest area and involve people with varied skills (e.g. design, marketing, community management) and viewpoints (e.g. designers, experts, stakeholders).

Examples of interest areas:

- UX
- Community (events, community management, etc)
- Documentation
- Accessibility

Development area sub teams can be created for development that focuses on a specific part of Wagtail or an installable module.
In addition, temporary sub teams may be spun up to work on a single feature, or to continue work started at sprints.

Examples of development areas and features:

- Personalisation
- Internationalisation
- Search
- wagtailtrans
- wagtail-whoosh
- react-streamfield
- Draftail
- Wagtail import export

As a rule of thumb, any sub team should have at least one core team member, however certain areas of interest or temporary teams could be spun up without a core team member in it.

### Sub-teams in GitHub

We use a mix of labels (e.g. `Accessibility`) and GitHub teams (e.g. `@wagtail/documentation`).
Any area with a label that matches a sub team name is owned by the given sub team. Mismatched labels should
be updated for clarity.


## Team leaders

Each sub team has a leader who is a member of the core team. Leaders don't have to be an expert on their team's area, nor do they need to contribute code in that area, but they need to be passionate about making that part of Wagtail better. If a team leader decides they no longer want to be a team leader or thinks their team is no longer useful, they must hand their leadership to another core team member or recommend disbanding the sub team.

What team leaders do:

- Decide how they will manage their sub team and its membership
- Keep the core team and their team up to date on their area
- Work with other leaders on issues that cross multiple sub team areas

In addition, they may also:

- Be a first point of contact for new contributors interested in their area
- Help members find things to work on
- Find reviewers for work done in their area
- Write blog posts about their team’s work
- Manage/moderate the public Slack channel for that area
- Aid in directing responses for area specific questions on [StackOverflow](https://stackoverflow.com/questions/tagged/wagtail)
- Monitoring for common issues and requests direct to Github issues.
- Assist in labelling GitHub issues for that area

## Members

Like core team membership, new members must be approved by existing members. Sub teams may also have minimum participation requirements for their members. Members who are no longer participating should be removed from the team.

All sub team members are made members of the Wagtail organisation on Github and are welcome to use the badge on their profile. They may be given commit rights to repos but not the main Wagtail repo (core team members only).

### Special mentions

A honorary `Alumni` sub team will be created to list and thank previous core or sub team members.

## Inspiration

This RFC takes some inspiration from the Governance RFC of the Rust Programming Language project.
You can find their RFC here:

    https://github.com/aturon/rfcs/blob/rust-governance/text/0000-rust-governance.md

and their list of sub teams here:

    https://www.rust-lang.org/governance/

While I’ve taken bits out of that RFC I’ve taken into account some differences between the two projects:

- Rust has many more full time core team members than Wagtail has
    - I expect Wagtail sub teams would work at a much slower pace as, in most cases, team leaders would be using their spare time
    - They also have more time to write RFCs -- sorry this RFC isn’t as detailed as theirs!
- The Rust project has many components to it (compiler, libs, dev tools) whereas Wagtail is just one
    - Because of this, the Rust core team is made up of only sub team leaders and they are still able to have a development focused core team. I am not suggesting that we change the core team membership of Wagtail.
- They make most of their decisions through an RFC process, which we don’t do so much so I haven’t included any mention of RFCs here.

