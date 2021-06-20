# RFC 68: Improve the new project user experience

* RFC: 68
* Author: Brylie Christopher Oxley
* Created: 2021-06-20
* Last Modified: 2021-06-20

## Abstract

Users will often want to jump straight to trying out the Wagtail admin interface when starting a new Wagtail project. By default, Wagtail comes with a `HomePage` model consisting of only a title field. When users change the title and view the live `HomePage` instance, they are greeted with the static home page content "welcome to Wagtail." Furthermore, when creating a new `HomePage` instance, it is unclear how to view the newly created content. This user experience can create confusion and make a poor impression early in a person's Wagtail journey. 

Granted, Wagtail is oriented towards developers, and the introduction tutorial is reasonably comprehensive. However, it may be helpful to present a more "functional" wagtail out of the box, to some extent similar to a WordPress base installation.

It is probably beneficial to flip the perspective and treat the Wagtail Admin UI as the first priority for new-user onboarding and secondary (excellent) developer experience.

## Specification

### Add a RichText `body` field to the HomePage model

A page without any editable text is somewhat underwhelming. Perhaps it would be an opportunity to showcase the wagtail rich text editor by adding a `body` field with the `RichTextField` widget.

### Rename the HomePage model for re-use

Since the `HomePage` model does not have a `max_count`, it might be helpful to rename it to make it more generic. While `Page` would be a good generic name for pages in wagtail projects, it is already taken by Wagtail core. Since this RFC aims to improve the new user experience and the initial content is essentially demo content, it might make sense to have a `DemoPage` model.

The `DemoPage` model would allow a new user to scaffold some test content while getting oriented to the Wagtail admin workflow.

### Change default home page/welcome text template structure

The welcome text with a wobbling egg is a great way to build quick excitement when starting a new Wagtail project. So, we should keep that text.

However, when a user changes the page content, such as adding content in the `body` field, the `DemoPage` instance should show that content instead of the default welcome template.


### Instructions for cleaning up demo/welcome content

Once the new user is satisfied with their understanding of the Wagtail Admin UI, they will likely want to roll up their sleeves and change how things work. It might be good to provide some brief documentation on exactly what to remove when the developer intends to tailor the Wagtail content model to fit the requirements of their project.

## Questions

### Open questions

- What would be a good model name instead of `HomePage`? E.g. `Page` would make sense but is obviously taken. Perhaps `DemoPage`?

### Resolved questions
