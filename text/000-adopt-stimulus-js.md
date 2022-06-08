# RFC 07?: Adopt Stimulus (lightweight JS framework)

- RFC: 07?
- Author: LB (Ben) Johnston
- Created: 2022-06-05
- Last Modified: 2022-06-05

- [Abstract](#abstract)
  - [Primary Goals](#primary-goals)
  - [Secondary Goals](#secondary-goals)
  - [History & Initial Research](#history--initial-research)
- [Specification](#specification)
  - [Initial Implementation Overview](#initial-implementation-overview)
  - [Implementation roadmap](#implementation-roadmap)
  - [Proposed Documentation](#proposed-documentation)
- [Open Questions](#open-questions)
  - [Storybook compatibility](#storybook-compatibility)
  - [API for a controller definition as an object](#api-for-a-controller-definition-as-an-object)
  - [Dispatched event names](#dispatched-event-names)
  - [Prefix on controllers](#prefix-on-controllers)
  - [TypeScript verbosity](#typescript-verbosity)
  - [Handling animations](#handling-animations)
- [Answered questions](#answered-questions)
  - [Why use a Django & HTML first approach](#why-use-a-django--html-first-approach)
  - [Why not more React](#why-not-more-react)
  - [Why not use Vue, Svelte, Angular, or Solid etc](#why-not-use-vue-svelte-angular-or-solid-etc)
  - [Why not Alpine.js](#why-not-alpinejs)
  - [Why not HTMX or Turbolinks](#why-not-htmx-or-turbolinks)
  - [Why Stimulus and not something else](#why-stimulus-and-not-something-else)
- [Additional Information](#additional-information)
  - [Links](#links)
  - [CSP / Inline Scripts](#csp--inline-scripts)
  - [Stimulus in the wild](#stimulus-in-the-wild)
  - [Future possibilities](#future-possibilities)
- [Why not Stimulus at all](#why-not-stimulus-at-all)

## Abstract

This RFC proposes that Wagtail adopt [Stimulus](https://stimulus.hotwired.dev/) for its lightweight JavaScript framework.

The aim of this lightweight framework is to provide a consistent, predictable and extensible way to build interactive JavaScript UI elements.

So that that Wagtail has a solid alternative to jQuery widgets, inline script tag usage, window globals and initialising JavaScript to the DOM when added dynamically. While further enhancing the ability for Wagtail to move towards a platform that can be further extended and built upon by other developers.

Stimulus is an evolution on what we are already doing in almost all JS interactivity written in the last few years. Setting up data attributes to attach JS interactivity. However, this is a more consistent approach without needing to init things and providing a simple way to do small scale reactivity (e.g. local state driven changes).

> 🚧 A prototype implementation of this proposal is available for review and testing on https://github.com/lb-/wagtail-rfcs/tree/rfc/stimulus or https://github.com/lb-/wagtail/pull/5.

### Primary Goals

- **Django/Python first** - Where possible, Django (using Python and then HTML templates) first and JavaScript second. It should be easy to to basic things (use existing widgets, modify initial data, change some basic behaviour) by using existing Django or Wagtail approaches (e.g. `attrs` on widgets or a similar approach on Wagtail Template components). More complex changes should be able to be done in templates alone. Finally building new JS behaviour or re-writing core logic should be able to be done in JavaScript where HTML data attributes are not suitable.
- **Not replacing React** - There is no need to replace React, it serves a critical purpose for complex, self-contained and heavy state driven UI elements. The framework must be able to be used with React and React must be able to be used by it.
- **Vanilla JS** - The framework should use and promote browser APIs, not getting in the way of native functions such as event handling and element selectors but add convenience where suitable that can easily be bypassed. Note: Vanilla JS is an overloaded term - refer to this site as a guide http://vanilla-js.com/ .
- **Accessibility** - it should be easy to use existing accessibility approaches or add them to components.
- **Secure** - The solution must support our move towards zero globals and zero inline scripts (CSP compliance) to build the UI elements and ensure they get instantiated wherever they appear (inline panel, modals, StreamField). See also [RFC 33](https://github.com/wagtail/rfcs/pull/33) and links below relating to CSP compliance.
- **Extensible** - Let Wagtail developers enhance/extend/reuse these elements without needing to have a JavaScript build tool, along with using this framework for their own custom UI elements.
- **Composable** - Within or outside of Wagtail it must be possible to compose behaviour using a mix of Wagtail provided and custom JS behaviours. Unlike any similar library Stimulus allows [multiple controllers](https://stimulus.hotwired.dev/reference/controllers#multiple-controllers) to be added to the same element.
- **Powerful for Frontend devs** - Ensure the core team and code contributors get to leverage the power of a build tool system, including TypeScript, unit testing and our pattern library (Storybook).
- **Namespaced** - Wagtail implementations should be namespaced or somehow isolated from other potential duplicate usage of the same libraries.
- **i18n/l10n** - It should be easy to provide translated / localised content to the elements.

### Secondary Goals

- **External reuse** - Is there a pathway toward the components that are built to be used in isolation of styles and in isolation of even Wagtail itself (e.g. as a importable module)?
- **ES6 Modules Compatible** - There should be an easy migration path to adoption of ES6 modules this will be out of scope for this RFC but it is good to call out.

### History & Initial Research

Throughout 2022, Wagtail has been releasing a major UI overhaul of the user interface and features in Wagtail’s page editor.

As new UI components have been built, there is an understanding that jQuery is not an option, but React is not suitable for content that needs to be driven mostly by content in Django templates. This has lead to extremely ad-hoc approaches for UI elements, there is almost no consistency between how the new breadcrumbs, tabs and tooltips have been built. This makes it extremely hard for new developers to get started and core developers to maintain frontend code.

React is an incredible library but better serves situations where the full control of the DOM branch is given to React, DOM rendering is 'given over' to JavaScript and Django templates cannot be leveraged (easily) to change what is inside those DOM elements.

jQuery has served a great purpose, providing easy DOM traversal and manipulation, however it is becoming a bottleneck for new features and modern DOM APIs are sufficient and jQuery does not solve the problem of making it easy to write consistent code (component state). Additional to this we end up having to a lot of 'initialising' of elements whenever added to the DOM by other means (other JS or maybe async HTML).

This lead us (UI team) to do initial research at the start of 2022 into what frameworks exist (based on suggestions from the Wagtail community). See [Lightweight JavaScript Framework research](https://github.com/lb-/bakerydemo/blob/ui-experiments/lightweight-frontend-framework-investigation-2022.md).

This also lead to various rounds of feedback about what approaches could work with a leaning towards Stimulus. See the following links:

- [main discussion sub-tread on a lightweight framework](https://github.com/wagtail/wagtail/discussions/7689#discussioncomment-2037913)
- [page editor initial discussion on a lightweight framework](https://github.com/wagtail/wagtail/discussions/7739#discussioncomment-2022909)
- [page editor discussion resulting in suggestions for frameworks](https://github.com/wagtail/wagtail/discussions/7739#discussioncomment-1926410)
- [feedback from Matt](https://github.com/wagtail/wagtail/discussions/7689#discussioncomment-2775453)

## Specification

### Initial Implementation Overview

- Install Stimulus `npm install @hotwired/stimulus --save`
- Due to Stimulus being an ES6 module and Wagtail's codebase transpiling to ES5, TypeScript/Webpack and Jest will need to be configured to ensure that this library within node modules is to be transpiled also.
- Set up a file `client/src/includes/stimulus.ts` which will contain the Stimulus application set up and register of initial controllers, it should export a named export `initStimulus` which will allow for control over when the Application is set up.
- The dispatched events and event listeners should be set up as described in the documentation snippets below.
- This file should have an automatic way to pull in any Controllers in the folder `client/src/controllers/` such that a controller file named `TagFieldController.ts` with a named export `{ TagFieldController }` will automatically be registered under the prefixed namespace `w-tag-field` for its identifier. The inspiration for this approach can be found here https://github.com/hotwired/stimulus-webpack-helpers
- There should be a base `AbstractController` that uses [TypeScript's Abstract class](https://www.typescriptlang.org/docs/handbook/2/classes.html#abstract-classes-and-members) system to provide any base Wagtail methods, core overrides to Stimulus' classes and ensure that derived classes adhere to any expected methods.
- If developers want to use this framework for their own custom JS interactive code within their Wagtail installation. We should provide a simple way to register custom controllers, without any prior knowledge of how JavaScript classes work or needing a build tool. The inspiration for this is [Alpine.js' bind approach](https://alpinejs.dev/globals/alpine-bind) and [StackOverflow's Stimulus JS library controller creation](https://stackoverflow.design/product/guidelines/javascript/#creating-your-own-stimulus-controllers). Note: Developers can still use any JavaScript approach they want, including the existing React globals if they choose.
- A functional version of all of the above can be found here - https://github.com/lb-/wagtail/tree/rfcs/stimulus-ui or https://github.com/lb-/wagtail/pull/5 (to see diffs)

### Implementation roadmap

1. [ ] Get initial base into core, as per above initial implementation
2. [ ] Make it possible to provide `attrs` to the `FieldPanel/Panel` and in general Template Components
3. [ ] **Adopt for new code** - Migrate Tabs, Breadcrumbs, new Modal + Modal trigger
4. [ ] **Replace core inline Scripts** - Anything that can be removed from `<script />` tags - e.g. `data-sprite`/`loadIconSprite` at the global level (`wagtail/admin/templates/wagtailadmin/skeleton.html` & `wagtail/admin/templates/wagtailadmin/admin_base.html`)
5. [ ] **Replace headerSearch** - Adopt for header search component (most common script tag)
6. [ ] **Adopt for tooltips** - `data-wagtail-tooltip` usage (next most common script tag), may be done as part of page editor work
7. [ ] **Common fields** - Text area auto-resize, Tag field, etc
8. [ ] **Other common components** - Collapse, Dropdowns, LockUnlockAction, dirty form check, etc
9. [ ] **static_src** - Review all non-modal workflow static_src items (modeladmin/prepopulate, images focus, sitesettings siteswitcher, image-url-generator)
10. [ ] **Larger items** - one of modal workflow, inline panel, bulk actions, depending on state of project

### Proposed Documentation

#### Documentation for Developers

> Proposed addition to the [Customising admin templates](https://docs.wagtail.org/en/stable/advanced_topics/customisation/admin_templates.html) page which currently houses similar frontend docs for Wagtail developers. Alternatively a new sub-page under [Extending Wagtail](https://docs.wagtail.org/en/stable/extending/index.html) may be more suitable.

##### Introduction

- Wagtail uses [Stimulus](https://stimulus.hotwired.dev/) as a way to attach interactive behaviour to DOM elements throughout Wagtail.
- You do not need to use this library to add behaviour to elements, vanilla (plain) JS will work fine. However, it is recommended to not use jQuery as this will be removed in a future version.

##### Adding a word count controller (without a build system)

```javascript
// myapp/static/js/word-count-controller.js

const wordCountController = {
  STATIC: {
    values: { max: { default: 10, type: Number } },
  },
  connect: function () {
    this.setupOutput();
    this.updateCount();
  },
  setupOutput: function () {
    if (this.output) return;
    const template = document.createElement("template");
    template.innerHTML = `<output name='word-count' for='${this.element.id}' style='float: right;'></output>`;
    const output = template.content.firstChild;
    this.element.insertAdjacentElement("beforebegin", output);
    this.output = output;
  },
  updateCount: function (event) {
    const value = event ? event.target.value : this.element.value;
    const words = (value || "").split(" ");
    this.output.textContent = `${words.length} / ${this.maxValue} words`;
  },
  disconnect: function () {
    this.element && this.element.remove();
  },
};

document.addEventListener(
  "wagtail:stimulus-ready",
  ({ detail: { createController, register } }) => {
    register({
      controller: createController(wordCountController),
      identifier: "word-count",
    });
  },
  // important: stimulus-ready may be called more than once, only run the registration once
  { once: true }
);
```

```python
# models.py
# https://docs.wagtail.org/en/stable/reference/pages/panels.html#fieldpanel
from django import forms

class BlogPage(Page):
    # ...
    content_panels = Page.content_panels + [
        FieldPanel('subtitle', classname="full"),
        FieldPanel(
            'introduction',
            classname="full",
            widget=forms.TextInput(
                attrs={
                    'data-controller': 'word-count',
                    'data-word-count-max-value': '5',
                    'data-action': 'word-count#updateCount paste->word-count#updateCount',
                }
            )
        ),
    #...
```

```python
# wagtail_hooks.py
# https://docs.wagtail.org/en/stable/reference/hooks.html
from django.utils.html import format_html_join
from django.templatetags.static import static

from wagtail import hooks


@hooks.register('insert_editor_js')
def editor_js():
    js_files = ['js/word-count-controller.js',]
    return format_html_join('\n', '<script src="{0}"></script>',
        ((static(filename),) for filename in js_files)
    )

```

##### Adding a word count controller (with a build system or ES6 modules)

- Install `@hotwired/stimulus` using `npm install @hotwired/stimulus --save`
- Alternatively you can simply use ES6 modules with a path to the Stimulus module or a public URL.
- Wagtail does not yet provide a controller to be imported, you will need to 'bring your own controller' class. This is due to conflicts with ES6 modules and the currently ES5 transpile target of Wagtail's JavaScript.

```javascript
// myapp/static/js/word-count-controller.js
// import { Controller } from "https://unpkg.com/@hotwired/stimulus/dist/stimulus.js"; // can be used as an ES6 module import
import { Controller } from "@hotwired/stimulus";

class WordCountController extends Controller {
  static values = { max: { default: 10, type: Number } };

  connect() {
    const output = document.createElement("output");
    output.setAttribute("name", "word-count");
    output.setAttribute("for", this.element.id);
    output.style.float = "right";
    this.element.insertAdjacentElement("beforebegin", output);
    this.output = output;
    this.updateCount();
  }

  setupOutput() {
    if (this.output) return;
    const template = document.createElement("template");
    template.innerHTML = `<output name='word-count' for='${this.element.id}' style='float: right;'></output>`;
    const output = template.content.firstChild;
    this.element.insertAdjacentElement("beforebegin", output);
    this.output = output;
  }

  updateCount(event) {
    const value = event ? event.target.value : this.element.value;
    const words = (value || "").split(" ");
    this.output.textContent = `${words.length} / ${this.maxValue} words`;
  }

  disconnect() {
    this.element && this.element.remove();
  }
}

document.addEventListener(
  "wagtail:stimulus-ready",
  ({ detail: { createController, register } }) => {
    register({
      controller: WordCountController,
      identifier: "word-count",
    });
  },
  // important: stimulus-ready may be called more than once, only run the registration once
  { once: true }
);
```

```python
# models.py
# https://docs.wagtail.org/en/stable/reference/pages/panels.html#fieldpanel
from django import forms

class BlogPage(Page):
    # ...
    content_panels = Page.content_panels + [
        FieldPanel('subtitle', classname="full"),
        FieldPanel(
            'introduction',
            classname="full",
            widget=forms.TextInput(
                attrs={
                    'data-controller': 'word-count',
                    'data-word-count-max-value': '40',
                    # decide when you want the count to update with data-action (e.g. 'blur->word-count#updateCount' will only update when field loses focus)
                    'data-action': 'word-count#updateCount paste->word-count#updateCount',
                }
            )
        ),
    #...
```

##### Events

Event listeners and event dispatching is how to interact with Stimulus.

##### Dispatched - `'wagtail:stimulus-init'`

- Called once the Stimulus application has been initialised, most Wagtail code will not be able to listen to this event, however if more complex JavaScript is injected to run before Wagtail's code it may be useful.
- It will be dispatched just before `stimulus-ready`
- **Cancellable** - No

##### Dispatched - `'wagtail:stimulus-ready'`

- Dispatched on `DOMContentLoaded` and DOM ready state changes to indicate that Stimulus is ready.
- `detail.createController` a function that accepts an object with `STATIC` and other values which will be built into a Stimulus Controller class.
- `detail.register` a function that accepts an array with an `identifier` (kebab-case string) and a controller class, used to register a controller class to the Stimulus application.
- **Cancellable** - No

##### Listened to - `'wagtail:stimulus-enable-debug'`

- Must only be fired after the Stimulus application is ready.
- Will enable the debug mode for Stimulus, useful when working in local development mode.

```javascript
document.dispatch(new CustomEvent("wagtail:stimulus-enable-debug"));
```

##### Listened to - `'wagtail:stimulus-register-controller'`

- Must only be fired after the Stimulus application is ready.
- Provides an ad-hoc way to register controllers.
- The event dispatched must provide two values within its `detail`;
- `detail.identifier` a kebab-case string to use as the identifier.
- `detail.controller` a controller class.

#### Documentation for Contributors

> Proposed addition to the [UI development guidelines](https://docs.wagtail.org/en/latest/contributing/ui_guidelines.html).

##### Introduction to Stimulus

- Wagtail uses [Stimulus](https://stimulus.hotwired.dev/) as a way to attach interactive behaviour to DOM elements throughout Wagtail.
- This is a lightweight JavaScript framework that allows a JavaScript class to be attached to any DOM element that adheres to a specific usage of `data-` attributes on the element.

##### When to use Stimulus

This is a migration in progress, any large refactors or new code should adopt this approach.

1. Investigate if the browser can do this for you or if CSS can solve this (e.g. changing visual styling on focus/over should be done with CSS, autofocus on elements can be done with HTML attributes, using buttons with `type='button'` instead of `event.preventDefault`, or using links to take the user to a new page instead of a button with a click to change the page).
2. Investigate if there is an existing JavaScript approach in the code, we do not need to build multiple versions of things.
3. Write the HTML first and then assess what parts need to change based on user interactions, if state is minimal (e.g. less than 10 discrete moving parts) then Stimulus may be suitable.
4. Finally, if needed React may be suitable in small cases, but remember that if we want anything to be driven by content in Django templates React may not be suitable.

#### How to build a controller

1. Start with the HTML, build as much of the component or UI element as you can in HTML alone, even if that means a few variants if there is state to consider. Ensure it is accessible and follows the CSS guidelines.
2. Once you have the HTML working, add a new `HeaderSearchController.ts` file, a test file and a stories file. Try to decide on a simple name (on word if possible) and name your controller.
3. Add a `connect` method, this is similar to the `constructor` but gets called once the DOM is ready and the JS is instantiated against your DOM element.
4. You can access the base element with `this.element`, review the Stimulus documentation for full details.
5. Remember to consider scenarios where the element may be disconnected (removed/moved in the DOM), use the `disconnect` method to do any clean up. If you use the `data-action` attributes you do not need to clean up these event listeners, Stimulus will do this for you.

#### Stimulus best practices

- Smaller controllers that do a small amount of 'work' that is collected together, instead of lots of large controllers.
- Think about the HTML, use Django templates, consider template overrides and blocks to provide a nice way for more custom behaviour to be added later.
- Use data-attributes where possible, as per the documented approach, to add event listeners and target elements. It is ok to add event listeners in the controller but opt for the `data-action` approach first.
- Use `this.dispatch` when dispatching `CustomEvent`s to the DOM and whenever possible provide a cancellable behaviour. Events are the preferred way to communicate between controllers and as a bonus provide a nice external API, if the behaviour can be resumed use a `resume` function provided to the event's detail.
- Multiple controllers can be attached to one DOM element for composing behaviour, where practical split out behaviour to separate controllers.
- Avoid mixing jQuery with Stimulus Controllers as jQuery events are not the same as browser DOM events and can cause confusion, either find a non-jQuery solution or just attach the jQuery widget and set up your own non-jQuery event listeners.
- Telepath will still be used as a data pickle/un-pickle convention if required for more complex data setup.
- Avoid writing too much HTML (more than `textContent` or basic elements without classes) in the Stimulus controller, instead leverage the `template` element to move large amounts of HTML back into the Django templates. This also helps for translations which can be done in Django and co-located with the other HTML.

#### Documentation in folder

> Proposed content of a new `client/src/controllers/README.md` file. This is based on a proposed convention from the UI team to put a small `README.md` in each main client folder.

- Each file within this folder should contain one Stimulus controller, with the filename `MyAwesomeController.ts` (UpperCamelCase.ts).
- If the controller has a static method `isIncludedInCore = true;` then it will be automatically included in the core JS bundle and registered.
- Controllers that are included in the core will automatically be registered with the prefix `w` (e.g. `w-tabs`).
- Controllers are classes and will allow for class inheritance to build on top of base behaviour for variations, however remember that static attributes do not get inherited.
- All Controller classes must inherit the `AbstractController` and not directly use Stimulus' controller (this will raise a linting error), this is so that base behaviour and overrides can easily be set up.
- See **LINKS_TO_DOCS** for more information no how to build controllers.

## Open Questions

### Storybook compatibility

- The implementation so far has not resolved how to get Stimulus to work with Storybook, this is due to the non-transpiling of Stimulus as Storybook does not use the core webpack entrypoints but instead compiles using its own build tool.
- Help would be needed from other frontend devs to get this working.
- An idea for this would be to get Storybook to simply pull in the compiled core.js (or similar) built output from Webpack.

### API for a controller definition as an object

- Proposed approach is a simple object with a special key `STATIC` which contains any values that are to be the static variables. The naming of this can be changed.
- An alternative approach is to try to be smart about the object and look at any non-function and assume it is static (however, technically functions can be static but not really used by Stimulus).
- Another approach is to prefix with `_` or `$` to indicate that it is static but this may create more friction to using the object syntax.
- This is something we would like to not have to change, so careful thought is needed.

### Dispatched event names

- Stimulus has a convenience method on Controllers `this.dispatch` which will automatically prefix the controller's identifier to the event. So that `this.dispatch('next', { detail: {someDetail: true}})` on the controller with identifier `w-tabs` will dispatch an event with the name `w-tabs:next`.
- Wagtail has started adopting the convention that events should be prefixed with the name `wagtail:`
- We can either adopt the wagtail convention and override `dispatch` on the `AbstractController` so that using it would produce an event `wagtail:w-tabs:next` or we allow for two prefixes; `wagtail:` when not specifically related to a controller or `w-*:` when it is.

### Prefix on controllers

- Current implementation is `w-` which aligns with newer classes (Tailwind utility) and component classes (e.g. `w-dialog`), so have just aligned with this.
- However, it may look confusing at first glance when you see `class="w-tabs"` and `data-controller="w-tabs"` however the classes are for styles and the data attributes are for JS behaviour - as currently documented here https://docs.wagtail.org/en/latest/contributing/ui_guidelines.html#html-guidelines
- The critical part is the prefix for controller names, we can use something more specific like `wx-tabs` but this adds length and may cause confusion the other way (when to use `wx-` and when to use `w-`).
- Alternatively we can add a prefix to the Stimulus data attributes so that it would be something like `data-w-controller="tabs"`, while this is a powerful ability of Stimulus it means we would have to communicate clearly that our approach is not aligned with the Stimulus docs.

### TypeScript verbosity

- Using with Typescript will work but it is a bit verbose, we can probably add more TypeScript magic to the `AbstractController` if this becomes an issue, any TypeScript feedback would be appreciated here.
- https://dieterlunn.ca/stimulus-and-typescript/
- [targets and typescript · Issue #121 · hotwired/stimulus](https://github.com/hotwired/stimulus/issues/121) & https://github.com/hotwired/stimulus/issues/221
- https://www.sourlemon.co.za/blog/rails-typescript-and-stimulus/
- The next Stimulus release should solve the most common case using generics, see https://github.com/hotwired/stimulus/pull/540/files & https://github.com/hotwired/stimulus/pull/529

### Handling animations

- jQuery has a simple API to do basic animations, these animations are use haphazardly but they are convenient.
- We will not have an ergonomic replacement for these with Stimulus and will likely need to add an animation like API to controllers or set up a util to do this in a consistent way.
- We could solve for most common cases via some additional tailwind utility classes.

## Answered questions

### Why use a Django & HTML first approach

- We want to continue to use Django and HTML (templates) where possible and support existing Django packages and widgets. Wagtail is built on Django and should consider that as the first level API that users see, thankfully Django already has a great convention of setting `attrs` on widgets which aligns with the Stimulus approach really well.
- While more advanced usage such as `StreamField` requires someone to learn Telepath and understand JavaScript, this would be at the more complex end of the spectrum (note: Stimulus may offer a way to make the simple cases of StreamField widgets not require this but that is out of scope of this RFC).
- The Wagtail community have been asked to advise whether they want to use Node tooling and the answer is mostly no [Poll about node tooling](https://github.com/wagtail/wagtail/discussions/7739#discussioncomment-2244359). A similar feedback was received on Slack.
- See also this discussion to [allow modifications using only Python/Django](https://github.com/wagtail/wagtail/discussions/8128).
- While community feedback needs to be taken in consideration of the overall project goals it is good to get a sense that many Django/Python devs prefer to work day to day in just Django/Python (which kind of makes sense).

### Why not more React

#### Where React fits

- React is a different paradigm to HTML with JS, instead it reverses the world as JS first and foremost (JSX is just JS under the hood).
- React is an incredible solution high complexity, large state, components with a high level of interactivity (e.g. Draftail editor) where the inside implementation is meant to be obscured and hard to control by outside intervention.
- React components are great for a self-contained component, owning its own state where the DOM structure is contained 100% in React.
- React provides a consistent approach to one way data flow that helps for testing and development.

#### Why React is not a catch all solution

- React elements need to be initialised for any DOM to be added to the browser, the HTML it produces does not exist until React builds it.
- Harder to 'replace' an implementation with an extended one, while some React components may be class based components that could be extended there is no central and consistent way to provide this 'API'.
- Not possible to use HTML provided by the server in an easy way that also sets up any dynamic JS behaviour on nested HTML. If you wanted to leverage any of the existing Django ecosystem like templating, custom form widgets, template includes for somewhere 'inside' the React component it is just not possible.
- A specific API must be exposed to customise how the inside is rendered, and if you want this data from the server we need to reach for something like Telepath to do it.
  - An example would be the new sidebar required a core code change in React code to support aria attributes & data attributes on menu items there was literally no way to do this by customisation alone (except maybe setting up a mutation observer that fights with the React renderer). HTML template driven approaches are used widely in Wagtail and these kinds of use cases mostly do not need to be explicitly supported in other areas.
- React takes over the whole tree it controls, so everything down the tree must also be React which means reimplementing everything in React and non-React. While it is possible to add a DOM element and attach something to it (like a jQuery initialisation) it is not idea.
- It is not practical to output Django server rendered HTML inside a React element, Telepath provides a nice abstraction for data though.
- It is important to have options when building UI in Wagtail, but not too many options, at the moment any new JS only really has the option of go all React or just build whatever you want in JS. This leads to a disconnect between implementations and confusion for new developers.
- Not all developers want to or should have to learn React to contribute to Wagtail, this may be controversial but while React is 'just JavaScript' it is also a bigger learning curve. When building small and simple things (relative to the comments architecture or the dynamic API driven page explorer sub-menu) it should be possible for server side devs to contribute without having to learn too much.

### Why not use Vue, Svelte, Angular, or Solid etc

All of these frameworks/libraries have larger communities than Stimulus, however they opt the development into a strong direction of using the framework in all code.

Using these frameworks will make it hard to continue to use Django templates, support simple widgets that exist in the Wagtail ecosystem and also mean we need to really decide if React is going to live alongside these others.

### Why not Alpine.js

**[Alpine.js](https://alpinejs.dev/)** is quite popular in the Django space, however it is not CSP compatible and promotes even more JS syntax in the DOM which moves us further away from our security goals and vanilla JS usage goals.

- **Composing & Extending** - Not simple to [register additional components outside of the initialisation](https://alpinejs.dev/globals/alpine-data#registering-from-a-bundle), components can be composed but cannot do more than one 'thing', not really possible to extend existing registered components without specific APIs built.
- **CSP - Not compatible**, there is away to 'ignore' a branch of a DOM tree to not allow Alpine to read those elements. There is a planned [CSP compatible build](https://alpinejs.dev/advanced/csp), however this is [not yet released and no timeline available](https://github.com/alpinejs/alpine/issues/237#issuecomment-999692410), the CSP build lacks all the functionality of the `x-` attributes though and 100% relies on classes.
- **Development** - Mostly uses functions, not classes.
- **Initialisation** - Initialises when added to the DOM, either on first render or after, however does not 'disconnect' when `x-data` is removed from the DOM element.
- **Links** - [When to use Alpine](https://lightit.io/blog/when-to-use-alpine-js/), [HTMX & Alpine in Django](https://www.saaspegasus.com/guides/modern-javascript-for-django-developers/htmx-alpine/), [Alpine speed issues](https://github.com/alpinejs/alpine/issues/566)
- **Platform** - No way to namespace the HTML attributes but the data name can be prefixed by convention, potential for accidental conflicts with external libraries using Alpine.
- **State & Reactivity** - `data` object that is initialised and then self-contained in the component, not accessible outside, uses vue's reactivity model under the hood.
- We can however, include some of the great ideas from Alpine.js, see future roadmap for a `cloak` controller and also the way that this RFC proposes custom controller registrations is heavily inspired by Alpine's init.

**Comparison of Alpine directives vs Stimulus**

Stimulus and Alpine solve similar problems, here is a comparison on how to solve similar things between the two.

| Alpine                                                                                                     | Stimulus                                                                                                                                                                                                                         | JS/HTML   | Notes                                                                                                                                                                                                                                                                                                   |
| ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`x-data`](https://alpinejs.dev/directives/data#re-usable-data) <br> `<form x-data="search">`              | [Controller Identifiers](https://stimulus.hotwired.dev/reference/controllers#identifiers) <br> `<form data-controller-target="search">`                                                                                          | HTML      | Alpine JS lets you declare a name for a reusable function in `x-data`.                                                                                                                                                                                                                                  |
| [`x-data`](https://alpinejs.dev/directives/data) <br> `<form x-data="{open: false}">`                      | [Values](https://stimulus.hotwired.dev/reference/values) <br> `<form data-controller-target="search" data-search-open-value="false">`                                                                                            | HTML      | Stimulus encourages data values for initial and also reactive value states, these can be defaulted in the JS controller and also typed.                                                                                                                                                                 |
| [`x-init`](https://alpinejs.dev/directives/init) <br> `<form x-init="console.log('ready!')">`              | [Lifecycle Callbacks - `initialize / connect`](https://stimulus.hotwired.dev/reference/lifecycle-callbacks#methods) <br> `export class extends Controller { connect() { console.log('ready!); } };`                              | JS        | This where Stimulus moves you to JavaScript - when you are writing JavaScript.                                                                                                                                                                                                                          |
| [`x-ref`](https://alpinejs.dev/directives/ref) <br> `<input type="text" x-ref="query">`                    | [Targets](https://stimulus.hotwired.dev/reference/targets) <br> `<input type="text" data-search-target="query">`                                                                                                                 | HTML      | Stimulus scopes targets to controllers with the data attribute naming convention.                                                                                                                                                                                                                       |
| [`x-on`](https://alpinejs.dev/directives/on) <br> `<input type="text" x-on:change="alert('new search!')">` | [Actions](https://stimulus.hotwired.dev/reference/actions) <br> `<input type="text" data-action="change->search#update">` or `<input type="text" data-action="change->search#update">`                                           | HTML & JS | Stimulus lets you declare the DOM event that triggers a behaviour but the behaviour itself is written in JS.                                                                                                                                                                                            |
| [`x-transition`](https://alpinejs.dev/directives/transition#applying-css-classes)                          | No direct equivalent, however declaring [Stimulus CSS Classes](https://stimulus.hotwired.dev/reference/css-classes) solves part of this problem. <br> `<form data-controller="search" data-search-loading-class="search--busy">` | N/A       | Stimulus is quite lightweight and would require us to build or use our own animations system (see open questions above).                                                                                                                                                                                |
| [`x-ignore`](https://alpinejs.dev/directives/ignore)                                                       | No equivalent, would need to block `data-` attributes in user generated HTML rendering.                                                                                                                                          | HTML & JS | This is much less of a risk for Stimulus but still to be considered.                                                                                                                                                                                                                                    |
| [`x-cloak`](https://alpinejs.dev/directives/cloak)                                                         | No equivalent, easy to build our own though as multiple controllers can exist on one element <br> `<form data-controller="search cloak" hidden>`                                                                                 | HTML & JS | In this example, the `search` controller has the search specific behaviour and the generic `cloak` controller will wait for other controllers to connect before removing the `hidden` attribute. [See POC code](https://github.com/lb-/stimulus-starter/blob/main/src/controllers/cloak-controller.js). |
| [`x-id`](https://alpinejs.dev/directives/id)                                                               | No equivalent                                                                                                                                                                                                                    | N/A       | Each controller has its own scope, even when nested so this is not an issue.                                                                                                                                                                                                                            |

Multiple alpine directives are not included (`x-show`, `x-bind`, `x-text`, `x-html`, `x-model`, `x-modelable`, `x-for`, `x-teleport`) as they more relate to either updating classes, mapping the values to behaviour or more complex rendering of HTML - Stimulus moves all this to the JS controller.

### Why not HTMX or Turbolinks

These libraries provide a way to patch in server-side HTML to parts of the DOM. This is useful but does not serve the purpose of the lightweight frontend framework. They are a complement to Stimulus instead of a replacement and they require a different approach to how server side partials are provided to the frontend.

Adopting Stimulus does not mean we cannot adopt `htmx` or something similar in the future, however this RFC is not about server-side driven HTML partials and assumes that we will keep the current approach for this (e.g. search results listing) but just move the existing JS from jQuery to a Stimulus controller using vanilla JS (e.g. `fetch` instead of `$.ajax`).

- [htmx has `hyperscript`](https://htmx.org/docs/#hyperscript) (similar to Alpine.js / Stimulus in purpose) and explicitly points out that this solves a different set of problems to HTMX.
- [hotwire has `turbo`](https://turbo.hotwired.dev/) which is similar to HTMX in the goals but intentionally isolated from the code and purpose of Stimulus.
- The [Django unicorn docs](https://www.django-unicorn.com/docs/) provide a great snapshot of the various libraries that are out there solving similar problems.

### Why Stimulus and not something else

- No matter what, we still need to write some JS to get the behaviour working, even with Alpine.js (due to the CSP build approach), so the focus should be on providing a consistent approach to building this that makes it easy for HTML to still be the first class citizen.
- Classes can be abstracted so that it is easy to separate the behaviour of an element from the classes that get added/removed (e.g. collapse element could be used but with `.my-custom-collapsed-class` defined), default classes can be set up if not provided also.
- Allows for default variables, so that data can be supplied as needed.
- Allows for the target elements to be changed (e.g. you could write the expanded formset in a way that the 'add' button is elsewhere in the DOM, or maybe there are two add buttons, with HTML only).
- Stimulus Controllers are not really components but more a 'chunk of behaviour' and as such it is a pretty powerful way to split out JS behaviour.
- Everything is based on data attributes, which we already use for many of our existing components such as finding nodes for sidebar/draftail and finding nodes for dropdown, plus some values are already read from data attributes (e.g. chooser modal), the exact attributes will change but it is not a stretch for all of these usages to be used by Stimulus.
- This means that if some customised version wants to opt-out of some behaviour, all they need to do is either remove the `data-controller` attribute (e.g. maybe on an Django field widget OR just one line of JavaScript) and that element will be disconnected (or never connect) from Stimulus and not do anything. Remember - it is not easy to remove event listeners with JS, but it is easy to change a data attribute.
- Even the mounting of React components could leverage Stimulus, providing a simple way to initialise various DOM elements with their React injection, with some of the props being supplied to the component as data attributes... which can even trigger a re-render if the data-attributes change from some other JS.
- Stimulus JS is modest, it does not try to solve everything (no animations), DOM manipulations still need to be written in JS for example.
- As the state is stored on the data attributes, any other code can modify these attributes to change the behaviour (for example, want to close all collapsibles, just change `data-w-collapsible-collapsed-value` to false with any JS and it will work), this means that Django templates can be used extensively for 'initial' data without having to write any init JS functions.
- This library is a core part of the Rails ecosystem (as of V7) and built by the team at Basecamp, it is unlikely to go anywhere anytime soon.
- We can also provide the ability to trigger the [debug flag](https://stimulus.hotwired.dev/handbook/installing#debugging) to be true when Django is in local dev mode, this will aid those customising Wagtail and also those working on Wagtail core. The current implementation has this via an event listener.

## Additional Information

### Links

- [POC Wagtail Stimulus Controllers](https://github.com/lb-/stimulus-starter/tree/main/src/controllers) - includes Bulk Actions, InlinePanel, Cloak Controller, Autoresize TextArea, Sortable and more.
- [Stimulus 2.0 - HN comments](https://news.ycombinator.com/item?id=25305467) - the good, bad and ugly feedback
- [Official Stimulus discussion board](https://discuss.hotwired.dev/)
- [Blog - Intro to Stimulus](https://www.smashingmagazine.com/2020/07/introduction-stimulusjs/)
- [Awesome Stimulus JS](https://github.com/skatkov/awesome-stimulusjs)
- [Better Stimulus](https://www.betterstimulus.com/) - a set of resources (similar to awesome list) + best practices/recommendations.
- [Podcast - Changelog - Stimulus JS](https://changelog.com/podcast/286)
- [jQuery events to DOM events](https://github.com/leastbad/jquery-events-to-dom-events) - For Stimulus or not, this library looks great and simple, allowing bidirectional event passing between jQuery and non-jQuery events
- [Mutation first development](https://leastbad.com/mutation-first-development) - good article on why handling initialisation manually (of components, e.g. jquery or even React dom render) is problematic.

### CSP / Inline Scripts

This RFC can possibly be considered as an opinionated extension of the existing [RFC 33](https://github.com/wagtail/rfcs/pull/33). Adopting Stimulus gives us a clear roadmap of how to move away from inline scripts for all existing usage. Even InlinePanel, while that is probably the more complex one, has been validated at a POC level with Stimulus while also adding features like drag & drop, undelete and copy.

See these additional issues for inline script usage and CSP issues

- https://github.com/wagtail/wagtail/issues/7053
- https://github.com/wagtail/wagtail/issues/1288
- https://github.com/wagtail/wagtail/issues/5247

### Stimulus in the wild

- [Full Library of Stimulus controllers](https://sub-xaero.github.io/stimulus-library/docs) - great example of a BaseController / Typescript usage but no unit tests it seems, We may not want to use this library but it is a good reference.
- **Stacks** - UI library used by StackOverflow - https://stackoverflow.design/product/guidelines/javascript/ & https://github.com/StackExchange/Stacks/blob/develop/lib/ts/stacks.ts (uses Stimulus v2)
- **Kanety** - A bunch of Stimulus controllers in isolated packages - https://www.npmjs.com/~kanety
- **Groundwork** - Django applications with JS components built with Stimulus - https://github.com/commonknowledge/groundwork
- **Orchid** - Lavarel framework with some good docs on how to use Stimulus inside their application - https://orchid.software/en/docs/javascript/#stimulus + good article on their justification of Simulus https://blog.orchid.software/lasting-stack/
- **Stimulus Components** - Library of components built with Stimulus https://github.com/stimulus-components/stimulus-components

### Future possibilities

Not in scope of the RFC but could be an option in the future.

- Wagtail's Template Components and Panels should provide the ability to declare `attrs` (similar to Django widgets) to make it easier for core code to leverage Stimulus and custom code.
- Leverage the Wagtail npm module to make controllers available outside of Wagtail.
- Provide access to the Stimulus application instance in events so that more complex customisations can change or even extend existing registered controllers.
- Implement something similar to [Alpine.js `x-cloak` directive](https://alpinejs.dev/directives/cloak), this is quite useful when you want to wait for the JS to trigger before showing some content.
- Provide a way for simple cases of `StreamField` usage to not require Telepath code to be set up, a generic `Block` that would provide a way to add `data-` attributes to new node instances would probably negate the need for Telepath in many cases (e.g. the textarea block). This is because Stimulus will instantly connect to any `data-controller=...` element when added to the DOM.

## Why not Stimulus at all

It would be good to list the reasons we may not want to go this direction to either ensure they are discussed or understood as risks.

- We tie ourselves to an external JS library, like any library it could be deprecated, abandoned or its API change significantly in the future. This library is a core part of Rails 7 so hopefully that gives us some confidence it will be kept stable.
- Wagtail wants to head towards a SPA (Single Page App) direction in the admin and in this roadmap, everything in the admin will be React and hence Stimulus or similar will not be needed.
- There is not a real goal to make the JS widgets that Wagtail comes with reusable by custom code, nor extensible with arbitrary HTML, if so we may want to adopt web-components which isolate the HTML inside.
- We build something bespoke that provides a similar set of solutions to Stimulus but maybe it is React driven instead. This would be a huge undertaking but in theory it would be possible to bootstrap a DOM element based on data attributes, 'eat' the inner HTML (or a template element) and convert it to a React DOM tree while also setting up any initial data, event listeners and sub-elements.