# RFC : Draftail Usage for General Text Entry

* RFC: 
* Author: Jacob Topp-Mugglestone
* Created: 2020-09-17
* Last Modified: 2020-09-17

## Abstract

This RFC proposes using cut back versions of Draftail for general text entry, including as the default widget for `TextField` and `CharField`. 


## Motivation

Currently, we have two main types of text entry: plain text, and rich text. Rich text entry uses the Draftail editor, which provides formatting functionality.

However, Draftail also enables/massively simplifies admin utility features like [inline comments](https://github.com/jacobtoppm/rfcs/blob/commenting/text/050-commenting.md), and [tone guidance](https://youtu.be/IMNFjrQ5OY4?t=1881), as well as other types of content analysis. These are features which are not intrinsically linked to a field using rich text formatting, and having them only on certain field types can create inconsistencies for users. Taking tone guidance as an example, users would be picked up for not following the style guide in a rich text ‘introduction’ field, but not in the page’s title. Taking commenting, “highlight text to add a comment” on all text fields is a much more consistent and user-comprehensible approach than having some text fields allowing inline comments, and some not.

Draftail provides a useful framework for attaching entities to text - such as comments, or spellcheck suggestions - and ensuring their positions are kept consistent as text is edited, as well as storing any state (deciding to hide a tone or spellcheck suggestion, for instance). It’s also easy to extract plain text content.

As a result, we propose making a cut down version of the Draftail editor (with no rich text features enabled) the default widget used for text entry in Wagtail, used to provide inline commenting on all text fields, as well as any text analysis features chosen. This would involve no changes to the database representation of these fields, so would be a smooth transition for users.

## Advantages
- Consistent approach to admin features for users: commenting and plugins can be used in the same way everywhere.
- Consistent approach for adding text features for developers:  no need to write both a Draftail plugin and custom widgets to add a single feature globally
- The new default widget can still be overridden to use the old version if desired


## Performance Comparison

We used the “Welcome to the Wagtail Bakery” page of the standard bakerydemo site was to test the effects on edit view performance of switching from the standard widgets to mini Draftail widgets. We tested three scenarios:

- The standard page (featuring a title, a hero and promo section, and a one block streamfield)
- The page as previously, with 5 additional heading blocks added to the streamfield
- The page with 40 additional heading blocks added to the streamfield

For each scenario, we analysed the page loading using Chrome DevTools five times, and profiled the interaction of adding five characters to an existing rich text field five times. For both loading and interaction phases, we calculated the average peak nodes, listeners, and JS heap size. For the loading phase, we also calculated the average DOM Loaded event time, and for the interaction phase, the average keypress response time. The results are given in a spreadsheet here: https://docs.google.com/spreadsheets/d/161rDpM6-4OvfRNt7Voge6EbZSNZJ6GmOTKzZ7WiLv60/edit?usp=sharing

The main points:

- The keypress response time differed very little between Draftail and plain widget scenarios, so this should not substantially impact the interaction time.
- Memory usage during the interaction phase for the first two scenarios was very similar between Draftail and plain widgets. In the third scenario, memory usage increased by 20%, up to 8.4MB.
- DOM load time was also similar for the first two scenarios (within 350ms), but increased in the third scenario by 20%. This is something we could look at optimising the initialisation of when bringing this change into core, and may also be helped by other proposed streamfield improvements.

Ultimately, while the load time could be improved, the interaction was still fluid even in the 40 block scenario, so we feel the consistent admin utility layer for text fields is worth the slight performance cost. We can also provide a setting to disable the new widget entirely.


## Extracting Comments from Plain Text Fields:

Most text analysis features will solely be client side plugins, but comment positions will need to be extracted from plain text fields within Django. In order to work within the Django forms framework, we intend for the new plain text Draftail widget to return a `str` subclass which has a `get_comments` method. For compatibility with this, we will also need to adjust the form fields slightly so their `to_python` methods accept this subclass rather than coercing to a plain string. This ensures that all existing code that expects a string will function correctly.
