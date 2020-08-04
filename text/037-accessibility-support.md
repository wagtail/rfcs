# RFC 37: Making Wagtail Accessible for users of assistive technologies

- RFC: 37
- Author: Thibaud Colas
- Created: 2019-04-25
- Last Modified: 2019-04-25

## Abstract

For users of assistive technologies, using Wagtail’s admin interface is difficult.[\[1\]][1] This is at odds with Wagtail’s general focus on user experience. For organisations choosing between CMSes, this might make Wagtail a bad option – because of concerns with equality in the workplace[\[6\]][6][\[8\]][8], and legislation that mandates compliance with accessibility standards[\[2\]][2][\[3\]][3][\[4\]][4][\[5\]][5].

Accessibility for users of all abilities should be a core goal of Wagtail, and we should make the necessary fixes for its admin interface to be usable and compliant.

## Specification

### Goals

The one goal is to make Wagtail accessible. I broke this down into three layers to separate the fundamentals of improving the user experience, from compliance requirements, and process improvements.

1. Enable users of assistive technology to efficiently manage and edit content in a Wagtail powered site.[\[1\]][1]
2. Meet accessibility standards - this means making Wagtail ‘perceivable, operable, understandable and robust’ for all users. **We will target the international accessibility standard, WCAG 2.1, to the AA conformance level.**
3. Make universal design and accessibility core parts of Wagtail’s design and development process, so future changes to Wagtail also meet our targeted standards of usability and compliance.

Non-goals:

- Target WCAG2.1 to the AAA conformance level, as this would require too drastic of changes to Wagtail’s user experience.[\[7\]][7]
- Target other country-specific standards (e.g. Section508, EU directives, UK 2018 regulation, NZ Web a11y), as these are too specific to given locales to be realistic targets for Wagtail, and in majority based on WCAG2 anyway.[\[8\]][8]

---

### 1. Enable users of Assistive technology to efficiently use Wagtail

Here are high-level tasks which once done should allow us to achieve the primary goal of making Wagtail accessible for all users.

#### UI overview map

Make a map of Wagtail’s admin UI, to determine which parts of the CMS UI to target in accessibility audits and ongoing testing.

- Create a map of all of the Wagtail UIs that could be audited – individual views, UI states and modals within pages, success/error/loading states, UI variations based on the amount of content like pagination and search results.
- Categorise the different parts of the Wagtail UI based on their perceived importance for common user journeys in the day to day administration of a site.
- Take into account upcoming changes to the CMS that will affect its UI (e.g. new StreamField, new "Images/documents use" views, new TableBlock, etc)

> WIP: [Wagtail accessibility compliance UI overview](https://docs.google.com/spreadsheets/d/1rTMilzKhcb43Y3fZKGJJeoKWLCR00bfgwVB9f6OaC2U/edit)

#### Initial accessibility audit

Identify issues to address to enable users of assistive technology to use Wagtail.

- Ideally done by Wagtail contributors with accessibility testing experience, so we can be confident in the audit’s comprehensiveness.
- Identify tools to use when auditing the UI (first draft: [Axe](9), [Pa11y](10) (“Pally”), VoiceOver)
- Automated audit of all of the Wagtail UI
- Manual audit of a representative subset of the Wagtail UI
- Create an audit report with pass/fail for different components of WCAG 2.1 AA

#### Initial accessibility issues backlog

Convert issues found in the audit into dev-level tasks ready to be estimated & implemented.

- Combine relevant issues together into easily actionable dev tasks
- Identify potential overlap with existing work (e.g. StreamField rebuild, CSS refactorings)
- Prioritise fixes based on the impact of issues on the user experience
- Have UX & dev accessibility experts go through the backlog of issues to determine solutions where the problem is non-trivial.

> WIP: [WCAG2.1 AA compliance project board](https://github.com/wagtail/wagtail/projects/5)

#### Training & accessibility awareness

Identify training resources for contributors to Wagtail to understand key principles of accessibility.

- To be referenced in Wagtail’s official documentation
- Identify training resources for manual testing with assistive technologies so the core team has more testing expertise
- Create a manual checklist of acceptance criteria for accessibility reviews

#### Tooling R&D

Identify tools to use for automated testing, and dev workflow integration.

- Development workflow
- Code analysis, for fast developer feedback
  - Linting tools
- In-browser analysis, for more comprehensive checks
  - Browser extension: (Axe, …Lighthouse, WAVE (higher coverage, but more noise))
- Automated testing, for full coverage
  - Browser automation: Pa11y
- Identify resources for manual accessibility testing

> WIP: [#4871 – [Discussion] choose appropriate tool for WCAG auto testing / visual regression tests](https://github.com/wagtail/wagtail/issues/4871)

#### Tooling setup

Set up the tools that have been identified.

- As part of CI (continuous integration)
- As part of the developer environment
- Document usage of tools that cannot be installed automatically
- Document methodology manual accessibility tests
- Add accessibility testing checklist for the PR review process

> WIP: [#5245 – Accessibility support targets & tooling setup](https://github.com/wagtail/wagtail/pull/5245)

#### UI development

Fix identified issues.

- For problems that require UX consideration, if this hasn't been done while reviewing the backlog, involve UX/design to determine a good solution.
- Prioritise changes depending on their impact on day to day CMS usage, and effort of implementation.
- Pull requests for individual issues, or group related issues together, as appropriate.
- Identify reusable UI components that would make the UI more consistent and make it easier for future developments to maintain accessibility - compliance.
- Review/testing by members of the Wagtail core team, and ideally people experienced with accessibility testing.

#### State Wagtail’s commitment to accessibility

Have a statement on Wagtail’s main GitHub repository or on its website, advertising the project’s commitment to accessibility.

Example: https://www.drupal.org/about/features/accessibility

---

### 2. Meet WCAG 2.1 AA accessibility standard compliance

Includes all tasks to make Wagtail accessible, and:

#### Professional accessibility audit

Commission an accessibility audit of a representative subset of the administrative interface by a professional auditor. The audit should be representative enough, and the auditor professional enough, to make a formal statement on Wagtail’s level of compliance with our target accessibility standard. Here are examples of organisations offering this type of service:

- https://digitalaccessibilitycentre.org/
- https://www.rnib.org.uk/rnib-business
- https://abilitynet.org.uk/accessibility-services/products-and-services
- https://www.deque.com/

#### State Wagtail’s compliance with international standards

Have a statement on Wagtail’s main GitHub repository or on its website, stating Wagtail’s compliance with our target accessibility standards.

Example: https://www.drupal.org/about/features/accessibility

---

### 3. Make universal design and accessibility part of Wagtail’s design and development process

**🚧This section requires further input from UX and design experts.** Includes all of the above, and:

#### Document Wagtail targets for accessibility from a design perspective

Have high-level recommendations for what should be taken into consideration when designing interfaces. This could be similar in spirit to the [GDS Dos and don'ts posters](https://accessibility.blog.gov.uk/2016/09/02/dos-and-donts-on-designing-for-accessibility/).

#### Define accessible design tokens for Wagtail

Wagtail already has a definition of its brand guidelines and design tokens in its [styleguide](http://docs.wagtail.io/en/v2.5/contributing/styleguide.html). These should be reviewed to make sure they match with our accessibility support targets.

> WIP: [#4628 – Add accessible color combinations to the Styleguide](https://github.com/wagtail/wagtail/pull/4628)

#### Have accessibility as a principle of a Wagtail design system

Wagtail already has a pattern library, but it could benefit from establishing a more mature[\[11\]][11] design system to structure its development. Having accessibility as a principle of such a design system would make it much easier to consistently build UIs that match our support goals, without having to retrofit accessibility after the fact[\[12\]][12].

Meaningful steps towards this would be to:

- Do a comprehensive design audit
- Agree on the final format of the design system (Sketch? React components? HTML + CSS snippets?)
- Agree on how the design system would evolve as part of Wagtail’s features development and release process
- Identify components to port from the current style guide of snippets to the design system

#### Cover assistive technologies in user testing

When testing user flows of Wagtail, it should be possible to recruit users of assistive technologies for them to feed back into feature development.

## Open Questions

- What is the best workflow when making and reviewing contributions to Wagtail’s UI, to make sure they match our support targets?
- What would be the best way to finance a professional accessibility audit of Wagtail?

## References

[1]: https://github.com/wagtail/wagtail/issues/4199#issue-288601594
[2]: https://www.digital.govt.nz/standards-and-guidance/nz-government-web-standards/web-accessibility-standard-1-0/
[3]: https://www.section508.gov/
[4]: https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=uriserv:OJ.L_.2016.327.01.0001.01.ENG
[5]: http://www.legislation.gov.uk/uksi/2018/952/introduction/made
[6]: https://en.wikipedia.org/wiki/Universal_design
[7]: https://www.w3.org/TR/WCAG21/#conformance-reqs
[8]: https://www.w3.org/WAI/policies/
[9]: https://github.com/dequelabs/axe-core
[10]: http://pa11y.org/
[11]: https://medium.com/slalom-engineering/a-maturity-model-for-design-systems-93fff522c3ba
[12]: https://www.deque.com/shift-left/

- [Issue #4199: Making Wagtail Accessible for users with disabilities][1]
- [NZ: Web Accessibility Standard][2]
- [US: Section 508][3]
- [EU: Directive on the accessibility of public sector websites and mobile applications.][4]
- [GB: The Public Sector Bodies (Websites and Mobile Applications) (No.2) Accessibility Regulations 2018][5]
- [Wikipedia: Universal design][6]
- [WCAG2.1 conformance levels][7]
- [W3C: Web Accessibility Laws and Policies][8]
- [Axe: Accessibility engine for automated Web UI testing][9]
- [Pa11y: your automated accessibility testing pal][10]
- [A maturity model for design systems][11]
- [Deque: What is Shift Left Accessibility Testing?][12]

## Credits

- Most of the problem definition from the perspective of assistive technology users comes from [@mush42](https://github.com/mush42)’s issue [#4199](https://github.com/wagtail/wagtail/issues/4199).
- The problem definition from a compliance perspective is based on guidance from the [UK GDS](https://gds.blog.gov.uk/2018/09/24/how-were-helping-public-sector-websites-meet-accessibility-requirements/)
- The high-level tasks for this RFC are derived from an accessibility plan for a [Torchbox](https://torchbox.com/wagtail-cms/) government client.
- High-level tasks are further informed by a past company-wide [OKR](https://en.wikipedia.org/wiki/OKR) at Torchbox.
- The general guidance to make accessibility part of the design and development process is inspired by Deque’s advocacy for a [“Shift left” approach to accessibility testing](https://www.deque.com/shift-left/).
- Thanks to participants to the [#accessibility Slack channel](https://github.com/wagtail/wagtail/wiki/Slack), [@hippogriffic](https://github.com/hippogriffic), [@SimonDEvans](https://github.com/SimonDEvans), and [@Katielocke3](https://github.com/Katielocke3) for their feedback on early versions of this RFC.
