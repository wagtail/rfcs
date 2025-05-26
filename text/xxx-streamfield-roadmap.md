# RFC x: Roadmap for new StreamField development

* RFC: x
* Author: Matthew Westcott
* Created: 2020-08-28
* Last Modified: 2020-08-28

## Abstract

This RFC presents a roadmap for future development of StreamField, based on surveys, interviews and general feedback from Wagtail developers.

## Specification

Based on the Wagtail developer survey and follow-up interviews, we've found that developers have the following feature requests for StreamField:

### UX and performance improvements to StreamField within the Wagtail admin

These include a number of features already implemented in third-party code - the ability to duplicate blocks (currently implemented in react-streamfield) and to split rich text blocks into multiple blocks (currently implemented as a customisation by NYPR).

Features such as these are currently blocked from being added to Wagtail core as they rely on undocumented properties of the client-side code which cannot be generalised to all block types, or cannot be guaranteed to remain stable in future Wagtail releases.

To illustrate: the rich client-side UX as seen in react-streamfield is built upon the ability to perform fundamental operations such as: "create a new instance of a StreamField block, populated with a given value". However, a key principle of StreamField is that any Django form field may be used as a block - and Django form fields do not guarantee anything about the client-side behaviour of a field (except to the extent that it needs to be able to produce an HTML form submission that the Django-side component can make sense of). Assumptions that might seem safe, such as a Django form field corresponding to a single HTML input element, cannot be relied upon - and without that, there is no fully robust way to implement those fundamental client-side operations.

(StreamField itself is an extreme example of this assumption being broken - it presents itself to other Django-side code as one form field, but may consist of many HTML form elements. This in itself is not a showstopper for react-streamfield, since there's no real prospect of a developer trying to embed StreamField inside a block inside StreamField. However, it does imply that any future development of a similar complexity to StreamField will not be usable within react-streamfield; and conversely, any future development that attempts to make the same assumptions as react-streamfield is liable to fail when presented with a StreamField. In other words: the "eat your own dogfood" test is a meaningful one to apply here!)

This limitation also causes performance issues in the current implementation of StreamField - since blocks populated with a given value cannot be created client-side, displaying the edit view for an existing page requires these to be rendered up-front on the server, often generating a lot of redundant HTML.

**Proposed development:** Define and implement a common interface for accessing and manipulating Django form fields on the front end. This will most likely take the form of a set of Javascript 'adapter' objects, one per field type, implementing operations on that field type such as "create a new instance of this field populated with the given value", and "extract the JSON-serialisable value from an instance of this field". Developers implementing new field types for use within StreamField will need to provide such an adapter for that field; for conventional form fields consisting of a single HTML input, this will be a straightforward and well-documented task.

### Easier ways to manipulate StreamField data within Python code

Wagtail does not provide any formal APIs for manipulating StreamField data in place. The officially recommended way to update StreamField data is to construct a new data structure replicating the previous value but with the desired changes applied, and assign that back to the StreamField; for complex nested structures, this is difficult and in certain edge cases, impossible. Workarounds include building the StreamField value in JSON format (which is a less human-friendly representation and adds an extra data-conversion round trip) or accessing StreamField internals (such as the StreamValue.stream_data property) directly, which is often poorly understood (leading to logic errors, especially when previewing) and liable to change across Wagtail releases.

**Proposed development:** Extend the existing StreamValue class (currently designed to be an immutable value object) with list-like methods that allow it to be modified in-place. Initial work on this has been started, at: https://github.com/wagtail/wagtail/pull/2886

### Ability to build your own StreamField-like editing interfaces outside of the Wagtail admin and have those interoperate with Wagtail's StreamField

Developers frequently request the ability to use StreamField on a site front-end, to allow users to contribute to a site without having a Wagtail editor account. Given the amount of work involved in making StreamField work in the relatively controlled environment of the Wagtail admin, and ensuring that all possible combinations of elements are styled appropriately, we do not feel that it's currently feasible to build a StreamField component for site front-ends with the full generality of the Wagtail admin's StreamField. However, a special-purpose StreamField interface built for one specific page type with a fixed set of blocks can take various shortcuts: it does not need to interoperate with Django's form framework, will most likely not need to handle arbitrary nesting of blocks and has fewer styling combinations to test.

As this would be site-specific bespoke code, written in the site implementer's preferred front-end framework, Wagtail itself would not provide any tools for building this; however, Wagtail does need to provide a way for the resulting data to be submitted back so that a page can be created.

**Proposed development:** Implement a REST API endpoint to allow creating pages from posted JSON data, where any StreamFields are also represented in JSON format. (It is expected that JSON will be a more convenient format for front-end developers to work with, rather than the HTML form submission currently used by the Wagtail admin.)

## Further development

Items 1 and 3 combined could potentially serve as the basis for an auto-save feature in the Wagtail page editor: the Javascript adapter system (item 1) would provide a way to extract the contents of the active edit view as JSON, and a background AJAX request could then post this JSON to a new REST API endpoint (item 3) that saves the data to a draft revision.
