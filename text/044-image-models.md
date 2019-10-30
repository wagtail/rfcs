# RFC 44: Make "custom image model" the default

* Authors: Karl Hobley
* Created: 2019-10-23
* Last Modified: 2018-10-23

## Abstract

Custom image models is a feature that Wagtail has had since the very beginning. It allows developers to replace Wagtail's built-in image model with their own one and customise it by adding new fields or overriding methods to change behaviour.

The problem is it’s very hard to migrate to them once you already have built a website that uses Wagtail’s built-in image model. This also makes it difficult for us to build optional add-on features for images that require a custom image model (such as AMP support or translation).

This RFC proposes that we start creating all new Wagtail projects with a custom image model from the start. This will add a small amount of extra code to the project template, which I know we would want to avoid. But I think avoiding a potentially very difficult migration later outweighs the overhead of having one extra app in the project.

This also applies to document models as well.

## Specification

To implement this RFC, we will make the following changes:


### Update the project template to use custom image, rendition, and document models

We will add a new app called `assets` into the base project template. This will contain three models: `Image`, `Rendition` and `Document`.

All three of these models will exactly match their built-in counterparts (we will not add an `alt` field to the image model out of the box).


### Add a custom image model to bakerydemo

We will add the `assets` app as described in the previous section and update the fixtures and foreign keys to use this new app.


### Rename the built-in models to fix naming conflict and discourage their use

Currently, custom image models are often called `CustomImage` or `WebsiteNameImage`. This is because you can’t call the model `Image` because this would cause the name of a reverse relation to clash.

We will not remove the built-in image model as that would force developers to perform a migration that this RFC hopes to put an end to. But we could rename it to something obsolete sounding to discourage its use (`LegacyImage`?). This should only require developers to update foreign keys and create a simple schema migration to update for.


### Tweak the beginners tutorial so it uses the custom image model

We will only make necessary tweaks so that it works with the new project template. I don’t think there’s any need to describe the image model in the beginners tutorial.


### Remove the “Custom image model” advanced topics doc

We could replace this with a reference document that describes the methods that developers can override to tweak its behaviour.


### Stop calling it “Custom image model”

The documentation will be updated to no longer refer to “Custom Image models” as an extra feature any more.

