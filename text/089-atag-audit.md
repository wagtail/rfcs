# [RFC 089](https://github.com/wagtail/rfcs/pull/89): ATAG audit & accessibility roadmap

- [RFC: 089](https://github.com/wagtail/rfcs/pull/89)
- Author: Thibaud Colas
- Created: 2023-10-31
- Last Modified: 2024-12-06

## Abstract

This RFC is an audit report of Wagtail 6.3 for conformance with the [Authoring Tool Accessibility Guidelines (ATAG) 2.0 standard](https://www.w3.org/TR/ATAG20/). This standard outlines requirements for content management systems like Wagtail, along two main areas:

1. [Making the authoring tool user interface accessible](https://www.w3.org/TR/ATAG20/#part_a)
2. [Supporting the production of accessible content](https://www.w3.org/TR/ATAG20/#part_b)

This audit report is created as an RFC for multiple reasons:

- Ease of reviewing and critiquing of specific findings from the audit, by all members of our community as relevant.
- Validation and acceptance of the audit’s findings by Wagtail’s core team.
- Validation and acceptance of recommended actions in particular.

Validating the audit’s results will help make sure the findings as part of upcoming roadmap items:

- [Admin accessibility improvements #71](https://github.com/wagtail/roadmap/issues/71)

## Audit findings overview

At the success criteria level as per the [W3C ATAG report tool](https://www.w3.org/WAI/atag/report-tool/), pass/fail:

- Total: 28 **Pass**, 22 **Fail**, 13 **Not applicable**.
- Part A: 14 **Pass**, 13 **Fail**, 4 **Not applicable**.
- Part B: 14 **Pass**, 9 **Fail**, 9 **Not applicable**.

At the success criteria level, by [level of conformance](https://www.w3.org/TR/ATAG20/#intro_understand_levels_conformance):

- Level A: 11 **Pass**, 7 **Fail**, 6 **Not applicable**.
- Level AA: 13 **Pass**, 8 **Fail**, 7 **Not applicable**.
- Level AAA: 4 **Pass**, 7 **Fail**, 0 **Not applicable**.

At the guidelines level as per the [W3C Authoring Tools list](https://www.w3.org/WAI/tools-list/authoring/), with partial support:

- Total: 6 **Pass**, 14 **Partially**, 2 **Fail**, 2 **Not applicable**.
- Part A: 2 **Pass**, 9 **Partially**, 1 **Fail**, 1 **Not applicable**.
- Part B: 4 **Pass**, 5 **Partially**, 1 **Fail**, 1 **Not applicable**.

### [A. Make the authoring tool user interface accessible](#a-make-the-authoring-tool-user-interface-accessible)

- [A.1. Authoring tool user interfaces follow applicable accessibility guidelines](#a1-authoring-tool-user-interfaces-follow-applicable-accessibility-guidelines)
  - **Partially**: [A.1.1. (For the authoring tool user interface) Ensure that web-based functionality is accessible](#a11-for-the-authoring-tool-user-interface-ensure-that-web-based-functionality-is-accessible)
    - **Fail**: [A.1.1.1 Web-Based Accessible (WCAG)](#a111-web-based-accessible-wcag) (Level A / AA / AAA)
  - **Not applicable**: [A.1.2. (For the authoring tool user interface) Ensure that non-web-based functionality is accessible](#a12-for-the-authoring-tool-user-interface-ensure-that-non-web-based-functionality-is-accessible)
    - **Not applicable**: [A.1.2.1 Accessibility Guidelines](#a121-accessibility-guidelines) (Level A)
    - **Not applicable**: [A.1.2.2 Platform Accessibility Services](#a122-platform-accessibility-services) (Level A)
- [A.2. Editing-views are perceivable](#a2-editing-views-are-perceivable)
  - **Partially**: [A.2.1. (For the authoring tool user interface) Make alternative content available to authors](#a21-for-the-authoring-tool-user-interface-make-alternative-content-available-to-authors)
    - **Fail**: [A.2.1.1 Text Alternatives for Rendered Non-Text Content](#a211-text-alternatives-for-rendered-non-text-content) (Level A)
    - **Fail**: [A.2.1.2 Alternatives for Rendered Time-Based Media](#a212-alternatives-for-rendered-time-based-media) (Level A)
  - **Partially**: [A.2.2. (For the authoring tool user interface) Ensure that editing-view presentation can be programmatically determined](#a22-for-the-authoring-tool-user-interface-ensure-that-editing-view-presentation-can-be-programmatically-determined)
    - **Fail**: [A.2.2.1 Editing-View Status Indicators](#a221-editing-view-status-indicators) (Level A)
    - **Not applicable**: [A.2.2.2 Access to Rendered Text Properties](#a222-access-to-rendered-text-properties) (Level AA)
- [A.3. Editing-views are operable](#a3-editing-views-are-operable)
  - **Partially**: [A.3.1. (For the authoring tool user interface) Provide keyboard access to authoring features](#a31-for-the-authoring-tool-user-interface-provide-keyboard-access-to-authoring-features)
    - **Fail**: [A.3.1.1 Keyboard Access (Minimum)](#a311-keyboard-access-minimum) (Level A)
    - **Pass**: [A.3.1.2 No Keyboard Traps](#a312-no-keyboard-traps) (Level A)
    - **Pass**: [A.3.1.3 Efficient Keyboard Access](#a313-efficient-keyboard-access) (Level AA)
    - **Fail**: [A.3.1.4 Keyboard Access (Enhanced)](#a314-keyboard-access-enhanced) (Level AAA)
    - **Fail**: [A.3.1.5 Customize Keyboard Access](#a315-customize-keyboard-access) (Level AAA)
    - **Pass**: [A.3.1.6 Present Keyboard Commands](#a316-present-keyboard-commands) (Level AAA)
  - **Partially**: [A.3.2. (For the authoring tool user interface) Provide authors with enough time](#a32-for-the-authoring-tool-user-interface-provide-authors-with-enough-time)
    - **Fail**: [A.3.2.1 Auto-Save (Minimum)](#a321-auto-save-minimum) (Level A)
    - **Pass**: [A.3.2.2 Timing Adjustable](#a322-timing-adjustable) (Level A)
    - **Pass**: [A.3.2.3 Static Input Components](#a323-static-input-components) (Level A)
    - **Fail**: [A.3.2.4 Content Edits Saved (Extended)](#a324-content-edits-saved-extended) (Level AAA)
  - **Fail**: [A.3.3. (For the authoring tool user interface) Help authors avoid flashing that could cause seizures](#a33-for-the-authoring-tool-user-interface-help-authors-avoid-flashing-that-could-cause-seizures)
    - **Fail**: [A.3.3.1 Static View Option](#a331-static-view-option) (Level A)
  - **Partially**: [A.3.4. (For the authoring tool user interface) Enhance navigation and editing via content structure](#a34-for-the-authoring-tool-user-interface-enhance-navigation-and-editing-via-content-structure)
    - **Not applicable**: [A.3.4.1 Navigate By Structure](#a341-navigate-by-structure) (Level AA)
    - **Pass**: [A.3.4.2 Navigate by Programmatic Relationships](#a342-navigate-by-programmatic-relationships) (Level AAA)
  - **Partially**: [A.3.5. (For the authoring tool user interface) Provide text search of the content](#a35-for-the-authoring-tool-user-interface-provide-text-search-of-the-content)
    - **Fail**: [A.3.5.1 Text Search](#a351-text-search) (Level AA)
  - **Pass**: [A.3.6. (For the authoring tool user interface) Manage preference settings](#a36-for-the-authoring-tool-user-interface-manage-preference-settings)
    - **Pass**: [A.3.6.1 Independence of Display](#a361-independence-of-display) (Level A)
    - **Pass**: [A.3.6.2 Save Settings](#a362-save-settings) (Level AA)
    - **Pass**: [A.3.6.3 Apply Platform Settings](#a363-apply-platform-settings) (Level AA)
  - **Pass**: [A.3.7. (For the authoring tool user interface) Ensure that previews are at least as accessible as in-market user agents](#a37-for-the-authoring-tool-user-interface-ensure-that-previews-are-at-least-as-accessible-as-in-market-user-agents)
    - **Pass**: [A.3.7.1 Preview (Minimum)](#a371-preview-minimum) (Level A)
    - **Pass**: [A.3.7.2 Preview (Enhanced)](#a372-preview-enhanced) (Level AAA)
- [A.4. Editing-views are understandable](#a4-editing-views-are-understandable)
  - **Partially**: [A.4.1. (For the authoring tool user interface) Help authors avoid and correct mistakes](#a41-for-the-authoring-tool-user-interface-help-authors-avoid-and-correct-mistakes)
    - **Fail**: [A.4.1.1 Content Changes Reversible (Minimum)](#a411-content-changes-reversible-minimum) (Level A)
    - **Pass**: [A.4.1.2 Settings Change Confirmation](#a412-settings-change-confirmation) (Level A)
    - **Pass**: [A.4.1.3 Content Changes Reversible (Enhanced)](#a413-content-changes-reversible-enhanced) (Level AAA)
  - **Partially**: [A.4.2. (For the authoring tool user interface) Document the user interface, including all accessibility features](#a42-for-the-authoring-tool-user-interface-document-the-user-interface-including-all-accessibility-features)
    - **Pass**: [A.4.2.1 Describe Accessibility Features](#a421-describe-accessibility-features) (Level A)
    - **Fail**: [A.4.2.2 Document All Features](#a422-document-all-features) (Level AA)

### [B. Support the production of accessible content](#b-support-the-production-of-accessible-content)

- [B.1. Fully automatic processes produce accessible content](#b1-fully-automatic-processes-produce-accessible-content)
  - **Pass**: [B.1.1. Ensure that automatically-specified content is accessible](#b11-ensure-that-automatically-specified-content-is-accessible)
    - **Pass**: [B.1.1.1 Content Auto-Generation After Authoring Sessions (WCAG)](#b111-content-auto-generation-after-authoring-sessions-wcag) (Level A / AA / AAA)
    - **Pass**: [B.1.1.2 Content Auto-Generation During Authoring Sessions (WCAG)](#b112-content-auto-generation-during-authoring-sessions-wcag) (Level A / AA / AAA)
  - **Pass**: [B.1.2. Ensure that accessibility information is preserved](#b12-ensure-that-accessibility-information-is-preserved)
    - **Not applicable**: [B.1.2.1 Restructuring and Recoding Transformations (WCAG)](#b121-restructuring-and-recoding-transformations-wcag) (Level A / AA / AAA)
    - **Pass**: [B.1.2.2 Copy-Paste Inside Authoring Tool (WCAG)](#b122-copy-paste-inside-authoring-tool-wcag) (Level A / AA / AAA)
    - **Not applicable**: [B.1.2.3 Optimizations Preserve Accessibility](#b123-optimizations-preserve-accessibility) (Level A)
    - **Not applicable**: [B.1.2.4 Text Alternatives for Non-Text Content are Preserved](#b124-text-alternatives-for-non-text-content-are-preserved) (Level A)
- [B.2. Authors are supported in producing accessible content](#b2-authors-are-supported-in-producing-accessible-content)
  - **Partially**: [B.2.1. Ensure that accessible content production is possible](#b21-ensure-that-accessible-content-production-is-possible)
    - **Fail**: [B.2.1.1 Accessible Content Possible (WCAG)](#b211-accessible-content-possible-wcag) (Level A / AA / AAA)
  - **Pass**: [B.2.2. Guide authors to produce accessible content](#b22-guide-authors-to-produce-accessible-content)
    - **Pass**: [B.2.2.1 Accessible Option Prominence (WCAG)](#b221-accessible-option-prominence-wcag) (Level A / AA / AAA)
    - **Not applicable**: [B.2.2.2 Setting Accessibility Properties (WCAG)](#b222-setting-accessibility-properties-wcag) (Level A / AA / AAA)
  - **Partially**: [B.2.3. Assist authors with managing alternative content for non-text content](#b23-assist-authors-with-managing-alternative-content-for-non-text-content)
    - **Pass**: [B.2.3.1 Alternative Content is Editable (WCAG)](#b231-alternative-content-is-editable-wcag) (Level A / AA / AAA)
    - **Pass**: [B.2.3.2 Automating Repair of Text Alternatives](#b232-automating-repair-of-text-alternatives) (Level A)
    - **Fail**: [B.2.3.3 Save for Reuse](#b233-save-for-reuse) (Level AAA)
  - **Partially**: [B.2.4. Assist authors with accessible templates](#b24-assist-authors-with-accessible-templates)
    - **Fail**: [B.2.4.1 Accessible Template Options (WCAG)](#b241-accessible-template-options-wcag) (Level A / AA / AAA)
    - **Pass**: [B.2.4.2 Identify Template Accessibility](#b242-identify-template-accessibility) (Level AA)
    - **Not applicable**: [B.2.4.3 Author-Created Templates](#b243-author-created-templates) (Level AA)
    - **Fail**: [B.2.4.4 Accessible Template Options (Enhanced)](#b244-accessible-template-options-enhanced) (Level AAA)
  - **Not applicable**: [B.2.5. Assist authors with accessible pre-authored content](#b25-assist-authors-with-accessible-pre-authored-content)
    - **Not applicable**: [B.2.5.1 Accessible Pre-Authored Content Options](#b251-accessible-pre-authored-content-options) (Level AA)
    - **Not applicable**: [B.2.5.2 Identify Pre-Authored Content Accessibility](#b252-identify-pre-authored-content-accessibility) (Level AA)
- [B.3. Authors are supported in improving the accessibility of existing content](#b3-authors-are-supported-in-improving-the-accessibility-of-existing-content)
  - **Partially**: [B.3.1. Assist authors in checking for accessibility problems](#b31-assist-authors-in-checking-for-accessibility-problems)
    - **Pass**: [B.3.1.1 Checking Assistance (WCAG)](#b311-checking-assistance-wcag) (Level A / AA / AAA)
    - **Not applicable**: [B.3.1.2 Help Authors Decide](#b312-help-authors-decide) (Level A)
    - **Not applicable**: [B.3.1.3 Help Authors Locate](#b313-help-authors-locate) (Level A)
    - **Pass**: [B.3.1.4 Status Report](#b314-status-report) (Level AA)
    - **Fail**: [B.3.1.5 Programmatic Association of Results](#b315-programmatic-association-of-results) (Level AA)
  - **Fail**: [B.3.2. Assist authors in repairing accessibility problems](#b32-assist-authors-in-repairing-accessibility-problems)
    - **Fail**: [B.3.2.1 Repair Assistance (WCAG)](#b321-repair-assistance-wcag) (Level A / AA / AAA)
- [B.4. Authoring tools promote and integrate their accessibility features](#b4-authoring-tools-promote-and-integrate-their-accessibility-features)
  - **Pass**: [B.4.1. Ensure the availability of features that support the production of accessible content](#b41-ensure-the-availability-of-features-that-support-the-production-of-accessible-content)
    - **Pass**: [B.4.1.1 Features Active by Default](#b411-features-active-by-default) (Level A)
    - **Pass**: [B.4.1.2 Option to Reactivate Features](#b412-option-to-reactivate-features) (Level A)
    - **Pass**: [B.4.1.3 Feature Deactivation Warning](#b413-feature-deactivation-warning) (Level AA)
    - **Pass**: [B.4.1.4 Feature Prominence](#b414-feature-prominence) (Level AA)
  - **Partially**: [B.4.2. Ensure that documentation promotes the production of accessible content](#b42-ensure-that-documentation-promotes-the-production-of-accessible-content)
    - **Fail**: [B.4.2.1 Model Practice (WCAG)](#b421-model-practice-wcag) (Level A / AA / AAA)
    - **Pass**: [B.4.2.2 Feature Instructions](#b422-feature-instructions) (Level A)
    - **Fail**: [B.4.2.3 Tutorial](#b423-tutorial) (Level AAA)
    - **Fail**: [B.4.2.4 Instruction Index](#b424-instruction-index) (Level AAA)

## [A. Make the authoring tool user interface accessible](https://www.w3.org/TR/ATAG20/#part_a)

### [A.1. Authoring tool user interfaces follow applicable accessibility guidelines](https://www.w3.org/TR/ATAG20/#principle_a1)

#### [A.1.1. (For the authoring tool user interface) Ensure that web-based functionality is accessible](https://www.w3.org/TR/ATAG20/#gl_a11)

See [Implementing A.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a11).

##### A.1.1.1 Web-Based Accessible (WCAG)

> (Level A / AA / AAA). See [Implementing A.1.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a111).

**Fail**. Evaluated as: **Level AA**. Wagtail [currently targets WCAG 2.2 AA and ATAG 2.0 AA conformance](https://wagtail.org/accessibility/) for the administrative interface of the CMS. Though a lot of progress has been made, there are still [known conformance issues and possible improvements](https://github.com/orgs/wagtail/projects/9/views/1).

As a representation of the state of Wagtail’s WCAG 2.1 AA conformance, here is a summary of WCAG 2.2 AA and best practice issues across releases, for the page editor UI (tested with [Welcome to the Wagtail bakery!](https://static-wagtail-v6-3.netlify.app/admin/pages/60/edit/)):

| Version                                                               | Total | [Color contrast](https://dequeuniversity.com/rules/axe/4.4/color-contrast) | [`nested-interactive`](https://www.browserstack.com/docs/accessibility/rules/wcag/nested-interactive) | [`aria-valid-attr-value`](https://accessibilityinsights.io/info-examples/web/aria-valid-attr-value/) | [`aria-input-field-name`](https://www.browserstack.com/docs/accessibility/rules/wcag/aria-input-field-name) | [Links purpose](https://dequeuniversity.com/rules/axe/4.4/identical-links-same-purpose) | [Empty heading](https://www.browserstack.com/docs/accessibility/rules/best-practice/empty-heading) | [Heading order](https://accessibility.browserstack.com/api/more-info/4.4/heading-order) | [`region`](https://accessibility.browserstack.com/api/more-info/4.4/region) | [Landmark unique](https://www.browserstack.com/docs/accessibility/rules/best-practice/landmark-unique) |
| --------------------------------------------------------------------- | ----- | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| [6.3](https://static-wagtail-v6-3.netlify.app/admin/pages/60/edit/)   | 2     | 0                                                                          | 0                                                                                                     | 0                                                                                                    | 2                                                                                                           | 0                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [6.2](https://static-wagtail-v6-2.netlify.app/admin/pages/60/edit/)   | 2     | 0                                                                          | 0                                                                                                     | 0                                                                                                    | 2                                                                                                           | 0                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [6.1](https://static-wagtail-v6-1.netlify.app/admin/pages/60/edit/)   | 2     | 0                                                                          | 0                                                                                                     | 0                                                                                                    | 2                                                                                                           | 0                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [6.0](https://static-wagtail-v6-0.netlify.app/admin/pages/60/edit/)   | 2     | 0                                                                          | 0                                                                                                     | 0                                                                                                    | 2                                                                                                           | 0                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [5.2](https://static-wagtail-v5-2.netlify.app/admin/pages/60/edit/)   | 6     | 2                                                                          | 0                                                                                                     | 1                                                                                                    | 2                                                                                                           | 0                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 1                                                                                                      |
| [5.1](https://static-wagtail-v5-1.netlify.app/admin/pages/60/edit/)   | 6     | 2                                                                          | 0                                                                                                     | 1                                                                                                    | 2                                                                                                           | 0                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 1                                                                                                      |
| [5.0](https://static-wagtail-v5-0.netlify.app/admin/pages/60/edit/)   | 7     | 2                                                                          | 0                                                                                                     | 1                                                                                                    | 2                                                                                                           | 2                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [4.2](https://static-wagtail-v4-2.netlify.app/admin/pages/60/edit/)   | 13    | 8                                                                          | 0                                                                                                     | 1                                                                                                    | 2                                                                                                           | 2                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [4.1](https://static-wagtail-v4-1.netlify.app/admin/pages/60/edit/)   | 12    | 7                                                                          | 0                                                                                                     | 1                                                                                                    | 2                                                                                                           | 2                                                                                       | 0                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [4.0](https://static-wagtail-v4-0.netlify.app/admin/pages/60/edit/)   | 7     | 1                                                                          | 0                                                                                                     | 1                                                                                                    | 2                                                                                                           | 2                                                                                       | 1                                                                                                  | 0                                                                                       | 0                                                                           | 0                                                                                                      |
| [3.0](https://static-wagtail-v3-0.netlify.app/admin/pages/60/edit/)   | 24    | 16                                                                         | 0                                                                                                     | 2                                                                                                    | 2                                                                                                           | 2                                                                                       | 1                                                                                                  | 1                                                                                       | 0                                                                           | 0                                                                                                      |
| [2.16](https://static-wagtail-v2-16.netlify.app/admin/pages/60/edit/) | 30    | 16                                                                         | 3                                                                                                     | 3                                                                                                    | 2                                                                                                           | 2                                                                                       | 1                                                                                                  | 1                                                                                       | 1                                                                           | 0                                                                                                      |

Suggested next steps:

- More test coverage for automated accessibility tests
- More feedback from users of assistive technologies

References:

- Most recent major improvement: [Enhanced contrast admin theme](https://docs.wagtail.org/en/stable/releases/6.3.html#enhanced-contrast-admin-theme).
- [WCAG 2.2 & ATAG 2.0 CMS admin](https://github.com/orgs/wagtail/projects/9/views/1)
- [Living accessibility audit (last update: December 2024)](https://docs.google.com/spreadsheets/d/1l7tnpEyJiC5BWE_JX0XCkknyrjxYA5T2aee5JgPnmi4/edit)

Full list of 21 currently-documented accessibility issues in GitHub:

- [Rich text ctrl + K keyboard shortcut should open the link or document tooltip #11627](https://github.com/wagtail/wagtail/issues/11627)
- [Add comment button should move focus to the new comment when the panel is closed #11021](https://github.com/wagtail/wagtail/issues/11021)
- [Row headers for permission forms as tables #12288](https://github.com/wagtail/wagtail/issues/12288)
- [Status icons need alternative text #12174](https://github.com/wagtail/wagtail/issues/12174)
- [StreamField FieldBlock / widgets labels’ missing semantic association with inputs #11179](https://github.com/wagtail/wagtail/issues/11179)
- [Confusing Markdown rich text keyboard shortcuts announcement in screen readers #12107](https://github.com/wagtail/wagtail/issues/12107)
- [Contrast themes / WHCM support improvements #11449](https://github.com/wagtail/wagtail/issues/11449)
- [Admin interface must implement 3.1.2 Language of Parts for multilingual interfaces #11376](https://github.com/wagtail/wagtail/issues/11376)
- [Admin interface must implement 3.1.2 Language of Parts for multilingual websites #11375](https://github.com/wagtail/wagtail/issues/11375)
- [Revamp revision comparison diff styles #10576](https://github.com/wagtail/wagtail/issues/10576)
- [Radio inputs do not correctly get associated with their parent (group) label for accessibility #10228](https://github.com/wagtail/wagtail/issues/10228)
- [Contrast themes – Info side panel’s button links need a border #8834](https://github.com/wagtail/wagtail/issues/8834)
- [Missing HTML widget attributes in StreamField widget rendering #10300](https://github.com/wagtail/wagtail/issues/10300)
- [Barriers for speech recognition users #9666](https://github.com/wagtail/wagtail/issues/9666)
- [Accessibility regression in userbar #10674](https://github.com/wagtail/wagtail/issues/10674)
- [Page chooser should use buttons instead of links with invalid targets #5408](https://github.com/wagtail/wagtail/issues/5408)
- [Implement focus management for chooser modals #5338](https://github.com/wagtail/wagtail/issues/5338)
- [Accessibility and UI issues with typed table block #7646](https://github.com/wagtail/wagtail/issues/7646)
- [Rework the admin UI’s landmark based on established best practices #5411](https://github.com/wagtail/wagtail/issues/5411)
- [Keyboard navigation breaks with links in paragraph in Chrome on MacOS #7693](https://github.com/wagtail/wagtail/issues/7693)
- [Datetimepicker UI component is not accessible to screen-reader and keyboard users #5325](https://github.com/wagtail/wagtail/issues/5325)

There have been 19 issues fixed since the last ATAG 2.0 audit:

- [Status tag contrast is too low #5778](https://github.com/wagtail/wagtail/issues/5778)
- [Empty table header usage across the admin #11596](https://github.com/wagtail/wagtail/issues/11596)
- [Draftail (rich text editor) - using cmd + left arrow to navigate to the start of the line is broken in Firefox #9366](https://github.com/wagtail/wagtail/issues/9366)
- [Accessibility issue with 'live' link's accessible text #12611](https://github.com/wagtail/wagtail/issues/12611)
- [Sub-menus within the main menu cannot be closed on mobile #10747](https://github.com/wagtail/wagtail/issues/10747)
- [Making Wagtail Accessible for users with disabilities #4199](https://github.com/wagtail/wagtail/issues/4199)
- [Action buttons are easy to miss in list views for high contrast mode users #12126](https://github.com/wagtail/wagtail/issues/12126)
- [Minimap component needs a toggle button #12175](https://github.com/wagtail/wagtail/issues/12175)
- [Page deletion confirmation text lacks sufficient contrast #12232](https://github.com/wagtail/wagtail/issues/12232)
- [Synchronisation between wagtailuserbar and "Checks" admin panel. #12157](https://github.com/wagtail/wagtail/issues/12157)
- ["Unpublished" styles in pages listings are very hard / impossible to see #5370](https://github.com/wagtail/wagtail/issues/5370)
- [Action buttons in chooser components are cut off by overflow in 5.0 #10553](https://github.com/wagtail/wagtail/issues/10553)
- [Empty table header found in the 'Your locked pages' dashboard section #11459](https://github.com/wagtail/wagtail/issues/11459)
- [TableBlock is impossible to reach with the keyboard #8893](https://github.com/wagtail/wagtail/issues/8893)
- [Chooser buttons focus colour is too dark in dark mode #10875](https://github.com/wagtail/wagtail/issues/10875)
- [Panel anchor target sizes are too small on mobile viewports #11411](https://github.com/wagtail/wagtail/issues/11411)
- [Page editing actions dropdown doesn’t support keyboard or screen reader navigation #7366](https://github.com/wagtail/wagtail/issues/7366)
- [Label-visible content mismatch 'Admin' and 'Edit your account' in sidebar #11372](https://github.com/wagtail/wagtail/issues/11372)
- [New Dialog (modal) does not visibly show a close button if there is no message #11306](https://github.com/wagtail/wagtail/issues/11306)

#### [A.1.2. (For the authoring tool user interface) Ensure that non-web-based functionality is accessible](https://www.w3.org/TR/ATAG20/#gl_a12)

See [Implementing A.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a12).

##### A.1.2.1 Accessibility Guidelines

> (Level A). See [Implementing A.1.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a121).

**Not applicable**. Wagtail is a web-based CMS.

##### A.1.2.2 Platform Accessibility Services

> (Level A). See [Implementing A.1.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a122).

**Not applicable**. Wagtail is a web-based CMS.

### [A.2. Editing-views are perceivable](https://www.w3.org/TR/ATAG20/#principle_a2)

#### [A.2.1. (For the authoring tool user interface) Make alternative content available to authors](https://www.w3.org/TR/ATAG20/#gl_a21)

See [Implementing A.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a21).

##### A.2.1.1 Text Alternatives for Rendered Non-Text Content

> (Level A). See [Implementing A.2.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a211).

**Fail**. For icons within the CMS, all have appropriate alt text. For CMS-managed images, Wagtail renders non-text content in nine scenarios, five of which are related to editing views and would require changes:

- Fail: Image upload fields in the image edit/create form. The image’s title displays as a field next to the visuals. The title acts as alt text by default in Wagtail. This is missing a programmatic association between the title or description text, and the image.
  - Example: [Wagtail 6.3 - Editing image Boston Cream Pie](https://static-wagtail-v6-3.netlify.app/admin/images/43/)
  - Current: The alt text is permanently set to the contents of the Title field on page load.
  - Proposed actions:
    - The image could be more clearly associated with the live title and description fields with an `aria-labelledby`.
    - The description field could have help text to clarify its use as the image’s alt text (at least in the CMS).
- Fail: Image chooser fields in forms. The selected image’s title displays next to the visuals. The description acts as alt text by default in Wagtail. This is missing a programmatic association between the title text and image.
  - Example: [Wagtail 6.3 - Editing Blog Page: Desserts with Benefits, Image field](https://static-wagtail-v6-3.netlify.app/admin/pages/77/edit/#panel-child-content-image-section)
  - Current: Alt text set to `alt=""`, with title displayed after the image.
  - Proposed actions:
    - Associate the visible text with the image with `aria-labelledby`.
    - Switch to the description field as alt text in the CMS.
- Fail: Image chooser fields with a custom alt text field next to them. The custom alt text field is not programmatically associated with the image.
  - Example (with Caption field): [Wagtail 6.3 - Editing Blog Page: Desserts with Benefits, Image block](https://static-wagtail-v6-3.netlify.app/admin/pages/77/edit/#block-556e76b0-0f5a-42bb-b039-653f3d6b1f0b-section)
  - Alt text set to `alt=""`, with title displayed after the image, and custom field further down.
  - Proposed actions:
    - Implement this pattern in the bakerydemo website, based on the new Wagtail 6.3 ImageBlock.
    - Associate both the title of the image, and the custom field, with `aria-labelledby`, or a combination of it and `aria-describedby`.
- Fail: Images within rich text fields. Here the image’s title is displayed in a tooltip associated with the image, but there is no programmatic association.
  - Example: unavailable
  - Proposed actions:
    - Add demo content following this pattern in bakerydemo.
    - Add a programmatic association between tooltip text and image with `aria-labelledby`.
    - Make sure the image has alt text accessible even when the tooltip is closed.
- Fail: Embeds within rich text fields. Here we display the embed’s thumbnail if there is one. The embed’s title is displayed in a tooltip associated with the embed, but there is no programmatic association.
  - Example: [Wagtail 6.3 - Editing Blog Page: Desserts with Benefits, Paragraph block](https://static-wagtail-v6-3.netlify.app/admin/pages/77/edit/#block-ac48af95-b3be-4602-8c2f-5c43fc080f17-section)
  - Current: Alt text set to `alt=""`, with no text displayed near the image.
  - Proposed actions:
    - Add a programmatic association between tooltip text and image with `aria-labelledby`.
    - Make sure the thumbnail image has alt text accessible even when the tooltip is closed.

Outside editing views (possibly not part of ATAG requirements), Wagtail renders images in the following scenarios:

- Revisions comparison with images. The image’s title is used as alt text. The title should be visible to the user in the UI, but it is not.
  - Example: [Wagtail 6.3 - Comparing Bread and Circuses](https://static-wagtail-v6-3.netlify.app/admin/pages/68/revisions/compare/46...108/)
  - Current: the image title is used as `alt` attribute.
  - Proposed actions:
    - Add demo content following this pattern in bakerydemo.
    - Display the images’ titles in the UI, with programmatic `aria-labelledby` associations.
    - Switch to the `description` field as alt text.
- Revisions comparison with images or embeds in rich text: currently unimplemented.
- Snippets listings. When there is an image column, its alt text is set but invisible in the UI.
  - Example: [Wagtail 6.3 – Snippets People](https://static-wagtail-v6-3.netlify.app/admin/snippets/base/person/)
  - Current: the image title is used as `alt` attribute.
  - Proposed actions:
    - Display the images’ titles in the UI, with programmatic `aria-labelledby` associations.
    - Switch to the `description` field as alt text.
- Image gallery. Here we display the title underneath the image as alt text, in a `figcaption`.
  - Example: [Wagtail 6.3 – Images](https://static-wagtail-v6-3.netlify.app/admin/images/)
  - Current: Alt text set to `alt=""`, but the image is within a `figure` with the image’s title as `figcaption`.
  - Proposed actions:
    - Associate the text and the image with `aria-labelledby`.
    - Switch to the `description` field as alt text.

Recommendation for Wagtail: Consider how best to sign-post the Description field as the image’s alt text in the CMS, and potentially also in the frontend (with clear options to mark images as decorative or define alt text in context).

Reference: [RFC 97: Alt Text Capabilities](https://github.com/wagtail/rfcs/pull/97).

##### A.2.1.2 Alternatives for Rendered Time-Based Media

> (Level A). See [Implementing A.2.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a212).

**Fail**. Wagtail’s only time-based media is animated GIFs. Their text alternatives work identically to other images in Wagtail, with the same characteristics listed in SC A.2.1.1.

#### [A.2.2. (For the authoring tool user interface) Ensure that editing-view presentation can be programmatically determined](https://www.w3.org/TR/ATAG20/#gl_a22)

See [Implementing A.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a22).

##### A.2.2.1 Editing-View Status Indicators

> (Level A). See [Implementing A.2.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a221).

**Fail**. Wagtail uses the following status indicators in editing views:

- Fail: Comments on fields. Comments are displayed as a "speech bubble" icon next to the field they are associated with. The association isn’t programmatically exposed. Even visually, the presence of a comment can only be identified on hover/focus within the field’s area.
  - Example: [Editing Home Page: Welcome to the Wagtail bakery!](https://static-wagtail-v6-3.netlify.app/admin/pages/60/edit/)
  - Proposed actions:
    - Add a programmatic association between fields and their comment presence indicator
    - Make the comment presence indicator visible at all times, either for all users or users of "prefers contrast" theming.
    - (WCAG issue) Make the comment addition buttons visible at all times in commenting mode.
- Fail: Comments in rich text. Comments are displayed as highlighted text within the rich text field. The association isn’t programmatically exposed.
  - Example: none available
  - Proposed actions:
    - Add demo content following this pattern in bakerydemo.
    - Research how other WYSIWYG interfaces programmatically associate comments with runs of text.
- Pass: Character count for rich text fields. The character count is displayed as a number next to the field it is associated with. The association is programmatically exposed with `aria-describedby` ("Character count: 18/120").
  - Example [Editing Recipe page: Southern Cornbread, Preface field](https://static-wagtail-v6-3.netlify.app/admin/pages/82/edit/#panel-child-content-preface-section)
- Pass: Concurrent editing notifications. The notification status is associated with a `aria-label` and tooltip on the status button.
- TBC (work in progress): content quality checks within page editor.

Outside editing views (possibly not part of ATAG requirements), Wagtail renders status indicators in the following scenarios:

- Accessibility checks in Wagtail userbar. They are currently not programmatically associated with the content they are for.

##### A.2.2.2 Access to Rendered Text Properties

> (Level AA). See [Implementing A.2.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a222).

**Not applicable**. Wagtail doesn’t allow editing of any text properties associated with the content.

### [A.3. Editing-views are operable](https://www.w3.org/TR/ATAG20/#principle_a3)

#### [A.3.1. (For the authoring tool user interface) Provide keyboard access to authoring features](https://www.w3.org/TR/ATAG20/#gl_a31)

See [Implementing A.3.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a31).

##### A.3.1.1 Keyboard Access (Minimum)

> (Level A). See [Implementing A.3.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a311).

**Fail**. Though the majority of the authoring tool’s functionality is keyboard accessible, there are specific areas that aren’t:

- In rich text fields, pin/unpin of the rich text toolbar.
  - Example: [Wagtail 6.3 - Editing Blog Page: Tracking Wild Yeast](https://static-wagtail-v6-3.netlify.app/admin/pages/62/edit/)
  - Proposed actions:
    - Research how other WYSIWYG interfaces allow keyboard interactions with all toolbar controls.
- In rich text fields, Edit functionality for links, documents, images, embeds.
  - Examples:
    - Links: [Wagtail 6.3 - Editing Standard page: About](https://static-wagtail-v6-3.netlify.app/admin/pages/76/edit/#panel-child-content-body-section)
    - Documents: [Wagtail 6.3 - Editing Recipe page: Southern Cornbread](https://static-wagtail-v6-3.netlify.app/admin/pages/82/edit/#block-910c5024-a47a-45b1-a3a3-8f8bb5a8fa70-section)
    - Images: no example
    - Embeds: [Wagtail 6.3 - Editing Blog page: Desserts with Benefits](https://static-wagtail-v6-3.netlify.app/admin/pages/77/edit/)
  - Proposed actions:
    - Research how other WYSIWYG interfaces allow keyboard interactions with all toolbar controls.
    - Possibly support selection of all those content types and press "Enter" to move focus to their UI.
    - See [Rich text ctrl + K keyboard shortcut should open the link or document tooltip #11627](https://github.com/wagtail/wagtail/issues/11627)
- In image/document/page/task/snippet choosers, the chooser dialog.
  - See [Making Wagtail Accessible for users with disabilities #4199](https://github.com/wagtail/wagtail/issues/4199)
  - See also [Implement focus management for chooser modals #5338](https://github.com/wagtail/wagtail/issues/5338)
  - Examples:
    - Images: [Wagtail 6.3 - Editing Bread page: Arepa](https://static-wagtail-v6-3.netlify.app/admin/pages/37/edit/#panel-child-content-image-section)
    - Snippets: [Wagtail 6.3 - Editing Bread page: Arepa](https://static-wagtail-v6-3.netlify.app/admin/pages/37/edit/#panel-child-content-origin-section)
    - Documents: no example
    - Pages: [Wagtail 6.3 - Editing workflow Moderator approval](https://static-wagtail-v6-3.netlify.app/admin/workflows/edit/1/#workflow-pages-section)
    - Tasks: [Wagtail 6.3 - Editing workflow Moderator approval](https://static-wagtail-v6-3.netlify.app/admin/workflows/edit/1/#inline_child_workflow_tasks-0-panel-section)
  - Proposed actions:
    - Complete [Re-implement chooser modals with new design #9246](https://github.com/wagtail/wagtail/pull/9246)
- In image create/edit forms, creation or editing of a focal area.
  - Current: it’s impossible to set a focal area without a mouse.
  - Example: [Wagtail 6.3 - Editing Boston Cream Pie](https://static-wagtail-v6-3.netlify.app/admin/images/43/)
  - Proposed actions:
    - Add keyboard support with a new implementation
    - Factor in [possible requirements](https://github.com/wagtail/wagtail/issues/10947#issuecomment-1746464044) for other types of image manipulation.
- In table blocks, editing of the table.
  - Example: [Wagtail 6.3 – Editing Recipe page: Hot Cross Bun](https://static-wagtail-v6-3.netlify.app/admin/pages/81/edit/#block-2b9b59cb-4dd7-4ebf-ac66-1ed43471609b-section)

##### A.3.1.2 No Keyboard Traps

> (Level A). See [Implementing A.3.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a312).

**Pass**. There are no known keyboard traps in the administrative interface.

##### A.3.1.3 Efficient Keyboard Access

> (Level AA). See [Implementing A.3.1.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a313).

**Pass**. The administrative interface provides the following mechanisms to improve keyboard navigation:

- A skip link, across all views, to skip the sidebar.
- Collapsible sections on long views to avoid having to tab through all of the content.
- A mechanism to link to specific collapsible sections, for direct access via bookmarks.
- A "Collapse/expand all sections" button on long forms to navigate more easily to a specific section.
- The "minimap" skip-menu on long forms to navigate more easily to a specific section.

##### A.3.1.4 Keyboard Access (Enhanced)

> (Level AAA). See [Implementing A.3.1.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a314).

**Fail**. See _A.3.1.1 Keyboard Access_. We would expect addressing all aspects listed in _A.3.1.1_ to also address this criterion.

##### A.3.1.5 Customize Keyboard Access

> (Level AAA). See [Implementing A.3.1.5](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a315).

**Fail**. None of Wagtail’s keyboard commands can be customized.

Proposed actions:

- Implement a "key map" for Wagtail’s keyboard commands and-or command palette, with a way to upload a new key map as JSON.

##### A.3.1.6 Present Keyboard Commands

> (Level AAA). See [Implementing A.3.1.6](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a316).

**Pass**. Across specific areas:

- Pass: In rich text fields, Markdown keyboard commands or keyboard shortcuts are displayed within tooltips for specific toolbar buttons.
- Pass: In rich text fields, "command palette" commands are displayed in the block chooser, and the command palette trigger is displayed in the fields’ placeholder.
- Pass: Wagtail’s other traditional "key combinations" keyboard shortcuts are displayed in the "Keyboard shortcuts" dialog. This includes:
  - Save draft
  - Preview

Proposed actions:

- View possible improvements in [Keyboard shortcut documentation for editor in the Wagtail UI #12050](https://github.com/wagtail/wagtail/discussions/12050)
- Consider a "Command palette" concept for Wagtail

#### [A.3.2. (For the authoring tool user interface) Provide authors with enough time](https://www.w3.org/TR/ATAG20/#gl_a32)

See [Implementing A.3.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a32).

##### A.3.2.1 Auto-Save (Minimum)

> (Level A). See [Implementing A.3.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a321).

**Fail**. Wagtail doesn’t provide auto-save functionality. For Wagtail sites, the default session time limit is 2 weeks. See [Autosave #24](https://github.com/wagtail/roadmap/issues/24) on the Wagtail roadmap.

##### A.3.2.2 Timing Adjustable

> (Level A). See [Implementing A.3.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a322).

**Pass**. For Wagtail sites, the default session time limit is 2 weeks.

##### A.3.2.3 Static Input Components

> (Level A). See [Implementing A.3.2.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a323).

**Pass**. There are no moving input components in the CMS.

##### A.3.2.4 Content Edits Saved (Extended)

> (Level AAA). See [Implementing A.3.2.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a324).

**Fail**. Wagtail doesn’t provide auto-save functionality. See _A.3.2.1 Auto-Save (Minimum)_. We expect the same approach to be followed for both SCs.

#### [A.3.3. (For the authoring tool user interface) Help authors avoid flashing that could cause seizures](https://www.w3.org/TR/ATAG20/#gl_a33)

See [Implementing A.3.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a33).

##### A.3.3.1 Static View Option

> (Level A). See [Implementing A.3.3.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a331).

**Fail**. Animated GIFs auto-play when rendered, with no option to pause them.

Proposed actions:

- Research accessibility best practices on animated GIFs
- Design a new interface for how CMS users interact with animated GIFs
- Implement the new interface according to ATAG, WCAG standards, and accessibility best practices.

#### [A.3.4. (For the authoring tool user interface) Enhance navigation and editing via content structure](https://www.w3.org/TR/ATAG20/#gl_a34)

See [Implementing A.3.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a34).

##### A.3.4.1 Navigate By Structure

> (Level AA). See [Implementing A.3.4.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a341).

**Not applicable**. Markup elements aren’t exposed in the CMS.

##### A.3.4.2 Navigate by Programmatic Relationships

> (Level AAA). See [Implementing A.3.4.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a342).

**Pass**. The only editable programmatic relationships are headings and element nesting in rich text fields, which can be navigated via the keyboard.

#### [A.3.5. (For the authoring tool user interface) Provide text search of the content](https://www.w3.org/TR/ATAG20/#gl_a35)

See [Implementing A.3.5](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a35).

##### A.3.5.1 Text Search

> (Level AA). See [Implementing A.3.5.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a351).

**Fail**. Wagtail supports browsers’ built-in text search which meets all criteria, but only allows searching within the currently-active tab of the editing view. For example, for pages, content under the "Promote" tab will only be searchable when this tab is active.

Proposed actions:

- Implement [hidden="until-found"](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/hidden).
- Research how other content management systems cater for this requirement.
- Fix [Activating a tab based on the URL hash should also set focus to the corresponding tabpanel #8483](https://github.com/wagtail/wagtail/issues/8483)

#### [A.3.6. (For the authoring tool user interface) Manage preference settings](https://www.w3.org/TR/ATAG20/#gl_a36)

See [Implementing A.3.6](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a36).

##### A.3.6.1 Independence of Display

> (Level A). See [Implementing A.3.6.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a361).

**Pass**. All of Wagtail’s UI settings can be adjusted without modifying the content.

##### A.3.6.2 Save Settings

> (Level AA). See [Implementing A.3.6.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a362).

**Pass**. Specific settings are saved differently. The following settings are persistent for a given user profile, across all sessions of said user:

- Language
- Time zone
- Notification settings
- Admin interface theme
- UI density
- Contrast theme

The following settings are persistent for a given browser, across all sessions within said browser:

- Sidebar expanded/collapsed
- Rich text toolbar pinned/unpinned
- Minimap opened/closed
- Currently-open side panel

##### A.3.6.3 Apply Platform Settings

> (Level AA). See [Implementing A.3.6.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a363).

**Pass**. Wagtail’s language, time zone, and theme settings default to respecting platform settings until set to a specific value by the user.

#### [A.3.7. (For the authoring tool user interface) Ensure that previews are at least as accessible as in-market user agents](https://www.w3.org/TR/ATAG20/#gl_a37)

See [Implementing A.3.7](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a37).

##### A.3.7.1 Preview (Minimum)

> (Level A). See [Implementing A.3.7.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a371).

**Pass**. Wagtail’s live preview for pages and snippets and its draft renders within the user’s browser.

##### A.3.7.2 Preview (Enhanced)

> (Level AAA). See [Implementing A.3.7.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a372).

**Pass**. Wagtail’s live preview for pages and snippets can only display within the user’s browser, but all saved draft content can be previewed in any browser/device the user is logged in.

### [A.4. Editing-views are understandable](https://www.w3.org/TR/ATAG20/#principle_a4)

#### [A.4.1. (For the authoring tool user interface) Help authors avoid and correct mistakes](https://www.w3.org/TR/ATAG20/#gl_a41)

See [Implementing A.4.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a41).

##### A.4.1.1 Content Changes Reversible (Minimum)

> (Level A). See [Implementing A.4.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a411).

**Fail**. Though a large number of authoring actions are reversible, not all are. The following actions are reversible:

- Plain text and rich text content editing within specific fields, reversible until the user submits the form.
- Editing of page or snippets content supporting revisions, reversible for the whole page/snippet at any later point in the site’s history.

The following actions are not reversible but do require confirmation to proceed:

- Permanent deletion of any content which has its own dedicated creation/editing views.
- Unpublishing of pages and snippets.
- Deletion of comments within page content.

The following actions are not reversible and do not require confirmation to proceed:

- StreamField / InlinePanel block-based content editing, reversible only as part of support for content revisions.
  - Consider implementing an in-browser undo-redo stack for those interactions.
- Image / Document / Page / Task / Snippet / Embed chooser fields, reversible only as part of support for content revisions.
  - Consider implementing an in-browser undo-redo stack for those interactions.
- Authoring actions on content that does not support revisions such as images, documents, etc.
  - Implement either a confirmation step for those actions, or revisions/versioning support.
- Reordering of pages within list views
  - Implement either a confirmation step for those actions, or revisions/versioning support.

##### A.4.1.2 Settings Change Confirmation

> (Level A). See [Implementing A.4.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a412).

**Pass**. All of Wagtail’s UI settings saved at the browser level can be reversed by the user directly within the UI. All of the settings saved at the user profile level can be set to an "unset" default value.

Recommendation for Wagtail:

- Reset all browser-level settings when users intentionally log out (not on session timeouts)
- Add a way to "Reset preferences" – reverse all browser-level and user profile settings to their default value within the user’s profile form.

##### A.4.1.3 Content Changes Reversible (Enhanced)

> (Level AAA). See [Implementing A.4.1.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a413).

**Pass**. Reversible plain text and rich text content changes can be reversed sequentially while the user remains on the page. Content supporting revisions can be restored at any point in the content’s history.

#### [A.4.2. (For the authoring tool user interface) Document the user interface, including all accessibility features](https://www.w3.org/TR/ATAG20/#gl_a42)

See [Implementing A.4.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_a42).

##### A.4.2.1 Describe Accessibility Features

> (Level A). See [Implementing A.4.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a421).

**Pass**. The following functionality would be used to meet Part A and needs to be described either in the documentation or in the user interface:

The following functionality is described in the user interface:

- Restore revisions
- Command palette trigger
- Keyboard shortcuts
  - Page-level keyboard shortcuts
  - Rich text formatting
  - Markdown commands for rich text
  - Comment in rich text

The following functionality is described in the documentation:

- Images’ alt text management
- Skip link
- Collapsible sections
- Link to specific collapsible sections
- Collapse/expand all
- Minimap
- Session time limit
- Editing of headings and elements nesting in rich text fields
- Text search
- Browser-level UI settings
  - Sidebar expanded/collapsed
  - Rich text toolbar pinned/unpinned
  - Minimap opened/closed
  - Currently-open side panel
- Profile-level UI settings
  - Language
  - Time zone
  - Notification settings
  - Admin interface theme
  - Density
  - Contrast theme
- Live preview
- Command palette available commands
- Keyboard shortcuts
  - Page-level keyboard shortcuts
  - Rich text formatting
  - Markdown commands for rich text
  - Comment in rich text

The following functionality is provided by the underlying platform:

- Embeds titles as alt text for embedded content

##### A.4.2.2 Document All Features

> (Level AA). See [Implementing A.4.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_a422).

**Fail**. Here is a high-level record of whether given functionality is documented. As a summary:

- For 19 high-level functional areas, 9 are partially documented and 4 are fully documented.
- For 145 specific features, 52 are documented.

This record does not cover functionality provided by the underlying platform (for example; automated embed creation) or unused by authors.

| Functionality                                     | Documented? |
| ------------------------------------------------- | ----------- |
| **Global**                                        | Partial     |
| Skip link                                         | Yes         |
| No-JS warning message                             | No          |
| Sidebar                                           | Yes         |
| Header                                            | No          |
| Messages                                          | No          |
| Relative date                                     | No          |
| Status tag                                        | Yes         |
| Table listing                                     | No          |
| Pagination                                        | No          |
| Dashboard                                         | Yes         |
| Unauthorized access (403)                         | No          |
| Page not found (404)                              | No          |
| Accessibility checker                             | No          |
| Wagtail userbar                                   | No          |
| **Pages**                                         | Partial     |
| Pages explorer                                    | Yes         |
| Add child page                                    | Yes         |
| Page type usage                                   | Yes         |
| Privacy                                           | Yes         |
| Move pages                                        | No          |
| Copy                                              | Yes         |
| Delete                                            | No          |
| Publish                                           | No          |
| Unpublish                                         | No          |
| View all revisions                                | Yes         |
| Compare revisions                                 | No          |
| Review revision                                   | Yes         |
| Create                                            | Yes         |
| Edit                                              | Yes         |
| Edit - Promote tab                                | No          |
| Preview                                           | No          |
| Scheduled publishing                              | Yes         |
| Explorer - Bulk actions                           | Yes         |
| Explorer - Bulk move                              | No          |
| Explorer - Bulk delete                            | No          |
| Explorer - Bulk publish                           | No          |
| Explorer - Bulk unpublish                         | No          |
| Explorer - View child pages                       | Yes         |
| Explorer - View child pages - Reorder child pages | No          |
| Explorer - View child pages - Sort by column      | No          |
| Explorer - Root level                             | No          |
| Search                                            | Yes         |
| Search - Filtered                                 | No          |
| **Images**                                        | Partial     |
| View all                                          | No          |
| Search                                            | No          |
| Search - Filtered                                 | No          |
| Edit                                              | Yes         |
| Add an image                                      | No          |
| Add multiple images                               | No          |
| Delete                                            | No          |
| Image URL generator                               | No          |
| Bulk actions                                      | Yes         |
| Bulk add tags                                     | No          |
| Bulk add to collection                            | No          |
| Bulk add delete                                   | No          |
| **Documents**                                     | Partial     |
| View all                                          | No          |
| Search                                            | No          |
| Search - Filtered                                 | No          |
| Edit                                              | Yes         |
| Add a document                                    | No          |
| Add multiple documents                            | No          |
| Delete                                            | No          |
| Bulk actions                                      | Yes         |
| Bulk add tags                                     | No          |
| Bulk add to collection                            | No          |
| Bulk add delete                                   | No          |
| **Snippets**                                      | Partial     |
| View all types                                    | Yes         |
| View all                                          | No          |
| Search snippets                                   | No          |
| Edit                                              | Yes         |
| Add                                               | No          |
| Delete                                            | No          |
| Bulk delete                                       | No          |
| Publish                                           | No          |
| Unpublish                                         | No          |
| Preview                                           | No          |
| Scheduled publishing                              | No          |
| View all revisions                                | No          |
| Compare revisions                                 | No          |
| Review revision                                   | No          |
| **Forms**                                         | No          |
| View all                                          | No          |
| View submissions                                  | No          |
| **Reports**                                       | Yes         |
| Locked Pages                                      | Yes         |
| Workflows                                         | Yes         |
| Workflow tasks                                    | Yes         |
| Site history                                      | Yes         |
| Aging pages                                       | Yes         |
| Page types usage                                  | Yes         |
| **Workflows**                                     | Partial     |
| View all                                          | Yes         |
| Add                                               | No          |
| Usage                                             | No          |
| Edit                                              | Yes         |
| **Workflow tasks**                                | Partial     |
| View all                                          | Yes         |
| Add                                               | Yes         |
| Edit                                              | No          |
| Disable                                           | No          |
| **Users**                                         | Partial     |
| View all                                          | Yes         |
| Search                                            | No          |
| Edit                                              | No          |
| Edit - Edit roles                                 | No          |
| Add                                               | No          |
| Add - Add roles                                   | No          |
| Delete                                            | No          |
| Bulk actions                                      | Yes         |
| Bulk delete                                       | No          |
| Bulk set active state                             | No          |
| Bulk assign role                                  | No          |
| **Groups**                                        | No          |
| View all                                          | No          |
| Search                                            | No          |
| Add a group                                       | No          |
| Edit                                              | No          |
| Delete                                            | No          |
| Group users                                       | No          |
| **Sites**                                         | No          |
| View all                                          | No          |
| Edit                                              | No          |
| Add a site                                        | No          |
| Delete                                            | No          |
| **Locales**                                       | No          |
| View all                                          | No          |
| Edit                                              | No          |
| Add a locale                                      | No          |
| Delete                                            | No          |
| **Collections**                                   | Partial     |
| View all                                          | Yes         |
| Edit                                              | No          |
| Add a collection                                  | Yes         |
| Privacy                                           | Yes         |
| **Redirects**                                     | Yes         |
| View all                                          | Yes         |
| Search                                            | Yes         |
| Import redirects                                  | Yes         |
| Add                                               | Yes         |
| Export Redirects                                  | Yes         |
| Edit                                              | Yes         |
| Delete                                            | Yes         |
| **Promoted search**                               | Yes         |
| View all                                          | Yes         |
| Search                                            | Yes         |
| Edit                                              | Yes         |
| Add results                                       | Yes         |
| Delete                                            | Yes         |
| **User account**                                  | Yes         |
| Account profile                                   | Yes         |
| Account notifications                             | Yes         |
| **Styleguide**                                    | No          |
| Styleguide                                        | No          |
| **Auth**                                          | No          |
| Login                                             | No          |
| Password reset                                    | No          |
| Password reset done                               | No          |

## [B. Support the production of accessible content](https://www.w3.org/TR/ATAG20/#part_b)

### [B.1. Fully automatic processes produce accessible content](https://www.w3.org/TR/ATAG20/#principle_b1)

#### [B.1.1. Ensure that automatically-specified content is accessible](https://www.w3.org/TR/ATAG20/#gl_b11)

See [Implementing B.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b11).

##### B.1.1.1 Content Auto-Generation After Authoring Sessions (WCAG)

> (Level A / AA / AAA). See [Implementing B.1.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b111).

**Pass**. Evaluated as: **Level AAA**. Wagtail doesn’t automatically generate content after authoring sessions. Processes that operate after authoring sessions and could alter the content are [scheduled publishing](https://docs.wagtail.org/en/stable/reference/pages/theory.html#id2) and [search index](https://docs.wagtail.org/en/stable/topics/search/indexing.html#wagtailsearch-indexing-update) updates, but in both cases any automatically-generated content would already be present during the session.

##### B.1.1.2 Content Auto-Generation During Authoring Sessions (WCAG)

> (Level A / AA / AAA). See [Implementing B.1.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b112).

**Pass**. Evaluated as: **Level AA**. Wagtail automatically generates content in a few scenarios. In the following scenarios, markup is accessible without further work:

- Pass: Links markup for links to pages, documents, external URLs, email addresses, phone numbers, and internal anchors within rich text fields.
- Pass: Embeds for external resources within rich text fields.
- Pass: Embeds for external resources in StreamField.
- Pass: Image markup for images in rich text fields. Images are rendered with alt text from an editable field, and a checkbox to mark the image as decorative.
- Pass: Image markup for images in StreamField. The Image block is rendered with alt text from an editable field, and a checkbox to mark the image as decorative.
- Pass: Table markup from TableBlock.
- Pass: Table markup for TypedTableBlock.

Proposed actions:

- Consider enforcing all editor accessibility-checks have to pass before publishing content.

#### [B.1.2. Ensure that accessibility information is preserved](https://www.w3.org/TR/ATAG20/#gl_b12)

See [Implementing B.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b12).

##### B.1.2.1 Restructuring and Recoding Transformations (WCAG)

> (Level A / AA / AAA). See [Implementing B.1.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b121).

**Not applicable**. Evaluated as: **Level AA**. The only transformation present in Wagtail is processing of clipboard paste information in rich text fields to sanitize the content, which preserves accessibility semantics for preserved content, but isn’t considered a content transformation per ATAG.

If it was considered a content transformation – rich paste processing preserves all formatting supported in rich text fields. Heading levels, bullet lists, and image alt text are preserved in particular.

##### B.1.2.2 Copy-Paste Inside Authoring Tool (WCAG)

> (Level A / AA / AAA). See [Implementing B.1.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b122).

**Pass**. Evaluated as: **Level AA**. Wagtail supports copy-paste of rich text content, which is fully preserved when copy-pasting between fields configured to support the same formatting. Fields configured differently will accordingly have their formatting stripped as needed.

##### B.1.2.3 Optimizations Preserve Accessibility

> (Level A). See [Implementing B.1.2.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b123).

**Not applicable**. Wagtail doesn’t perform any optimizations that would affect accessibility.

##### B.1.2.4 Text Alternatives for Non-Text Content are Preserved

> (Level A). See [Implementing B.1.2.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b124).

**Not applicable**. See _B.1.2.1 Restructuring and Recoding Transformations (WCAG)_.

### [B.2. Authors are supported in producing accessible content](https://www.w3.org/TR/ATAG20/#principle_b2)

#### [B.2.1. Ensure that accessible content production is possible](https://www.w3.org/TR/ATAG20/#gl_b21)

See [Implementing B.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b21).

##### B.2.1.1 Accessible Content Possible (WCAG)

> (Level A / AA / AAA). See [Implementing B.2.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b211).

**Fail**. Evaluated as: **Level AA**. Wagtail places extensive restrictions on the production of web content, which all nonetheless allow for the production of accessible content, with the exception of:

- Missing support for setting `lang` attributes within rich text. This could be worked around by using other types of content modeling for multilingual content, which is possible but unlikely.

Proposed actions:

- Implement [Feature request: Support for declaring language on elements in rich text. #4694](https://github.com/wagtail/wagtail/issues/4694)

#### [B.2.2. Guide authors to produce accessible content](https://www.w3.org/TR/ATAG20/#gl_b22)

See [Implementing B.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b22).

##### B.2.2.1 Accessible Option Prominence (WCAG)

> (Level A / AA / AAA). See [Implementing B.2.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b221).

**Pass**. Evaluated as: **Level AA**. Where text styling options are available, they are presented alongside semantic formatting options such as headings. This is for example the case in rich text formatting options.

For StreamField block formats, the order is up to each site implementer to decide on. There are no built-in formats that are automatically included.

##### B.2.2.2 Setting Accessibility Properties (WCAG)

> (Level A / AA / AAA). See [Implementing B.2.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b222).

**Not applicable**. Evaluated as: **Level AAA**. Wagtail doesn’t support setting web content attribute values. This has been discussed extensively for links, as well as an option to set `aria-label`, but hasn’t been implemented yet.

See:

- [Feature request: Support for declaring language on elements in rich text. #4694](https://github.com/wagtail/wagtail/issues/4694)
- [Feature: ability to set links as "nofollow" rel via the WYSIWG editor's link chooser #4474](https://github.com/wagtail/wagtail/issues/4474)

#### [B.2.3. Assist authors with managing alternative content for non-text content](https://www.w3.org/TR/ATAG20/#gl_b23)

See [Implementing B.2.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b23).

##### B.2.3.1 Alternative Content is Editable (WCAG)

> (Level A / AA / AAA). See [Implementing B.2.3.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b231).

**Pass**. Evaluated as: **Level AA**. All areas where images can be used support providing alt text. This includes:

- Global alt text via the built-in "Description" field.
- Contextual alt text within rich text via the "Alt text" field and "Decorative" checkbox.
- Contextual alt text in blocks via the "Alt text" field and "Decorative" checkbox.

##### B.2.3.2 Automating Repair of Text Alternatives

> (Level A). See [Implementing B.2.3.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b232).

**Pass**. Wagtail doesn’t attempt to repair text alternatives. It does use the image’s file name as the default value for the image’s title field when creating a new image, which is used for alt text, but this is part of the upload/editing process (In-Session Repairs) and not an automated process. Said file name is filtered by default to remove the file extension.

##### B.2.3.3 Save for Reuse

> (Level AAA). See [Implementing B.2.3.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b233).

**Fail**. By default, Wagtail saves each image’s text alternative and reuses it everywhere the image is reused ("Save and Suggest"). It is possible to replace this text alternative with a new one, but it isn’t possible to delete it.

Suggested action: incorporate this requirement into the contextual alt text support.

#### [B.2.4. Assist authors with accessible templates](https://www.w3.org/TR/ATAG20/#gl_b24)

See [Implementing B.2.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b24).

##### B.2.4.1 Accessible Template Options (WCAG)

> (Level A / AA / AAA). See [Implementing B.2.4.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b241).

**Fail**. Evaluated as: **Level AA**. With rich text formatting and StreamField blocks, Wagtail provides templates for basic text content as well as complex formatting like tables. Wagtail also provides templates for form fields within its forms module. Specific templates have accessibility issues.

###### Rich text formats

Note: Blockquote formatting does not support the optional blockquote `cite` attribute or `<cite>` element.

###### Field block types

| Template             | Accessibility issues with output |
| -------------------- | -------------------------------- |
| CharBlock            | None                             |
| TextBlock            | None                             |
| EmailBlock           | None                             |
| IntegerBlock         | None                             |
| FloatBlock           | None                             |
| DecimalBlock         | None                             |
| RegexBlock           | None                             |
| URLBlock             | None                             |
| BooleanBlock         | None                             |
| DateBlock            | None                             |
| TimeBlock            | None                             |
| DateTimeBlock        | None                             |
| RichTextBlock        | None                             |
| RawHTMLBlock         | None                             |
| BlockQuoteBlock      | None                             |
| ChoiceBlock          | None                             |
| MultipleChoiceBlock  | None                             |
| PageChooserBlock     | None                             |
| DocumentChooserBlock | None                             |
| ImageBlock           | None                             |
| ImageChooserBlock    | None                             |
| SnippetChooserBlock  | None                             |
| EmbedBlock           | None                             |
| TableBlock           | None                             |
| TypedTableBlock      | None                             |

Note: BlockQuoteBlock does not support the optional blockquote `cite` attribute or `<cite>` element.

###### Structural block types

Structural block types do not render any content and as such have no accessibility issues.

| Template    | Accessibility issues |
| ----------- | -------------------- |
| StaticBlock | None                 |
| StructBlock | None                 |
| ListBlock   | None                 |
| StreamBlock | None                 |

###### Form builder field types

All form builder fields suffer from the same issue: Error messages aren’t programmatically associated with the field.

| Template         | Accessibility issues            |
| ---------------- | ------------------------------- |
| Single line text | Errors association with fields. |
| Multi-line text  | Errors association with fields. |
| Email            | Errors association with fields. |
| Number           | Errors association with fields. |
| URL              | Errors association with fields. |
| Checkbox         | Errors association with fields. |
| Checkboxes       | Errors association with fields. |
| Drop down        | Errors association with fields. |
| Multiple select  | Errors association with fields. |
| Radio buttons    | Errors association with fields. |
| Date             | Errors association with fields. |
| Date/time        | Errors association with fields. |
| Hidden field     | None                            |

This may be addressed in the future, via improvements in Django. See:

- [Django ticket #32819 - Fields’ help text and errors should be associated with input](https://code.djangoproject.com/ticket/32819)
- [Fixed #32819 -- Established relationship between form fields and their errors. #17520](https://github.com/django/django/pull/17520)

Proposed actions:

- Support improvements in Django

##### B.2.4.2 Identify Template Accessibility

> (Level AA). See [Implementing B.2.4.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b242).

**Pass**. Wagtail’s template selection mechanism is the block chooser (or field chooser for form builder fields). In both cases, there would be no options defined unless configured by site implementers, which can customize block icons or labels / names to indicate accessible block types.

##### B.2.4.3 Author-Created Templates

> (Level AA). See [Implementing B.2.4.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b243).

**Not applicable**. Wagtail doesn’t allow authors to create custom block templates.

##### B.2.4.4 Accessible Template Options (Enhanced)

> (Level AAA). See [Implementing B.2.4.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b244).

**Fail**. Some of Wagtail’s template options aren’t accessible. See _B.2.4.1 Accessible Template Options_.

#### [B.2.5. Assist authors with accessible pre-authored content](https://www.w3.org/TR/ATAG20/#gl_b25)

See [Implementing B.2.5](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b25).

##### B.2.5.1 Accessible Pre-Authored Content Options

> (Level AA). See [Implementing B.2.5.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b251).

**Not applicable**. Wagtail doesn’t provide pre-authored content.

##### B.2.5.2 Identify Pre-Authored Content Accessibility

> (Level AA). See [Implementing B.2.5.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b252).

**Not applicable**. Wagtail doesn’t provide pre-authored content.

### [B.3. Authors are supported in improving the accessibility of existing content](https://www.w3.org/TR/ATAG20/#principle_b3)

#### [B.3.1. Assist authors in checking for accessibility problems](https://www.w3.org/TR/ATAG20/#gl_b31)

See [Implementing B.3.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b31).

##### B.3.1.1 Checking Assistance (WCAG)

> (Level A / AA / AAA). See [Implementing B.3.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b311).

**Pass**. Evaluated as: **Level A**. There are a number of formatting / content entry options in the CMS that can lead to accessibility issues. The built-in accessibility checker provides automated tests for a number of possible issues, but not all. Available checks are:

- `button-name`: `<button>` elements must always have a text label.
- `empty-heading`: This rule checks for headings with no text content. Empty headings are confusing to screen readers users and should be avoided.
- `empty-table-header`: Table header text should not be empty
- `frame-title`: `<iframe>` elements must always have a text label.
- `heading-order`: This rule checks for incorrect heading order. Headings should be ordered in a logical and consistent manner, with the main heading (h1) followed by subheadings (h2, h3, etc.).
- `input-button-name`: `<input>` button elements must always have a text label.
- `link-name`: `<a>` link elements must always have a text label.
- `p-as-heading`: This rule checks for paragraphs that are styled as headings. Paragraphs should not be styled as headings, as they don’t help users who rely on headings to navigate content.
- `alt-text-quality`: A custom rule ensures that image alt texts don’t contain anti-patterns like file extensions and underscores.

There are a number of success criteria that do not have automated checks nor instructions for manual checking:

- 2.4.4 Link Purpose (In Context) (AA) – instructions and potentially limited automated checks on correct link text (avoid "click here")
- 1.4.5 Images of Text (AA) – instructions on avoiding images of text except for scenarios where there is no alternative
- 2.4.9 Link Purpose (Link Only) (AAA) – see above
- 1.4.9 Images of Text (No Exception) (AAA) – see above
- 3.1.5 Reading Level (AAA) – instructions for manual checking or automated checks
- 2.4.10 Section Headings (AAA) – instructions for manual checking or automated checks
- 3.3.5 Help (AAA) – instructions for manual checking or automated checks for the form builder

Proposed actions:

- Implement more automated checks for WCAG SCs in question. [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063)
- Implement a manual check process for WCAG SCs in question. [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063)
- Roadmap: [Readability checks #50](https://github.com/wagtail/roadmap/issues/50)
- Content quality API integrations for text quality and image computer vision
- Improve options for organizations to embed their own instructions within the CMS, via HelpBlock and help text.

##### B.3.1.2 Help Authors Decide

> (Level A). See [Implementing B.3.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b312).

**Not applicable**. Currently all of Wagtail’s automated checks are pass/fail with no author input required. Discussion on further checks: [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063)

##### B.3.1.3 Help Authors Locate

> (Level A). See [Implementing B.3.1.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b313).

**Not applicable**. Currently all of Wagtail’s automated checks are pass/fail with no author input required. Those checks do indicate which elements they are flagging. Proposed improvements:

- [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063)
- [Accessibility checker in page editor #10136](https://github.com/wagtail/wagtail/issues/10136)
- Correct identification of Wagtail content so errors are only reported on CMS-managed content

##### B.3.1.4 Status Report

> (Level AA). See [Implementing B.3.1.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b314).

**Pass**. Wagtail’s accessibility checker reports on the number of detected issues, and upon interaction lists all rules that detected problems and where on the page.

Possible improvements:

- [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063) – making this data available in more readily accessible reports
- Consider making the information easily copy-pasteable

##### B.3.1.5 Programmatic Association of Results

> (Level AA). See [Implementing B.3.1.5](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b315).

**Fail**. Currently the association isn’t programmatic, due to expected compatibility issues.

Possible resolution: Correct identification of Wagtail content so errors are only reported on CMS-managed content.

#### [B.3.2. Assist authors in repairing accessibility problems](https://www.w3.org/TR/ATAG20/#gl_b32)

See [Implementing B.2.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b32).

##### B.3.2.1 Repair Assistance (WCAG)

> (Level A / AA / AAA). See [Implementing B.3.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b321).

**Fail**. Evaluated as: **Level A**. Currently Wagtail’s 9 rules report the presence of a problem but do not suggest specific solutions.

Proposed improvements:

- [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063)
- Correct identification of Wagtail content so errors are only reported on CMS-managed content

### [B.4. Authoring tools promote and integrate their accessibility features](https://www.w3.org/TR/ATAG20/#principle_b4)

#### [B.4.1. Ensure the availability of features that support the production of accessible content](https://www.w3.org/TR/ATAG20/#gl_b41)

See [Implementing B.4.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b41).

##### B.4.1.1 Features Active by Default

> (Level A). See [Implementing B.4.1.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b411).

**Pass**. Wagtail’s accessibility checker is on by default and cannot be turned off unless site implementers make CMS customizations in code.

##### B.4.1.2 Option to Reactivate Features

> (Level A). See [Implementing B.4.1.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b412).

**Pass**. See _B.4.1.1 Features Active by Default_.

##### B.4.1.3 Feature Deactivation Warning

> (Level AA). See [Implementing B.4.1.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b413).

**Pass**. See _B.4.1.1 Features Active by Default_.

##### B.4.1.4 Feature Prominence

> (Level AA). See [Implementing B.4.1.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b414).

**Pass**. Wagtail currently has no features relating to invalid markup, syntax errors, spelling errors, grammar errors. Possible future changes:

- Implement more automated checks for WCAG SCs in question. [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063)
- Implement a manual check process for WCAG SCs in question. [Content quality checkers #11063](https://github.com/wagtail/wagtail/discussions/11063)
- Roadmap: [Readability checks #50](https://github.com/wagtail/roadmap/issues/50)

#### [B.4.2. Ensure that documentation promotes the production of accessible content](https://www.w3.org/TR/ATAG20/#gl_b42)

See [Implementing B.4.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#gl_b42).

##### B.4.2.1 Model Practice (WCAG)

> (Level A / AA / AAA). See [Implementing B.4.2.1](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b421).

**Fail**. Evaluated as: **Level AA**. Wagtail’s documentation for content authors does not demonstrate accessible authoring practices. The documentation for developers does: [Accessibility considerations](https://docs.wagtail.org/en/stable/advanced_topics/accessibility_considerations.html).

Proposed actions:

- [Accessibility features documentation #69](https://github.com/wagtail/roadmap/issues/69)
- Add accessibility criteria to contributor documentation for documentation writing
- Review developer documentation for accessibility improvements in examples

##### B.4.2.2 Feature Instructions

> (Level A). See [Implementing B.4.2.2](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b422).

**Pass**. Wagtail’s accessibility checker, heading levels, and alt text implementation are documented in the guide for content authors. The [documentation for developers](https://docs.wagtail.org/en/stable/advanced_topics/accessibility_considerations.html) also cover this.

Proposed actions:

- Create a single page summarizing all accessibility-related information.

##### B.4.2.3 Tutorial

> (Level AAA). See [Implementing B.4.2.3](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b423).

**Fail**. Wagtail’s accessibility features doesn’t have a tutorial.

Proposed actions:

- Create a written tutorial in addition to how-to material.
- Create a video tutorial in addition to the user guide website.

##### B.4.2.4 Instruction Index

> (Level AAA). See [Implementing B.4.2.4](http://www.w3.org/TR/2015/NOTE-IMPLEMENTING-ATAG20-20150924/#sc_b424).

**Fail**. Wagtail’s documentation for content author does not have such an index. The documentation for developers does.

Proposed actions:

- Add an index of accessibility features to the user guide.
- [Outreachy: Accessibility features documentation](https://wagtail.org/blog/our-outreachy-projects-in-2023/)
