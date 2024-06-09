# RFC 097: Alt Text Capabilities

* RFC: 097
* Author: Chiemezuo Akujobi
* Created: 2024-06-04
* Last Modified: 2024-06-04

## Abstract

In its current state, Wagtail’s alt text functionality presents a significant accessibility challenge. Its approach to alt text relies heavily on the implementation of developers who use Wagtail, with the only default mechanism being that when alt text is not provided, the title of the image is used as the alt text. The problem with this is that image titles are also defaulted to what the file is saved as, and site editors are not mandated to change the titles from this default. Research by Wagtail’s accessibility team has shown that many sites have not attempted to customize the alt text mechanism, instead, relying on the default to file names. Much too often, this leads to a very poor experience for screen-reader users.

Additionally, the developers who create their own ways of enforcing alt text do so in a way that does not allow alt text to be tailored to the context in which the image is used. This is likely due to a general lack of awareness of the value that contextual alt text provides to users. Therefore with images in Wagtail, we have a lot of sites not adhering to Web Accessibility Standards, despite the developers behind them thinking otherwise.

## Intended Outcome

This RFC proposes a solution to improve the overall accessibility of Wagtail sites by expanding the alt text capabilities provided out of the box, thereby enforcing good practices. It approaches this by adding supported mechanisms for handling alt texts in their most common use-case scenarios; such as in `StreamField` Image blocks, `StructBlocks`, and by providing helpful image descriptions in the `AbstractImage` model.

Achieving these objectives, will by extension, provide a way for contextual alt text to be supported for images, especially when they appear in more than one place in a Wagtail website.


## Specification

There are three major changes needed:
1. Creation of an `ImageBlock` to support contextual alt-text.
2. Addition of an `image_description` field to the `AbstractImage` model.
3. Documentation efforts for both processes (1) and (2).

### 1. Creating `ImageBlock`
At this time of writing, Wagtail provides an `ImageChooserBlock`, and although we want to enforce standards and advocate for best practices, we do not want to make breaking changes or affect backward compatibility. This is the reason we will be creating a new block for images. 
We want a new block that can leverage existing logic and provide a means to add alt text, as well as a way to indicate whether certain images are purely decorative instead, while not needing us to change the existing `ImageChooserBlock` implementation. This will be the first major change. The name `ImageBlock` has been selected (although still up for deliberation) because for other StreamField blocks, the prefixes convey what their types are: such as `RichTextBlock` for rich text, `CharBlock` for characters, `EmailBlock` for emails, and so on. 

This `ImageBlock` class will subclass the `StructBlock` class, and it will have three properties:
1. An `image` property, which is an `ImageChooserBlock`.
2. An `alt_text` property, which is a `CharBlock.
3. A `decorative` property, which is a `BooleanBlock` (which renders as a checkbox)

These properties only define the visual layout in the Admin interface, but do not handle custom logic. To cater to this, this RFC proposes introducing and registering ImageBlock-specific Javascript logic (a new static file) for the Admin interface, to allow the `alt_text` field to be enabled/disabled by toggling the `decorative` checkbox. If disabled, the field will be cleared and its value will be an empty string. Additionally, this RFC proposes that the `StructValue` of this new block be an Image instead of a Dict. This will make alt text detection automatic when the `{% image %}` template tag is used. The `ImageBlock’s` behaviour will be identical to the behaviour already in place for the `ImageChooserBlock`. To achieve this, this RFC proposes overriding some parent methods, particularly the ones that typically return Dict values, as will be shown in the snippet in the paragraph below. Optionally, a decision can be made on whether or not alt texts of images should be searchable.

Do take a look at some snippets of the sample code implementation [here](https://github.com/wagtail/wagtail/pull/11791/files) (including a sample template).


### 2.  Adding `image_description` field to `AbstractImage`

The next major action is to improve image accessibility **within** the Admin interface for administrative users/editors. For this, we will add an `image_description` field to the AbstractImage class for a high-level overview of the image. It will be used as the alt text inside the image listing view and inside image picker dialogs. With this in place, the behaviour of the gallery in the admin interface would be such that stored images are described. This would be important to screen-reader users using the Wagtail admin, and it would be a step in mentally preparing editors to be thorough with alt text at the point of image usage. However, with backward compatibility in mind, this new field will default to a blank string for existing images.

### 3. Documentation efforts
We will include a note in the upgrade considerations on how developers can migrate. In practice, some developers have added their own alt text field to their custom image model. These migration instructions will help this group of developers migrate from their own alt text field to the new image description field.


## Caveats

Here are some of the things we will ideally not be doing:
1. We will not be doing away with the title field for the AbstractImage model. This is because titles will still be used to identify specific images, especially among a list or point of reference.
2. We will not fall back to the title in the absence of an alt text. This is because we want to enforce alternative text, and a fallback (to another fallback) would be counter-productive to this effort.
