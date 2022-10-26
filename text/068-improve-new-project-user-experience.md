# RFC 68: Improve the new project user experience

* RFC: 68
* Author: Brylie Christopher Oxley
* Created: 2021-06-20
* Last Modified: 2021-09-08

## Abstract

Users will often want to jump straight to trying out the Wagtail admin interface when starting a new Wagtail project. By default, Wagtail comes with a `HomePage` model consisting of only a title field. When users change the title and view the live `HomePage` instance, they are greeted with the static home page content "welcome to Wagtail." Furthermore, when creating a new `HomePage` instance, it is unclear how to view the newly created content. This user experience can create confusion and make a poor impression early in a person's Wagtail journey. 

Wagtail caters to developers, and the introduction tutorial is reasonably comprehensive. However, it may be helpful to present a more "functional" wagtail out of the box, to some extent similar to a WordPress base installation.

It is probably beneficial to flip the perspective and treat the Wagtail Admin UI as the priority for new-user onboarding and secondary (excellent) developer experience.

## Specification

### Add a StreamField `body` field to the HomePage model

A page without any editable text is somewhat underwhelming. Perhaps it would be an opportunity to showcase the wagtail stream field by adding a `StreamField` as the `body` field.

### Add a `RootPage` proxy model

The Wagtail page tree "root" can be designated by creating a new `RootPage` model that is a proxy of `Page`.

### Allow a single `HomePage` model instance under `RootPage`

There should be only one Home Page. The Home Page should reside near the Wagtail page root.

- Define the `parent_page_types` field in the `HomePage` model to only create the Home Page under the `RootPage`.
- Add a `max_count = 1` to the `HomePage` model so there can only be one home page.

```py
from wagtail.wagtailcore.models import Page, RootPage

class HomePage(Page):
    # There should be only one home page
    max_count = 1
    
    # Homepages can only be created at the root level
    parent_page_types = [RootPage]
```


### Create a `ContentPage` model

Since this RFC aims to improve the new user experience and new users will likely want to create a new page, adding a `ContentPage` model would be helpful.

The `ContentPage` model would allow a new user to scaffold some test content while getting oriented to the Wagtail admin workflow. Similar to the `HomePage`, adding a `StreamField` to the `ContentPage` `body` would allow new users to try out the Wagtail stream field.

### Change default home page/welcome text template structure

The welcome text with a wobbling egg is a great way to build quick excitement when starting a new Wagtail project. So, we should keep that text.

However, when a user changes the page content, such as adding content in the `body` field, the `ContentPage` or `HomePage` instance should show the `body` content instead of the default welcome template.

### Instructions for cleaning up demo/welcome content

Once the new user is satisfied with their understanding of the Wagtail Admin UI, they will likely want to roll up their sleeves and change how things work. Therefore, it might be good to provide some brief documentation on exactly what to remove when the developer intends to tailor the Wagtail content model to fit the requirements of their project.

## Questions

### Open questions


### Resolved questions

- What would be a good model name instead of `HomePage`? E.g. `Page` would make sense but is taken. Perhaps `DemoPage`?
