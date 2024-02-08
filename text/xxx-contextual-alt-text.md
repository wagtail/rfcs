# RFC x: Contextual alt text

* RFC: x
* Author: Matthew Westcott
* Created: 2020-05-31
* Last Modified: 2020-05-31

## Abstract

Wagtail's default image model does not currently provide an alt text field - the title is used instead. Wagtail could add a dedicated alt text field (and developers often do so on custom image models); however, ideally [alt text should be tailored to the context where the image is being used](http://www.ala.org/support/context-important-alt-text) - for example, a gallery implemented with an InlinePanel would make the alt text a property of the GalleryItem model - and adding a field at the image level would interfere with this 'ideal' approach. Unfortunately, since both approaches require up-front effort on the part of the developer, it's common for neither one to be implemented. We therefore seek a solution for context-aware alt text that is as frictionless as possible for both developers and editors. This RFC proposes a solution for this within StreamField image blocks.

## Specification

A proof of concept is available at [gasman/wagtail/feature/imageblock-with-alt](https://github.com/gasman/wagtail/commit/1f908fc403dbb9f06c76924b8a6d25f3e7b6df07).

We add a `contextual_alt_text` attribute (as opposed to a model field) to the Image model, and have Rendition's `alt` property pick up alt text from there when defined, in preference to the image's `default_alt_text` property (which returns the title, or can be overridden on custom image models to a more appropriate field). With this in place, any mechanism within Wagtail that would ordinarily return an Image instance (e.g. ImageChooserBlock) can set `contextual_alt_text` to a context-specific string so that it will be used as alt text at the point of rendering the image - without the template author having to explicitly handle alt text, and without any database-level changes to the image.

We now introduce a new `ImageBlock` StreamField block, a backwards-compatible variant of `ImageChooserBlock`. Within the editor this is presented as the usual image chooser widget with a text field for alt text alongside it. `ImageChooserBlock` will still remain, for scenarios where alt text is not applicable (e.g. as part of a call-to-action block that allows specifying a background image).

On choosing an image, the text field is auto-populated with the `default_alt_text` of the image (i.e. the title, or a more appropriate field for custom image models) but can be changed by the editor. By auto-populating the field, we ensure that the new field does not create any new data-entry work for editors.

The native value of the block (the value returned when iterating over blocks in a template) is an Image instance with the `contextual_alt_text` attribute set to the content of that field.

The JSON representation of the block (as stored in the database) is a dict consisting of the image ID and the alt text string; however, when loading back from the database an image ID (by itself, not inside a dict) is recognised as an alternative format. This means that developers can change an ImageChooserBlock to an ImageBlock in their StreamField definitions without the need for a data migration (and without any template changes).

By naming the new block type ImageBlock (rather than, say, ImageChooserWithAltBlock), it is hoped that it will become established as the preferred block to use for images, regardless of whether the developer has an awareness of accessibility issues. It also leaves open the possibility of making further 'opinionated' changes in future that support Wagtail's goal of promoting best practices on the web.