# RFC 33: Inline script deprecation

* RFC: 33
* Author: Matthew Westcott
* Created: 2019-03-01
* Last Modified: 2019-03-01

## Abstract

Inline script elements are used widely throughout the Wagtail admin to add custom client-side behaviour to form widgets, using a pattern such as

    <input type="text" id="id-publish_date" name="publish_date">
    <script>initDateChooser('id-publish_date', 'yyyy-mm-dd', 'en-GB');</script>

(where `initDateChooser` is implemented in an external JS file, imported through form media). This works, and has the nice feature of not needing any special treatment when widgets are dynamically added to the page (as seen in `InlinePanel`, `StreamField` and modals). However, a neater approach would be to define widget parameters through `data-` attributes, and have the external JS code apply behaviour based on these:

    <input type="text" data-widget="dateChooser" data-format="yyyy-mm-dd" data-language="en-GB" name="publish_date">

This provides the following advantages:

* Code is more compact and readable
* It follows the letter and intent of [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), which expects all scripting to be statically defined up-front
* React code that dynamically inserts widgets does not have to enable the `dangerouslyRunInnerScripts` flag
* More flexibility over how JS includes are defined (they can now be placed in the footer or loaded asynchronously)

This RFC proposes a new convention to be used within the Wagtail admin whenever Javascript behaviour needs to be attached to HTML elements. This convention is intended to be agnostic of any client-side framework, and will handle elements being inserted dynamically.


## Specification

Consider a widget that is currently implemented as:

    <input type="text" id="id-publish_date" name="publish_date">
    <script>initDateChooser('id-publish_date', 'yyyy-mm-dd', 'en-GB');</script>

    # date-chooser.js
    function initDateChooser(id, format, language) {
        /* Set up date chooser behaviour on the element with the given ID */

        var element = document.getElementById(id);
        /* ...define custom behaviour on element */
    }


Rather than defining a global `initDateChooser` function to be called on an individual element ID, each widget type shall instead define a distinctive set of `data-` attributes. As a convention, it is proposed that each widget type adopts a unique identifier to use as the `data-widget` attribute. The associated JS include defines an initialisation function which accepts a container element (defaulting to the whole document) and sets up the desired behaviour on all matching elements within that container. This initialisation function is then attached to the `DOMContentLoaded` event.

The above widget could thus be rewritten as:

    <input type="text" data-widget="dateChooser" data-format="yyyy-mm-dd" data-language="en-GB" name="publish_date">

    # date-chooser.js
    function initDateChoosers(container) {
        (container || document).querySelectorAll('[data-widget="dateChooser"]').forEach(function(element) {
            var format = element.dataset.format;
            var lang = element.dataset.language;
            /* ...define custom behaviour on element */
        });
    }

    document.addEventListener('DOMContentLoaded', initDateChoosers, false);

The additional attributes (`data-format` and `data-language` here) are arbitrary and specific to that widget type. If the widget requires parameters that are more complex than strings/numbers, it is suggested that these are passed as JSON strings.


### Dynamically-created elements

The `DOMContentLoaded` event will not account for elements that are added dynamically after page load. To accommodate these, the Wagtail admin will maintain a global list of widget initialiser functions, and provide two global functions:

    registerWidgetInit(initialiser)

Adds `initialiser` to the global list of initialiser functions; to be called alongside attaching the `DOMContentLoaded` event handler.

    initWidgets(container)

Calls all registered initialiser functions, passing `container` as the container element. Any piece of Javascript code that dynamically inserts HTML (potentially including widgets) into the DOM shall call this immediately after performing the insertion.

It is suggested that widget code should test the existence of `registerWidgetInit` before calling it, so that it continues to work outside of Wagtail admin. The final `date-chooser.js` thus becomes:

    function initDateChoosers(container) {
        (container || document).querySelectorAll('[data-widget="dateChooser"]').forEach(function(element) {
            var format = element.dataset.format;
            var lang = element.dataset.language;
            /* ...define custom behaviour on element */
        });
    }

    document.addEventListener('DOMContentLoaded', initDateChoosers, false);
    if (window.registerWidgetInit) registerWidgetInit(initDateChoosers);


### Deprecation process

Even after converting Wagtail's own widget code to this convention, third-party JS code might still continue to use inline `<script>` tags, and so it will not be possible to deactivate React's `dangerouslyRunInnerScripts` flag immediately. We will introduce a settings flag `WAGTAIL_RUN_INNER_SCRIPTS`, initially defaulting to True, and pass this value on to `dangerouslyRunInnerScripts`. After two releases, the default will be changed to False. This will provide a two-release window for third-party code to be updated to the new convention. Sites which want to make use of strict CSP, and do not have any legacy widgets in use, can set `WAGTAIL_RUN_INNER_SCRIPTS` to `False` before this period is up; conversely, sites that are still using legacy widgets after this period can set `WAGTAIL_RUN_INNER_SCRIPTS` to `True`.


## Open Questions

* The above deprecation process will provide backwards compatibility for legacy third-party widgets, but will not provide backwards compatibility for third-party code that performs DOM insertions, as this code will not know to call `initWidgets`. Is there a way to provide this?
