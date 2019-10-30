# RFC 44: Make "custom image model" the default

* Authors: Karl Hobley
* Created: 2019-10-23
* Last Modified: 2018-10-23

## Abstract

"Custom image models" is a feature that allows developers to supply a
model for Wagtail to use to store image metadata instead of using the
built-in one. This feature enables developers to add custom fields to
images and customise Wagtail's behaviour by overriding its methods.

The problem is that it\'s challenging to switch to a custom image model
once the site has already made use of Wagtail's built-in image model.
Because of this, it\'s difficult for us to build optional add-on
features for images that require a custom image model (such as AMP
support or translation) as many users will need to go through a
complicated migration to use them.

This RFC proposes that we start creating new Wagtail projects with a
custom image model from the start. This change will add a small amount
of extra code to the project template, which I know we would want to
avoid, but I think preventing a potentially arduous migration later
outweighs the overhead of having one extra app in the project.

This change also applies to image rendition and document models as well.

## Specification

To implement this RFC, we will make the following changes:

### Update the project template to use custom image, rendition, and document models

We will add a new app called `assets` into the base project template.
This app will contain three models: `Image`, `Rendition` and `Document`.

All three of these models will exactly match their built-in counterparts
(we will not add an `alt` field to the image model out of the box).

### Add a custom image model to bakerydemo

We will add the `assets` app as described in the previous section and
update the fixtures and foreign keys to use this new app.

### Rename the built-in models to fix naming conflict and discourage their use

Currently, custom image models are often called `CustomImage` or
`WebsiteNameImage` because calling the model `Image` would cause the
name of a reverse relation to clash.

We will not remove the built-in image model, as that would only force
developers to perform the complex migration that this RFC aims to
eliminate for future projects.

### Tweak the beginners tutorial, so it uses the custom image model

We will only make necessary tweaks so that it works with the new project
template. I don't think there's any need to describe the image model in
the beginners tutorial.

### Remove the "Custom image model" advanced topics doc

We could replace this with a reference document that describes the
methods that developers can override to tweak its behaviour.

### Stop calling it "Custom image model"

The documentation will be updated to no longer refer to "Custom Image
models" as an extra feature any more.
