# RFC 41: organisation wide project guidelines

- RFC: 41
- Author: Jonny Scholes
- Created: 2019–06–05
- Last Modified:
  2019–06–05

## Abstract

As Wagtail grows and more projects are added to the central organisation, there should be
a consistent set of guidelines and standards applied to all projects.

## Motivation

The wagtail core project has a set of guidelines for contributors to follow when
submitting code/design.
Consisting of both coding practices, expected contributor behaviour and guiding principles,
they have been carefully put together with the aim of maintaining a productive and
professional project and working ethic.

Wagtail occasionally produces separate packages for wagtail either to extend functionality
(“plugins”) or as a way to preview functionality or use legacy functionality.
Often packages are brought into the Wagtail org that are maintained by developers who are
not core developers or who are less involved in contributions to Wagtail core.

This presents us with some challenges as it means the community and other potential
contributors see projects listed under the Wagtail Github organisation as “Wagtail
approved” projects and so may reasonably assume a few things:

- The same contributor guidelines are in place
- Wagtail core will ensure a level of competency within the project
- That Wagtail core developers are the correct people to reach out to for
  help/advice/concerns about those projects
- That should any security/moral issues arise with those projects that, Wagtail (or possibly
  Torchbox) are responsible for resolving these issues
- That they would have the same license as Wagtail core

The aim of this document and its related actions is to attempt to resolve some if not all
of these issues by providing a template or process for creating new projects under the
Wagtail org. Whilst also ensuring we are not too heavily burdened and we can still both
create new packages easily (eg a proof of concept package) and move packages into the
Wagtail organisation easily.

## Proposed measures

The following are a series of rules that will be documented and will need to be met before
a package can be added to the Wagtail organisation.

A Wagtail project template will also be produced that includes all of these to make
migration simpler (see what [Webpack](https://github.com/webpack-contrib/webpack-defaults)
does as a good example).

### Code of conduct

All Wagtail's org projects must adhere to [Wagtail’s code of conduct)[https://github.com/wagtail/wagtail/blob/master/CODE_OF_CONDUCT.md]
and must reference it in their readme file.

### Contributing guidelines

All Wagtail org projects must adhere to the same set of contributing guidelines.

We have a [contributing to Wagtail](https://docs.wagtail.io/en/v2.5/contributing/) section in
our docs. Each project must include a dedicated CONTRIBUTING.md file in the root of the
project that links to Wagtail’s contributing guidelines.

Amendments are allowed, given in some cases you may need some specific and additional
instructions to be followed.
However package maintainers are asked in good faith not to provide amendments that go
against the sentiment/purpose of anything in the base contributing document.

### Code style

All Wagtail org projects must adhere to the same coding guidelines.

Code configuration will be turned into packages where possible to make including in
projects easier, the project template will be the source of truth for Wagtail’s code style
“opinions”.

### Maintainers/maintenance

All repos must include a “Maintainers/maintenance” heading.

Under this heading lists either who the person to contact might be, how to contact them
(probably github issues) and if there is no one - a statement saying that no one is
currently maintaining this project however you can submit a PR and ping a Wagtail core
group and they _might_ respond.

### License

All repos _must_ have a LICENSE file.

As Wagtail is BSD, using the BSD license is preferred. However the main thing is that the chosen
license is a “permissive and open source compatible license that allows commercial use”.

## Open Questions

Should each project have their own sub-team?
Are we suggesting a license - or enforcing one?
