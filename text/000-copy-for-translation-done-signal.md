# RFC 000: Copy for translation done signal

* RFC: 000
* Author: Coen van der Kamp
* Created: 2024-05-22
* Last Modified: 2024-05-22

## Abstract

This RFC proposes the inclusion of a "copy for translation done" signal within Wagtail. This signal will be triggered whenever content is copied into another locale, facilitating post-processing of copied content such as automating translation or alerting translators.

## Background

Historically, Wagtail relied on third-party packages for translation capabilities to maintain a lean and un-opinionated core. These packages often required patching Wagtail code, complicating maintenance and updates, and locking projects into specific solutions.

To address this, Wagtail introduced basic internationalization features like the TranslationMixin, which added locale and uuid fields, and user interface enhancements for switching between translations. However, these features serve as a foundation rather than a complete solution.

Wagtail Simple Translation (a contrib module) and Wagtail Localize (a third-party module supported by Torchbox) were introduced. They use Wagtail's internationalization features and provide the user interface. They are the most used packages for implementing translations in Wagtail.

Wagtail Simple Translation:

- A Wagtail contrib-module
- Provides a user interface for copying content into another locale
- Does not translate content
- Minimalistic

Wagtail Localize:

- A third-party package
- Provides a comprehensive feature set
- Overrides default Wagtail edit views to display source and translated strings side-by-side
- Facilitates synchronization of translations
- Supports features such as export/import and configuring machine translation services
- Comprehensive

Wagtail Simple Translation and Wagtail Localize are at opposite ends of the functionality/complexity spectrum. 

Although, at first glance, one might favour Wagtail Localize, most projects benefit from simpler solutions. For many multi-language projects that would be something in between these two. This RFC is about enhancing Wagtail's extensibility to facilitate the development of such solutions.

With a "copy for translation done" signal in place, the content translation processes can be triggered with the signal. Signal receivers that do the actual translation would complement Wagtail Simple Translation or any other code that calls `copy_for_translation`.

## Proposal

Introducing a "copy for translation done" signal to enhance Wagtail's extensibility. Signals operate at the model level, allowing to listen for this event and react accordingly. This brings the translation process to the model-level.

## Specification

The signal will be dispatched after content is copied for translation. It will include the sender (class sending the signal), and the source and target objects as arguments.

Providing both the source and target objects allows the signal receiver maximum flexibility in post-processing the content.

Example implementation:

```python
# wagtail/actions/copy_for_translation.py
import django.dispatch

copy_for_translation_done = django.dispatch.Signal()


class CopyPageForTranslationAction:
    ...
    
    def walk(self, current_page):
        for child_page in current_page.get_children():
            translated_page = self._copy_for_translation(
                child_page,
                self.locale,
                self.copy_parents,
                self.alias,
                self.exclude_fields,
            )

            # Send signal
            copy_for_translation_done.send(
                sender=self.__class__, 
                source_obj=child_page.specific,
                target_obj=translated_page,
            )

            self.walk(child_page)
            
    ...

    def execute(self, skip_permission_checks=False):
        self.check(skip_permission_checks=skip_permission_checks)

        translated_page = self._copy_for_translation(
            self.page, self.locale, self.copy_parents, self.alias, self.exclude_fields
        )

        # Send signal
        copy_for_translation_done.send(
            sender=self.__class__, 
            source_obj=self.page, 
            target_obj=translated_page,
        )

        if self.include_subtree:
            self.walk(self.page)

        return translated_page
```

A signal receiver example:

```python
from wagtail.actions.copy_for_translation import copy_for_translation_done
from django.dispatch import receiver


@receiver(copy_for_translation_done)
def post_process(sender, source_obj, target_obj, **kwargs):
    source_language_code = source_obj.locale.language_code
    target_language_code = target_obj.locale.language_code
    # Add post-processing, or other behavior here
    ...
```
Both `source_obj` and `target_obj` are instances of `TranslatableMixin`, which includes `locale` (a ForeignKey to Locale) and `uuid` fields.


## Alternatives

Triggering translation at the view level was considered but found impractical due to limited access to the source object and source locale, and tight coupling of translation processes to the view. This RFC's approach maintains separation of concerns and supports programmatic content copying.


## Open Questions

// Include any questions until Status is ‘Accepted’
