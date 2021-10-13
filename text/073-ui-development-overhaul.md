# RFC 73: UI development overhaul

- RFC: 73
- Author: Thibaud Colas
- Created: 2021-10-13
- Last Modified: 2021-10-13

## Abstract

This RFC proposes a programme of UI development work over a sustained 3 to 6 months’ effort, as part of which we will create a new “UI development” team to modernise Wagtail’s UI development practices so we have a strong foundation for upcoming large one-off UI improvements as well as long-term maintenance.

## Overall strategy

We will be tackling long-standing UI [technical debt](https://github.com/wagtail/wagtail/issues/3804) from October to December 2021. As part of this work, we propose to:

1. Create a UI development team to foster more community involvement.
2. Introduce more automation to QA & testing for UI dev to increase quality & reduce manual testing effort.
3. Apply the principles of [component-driven development](https://www.componentdriven.org/) to Wagtail, to increase the quality & productivity of UI development.
4. Implement long-needed behind-the-scenes UI & tooling improvements

## UI development team

The team will follow the ways of working of the [accessibility team](https://github.com/wagtail/wagtail/wiki/Accessibility-team), additionally with the goal of having a bigger time commitment from contributors over the initial 3 months, and potentially stopping the team after the first 6 months term.

---

The remainder document describes a backlog of improvements the team could go on to deliver for Wagtail, so others can help determine short-term priorities and refine our strategy.

## Testing & QA automation

### Test automation

All changes in Wagtail core. All running on every PR. Using existing [tests site](https://github.com/wagtail/wagtail/tree/main/wagtail/tests) as fixtures for now, pattern library in the future.

1.  Trial visual regression testing with [Percy](https://www.browserstack.com/percy), [Chromatic](https://www.chromatic.com/), [BackstopJS](https://github.com/garris/BackstopJS).
2.  Trial browser integration tests with either [BrowserStack Automate](https://www.browserstack.com/automate), or Playwright, or Puppeteer (see [Pleasantest](https://github.com/cloudfour/pleasantest)).
3.  Set up automated browser-based accessibility checks with Axe, either via integration tests, or [Pa11y](https://pa11y.org/).
4.  Trial accessibility tree snapshot testing ([Paul Grenier](https://medium.com/factset/the-automated-ui-testing-methodology-you-need-to-try-9ce4d8afe623), [Pleasantest](https://medium.com/factset/the-automated-ui-testing-methodology-you-need-to-try-9ce4d8afe623))

### Monitoring

- Trial Lighthouse for Wagtail admin -- either DebugBear on nightly site, or in CI as part of test automation.
- Trial Pa11y for Wagtail admin -- either pa11y-dashboard on nightly site, or in CI (see above).

### QA automation

1.  Set up Prettier [#6059](https://github.com/wagtail/wagtail/issues/6059) for JS, TS, CSS, Sass, JSON, YAML (not Markdown?)
2.  Set up [djhtml](https://github.com/rtts/djhtml) & [curlylint](https://www.curlylint.org/) for templates
3.  Trial [Axe linter GitHub app](https://github.com/apps/axe-linter)
4.  Upgrade Wagtail's [ESLint configuration](https://github.com/wagtail/eslint-config-wagtail)
5.  Make sure [all JS code is linted](https://github.com/wagtail/wagtail/pull/7536)
6.  Upgrade Wagtail's [stylelint configuration](https://github.com/wagtail/stylelint-config-wagtail)

## Component-driven development

### Pattern library for Django templates

- Replace manual styleguide with [django-pattern-library](https://torchbox.github.io/django-pattern-library/)
- Finish icons componentization [#6107](https://github.com/wagtail/wagtail/issues/6107)
- Introduce components as templates for more of the UI beyond icons
- Refactor UI components to [co-locate template, styles, JS, docs](https://github.com/wagtail/wagtail/issues/3804#user-content-co-locate)
- Hook up automated tests to pattern library

### Storybook

- Finish Storybook setup: icons [#7198](https://github.com/wagtail/wagtail/pull/7198)
- Finish Storybook setup: Storyshots [#7176](https://github.com/wagtail/wagtail/pull/7176)
- Trial [storybook-django](https://github.com/torchbox/storybook-django) to bring Django UI components to Storybook
- Hook up automated tests to Storybook

### Refactoring to components

- Finish component styles refactoring [#5015](https://github.com/wagtail/wagtail/issues/5015)
- Plan for JS components refactoring [#5252](https://github.com/wagtail/wagtail/issues/5252)
- Wagtail design system packaging & documentation

## UI & tooling improvements

- Remove remaining IE11 hacks [#6170](https://github.com/wagtail/wagtail/search?q=IE11&unscoped_q=IE11)
- Post-IE11 architecture changes & low-hanging fruits [#6170](https://github.com/wagtail/wagtail/issues/6170)
- Right-to-left (RTL) language support
- Frontend performance optimisations
- Fully replace Gulp scripts with Webpack
- Get Assistiv Labs sponsorship for Wagtail
