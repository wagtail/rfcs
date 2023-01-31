# RFC 83: wagtail_ready hook.

* RFC: 83
* Author: Darrel O'Pry
* Created: 2023-01-30
* Last Modified:

## Abstract

This feature helps eliminate the dependency of the proper operating of Wagtail on specific ordering of INSTALL_APPS. Wagtail builds a number of registries once Django fires AppConfig.ready(). For contributed app authors and site builders this leads to scenarios where an app needs to be broken up in multiple parts to get proper ordering of template resolution and bootstrapping. This RFCs proposes that wagtail implement a single App.ready() handler that then uses the existing wagtail hooks feature to fire off a series of bootstrap hooks allowing wagtail modules to act in a
more deterministic ordering.

I ran into a situation where this could help me recently. I've built an app that I will be deploying to a few sites. In my INSTALLED_APPS it comes after grapple so it can use the Graphene types it generates on AppConfig.ready, but before Wagtail so it's theme overrides will take precendence. In Wagtail 4.1.1 populate of the snippet registry was deferred to app.ready so when grapple and my app ran the snippets were not registers so no Graphene types were generated for the snippets and any relationships to the GrapheneSnippet types threw exceptions. I need to split up my app now in order for it to work and I need to be very mindful of the order of INSTALLED_APPS. I will also be constantly anxious that something like this is going to happen again.  Adding a deferred wagtail_ready would eliminate some of the dependence on the order of INSTALLED_APPS and maybe help prevent related issues.



## Specification

Wagtail.apps.WagtailAppConfig.ready() SHOULD be the only implementation of AppConfig.ready() in the Wagtail Distribution.

WagtailAppConfig.ready() will fire the following hooks in the order listed.

1. 'django_ready' - Let wagtail distribution apps the Django registry is fully populated so that they can build their own registries.. All wagtail core registries MUST be fully populated when 'django_ready' completes running.

2. 'wagtail_ready' - Let 3rd party wagtail apps know that core has completed all of it's setup and all registries, signal handlers, etc are ready to go.
