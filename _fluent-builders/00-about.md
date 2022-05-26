---
masthead_title: "Fluent Builders"
title: "About Fluent Builders"
permalink: /fluent-builders/about
redirect_from:
    - /fluent-builders/
---

## What Is It?
This is a simple library for creating builders for types used within unit tests; it is not recommended to use this library within production code due to relying heavily on reflection for its functionality and as a result its ability to circumvent most member security restrictions.

## Why Use It?
Fluent Builders was originally designed for testers, or people otherwise without much coding experience, to write C# based UI Automation tests. This means that the interfaces used for interacting with builders are kept very simple to the point that the functionality is largely discoverable simply by using intellisense.
We've even gone so far as to hide the standard System.Object members from view; so long as your IDE configuration does not ignore the `EditorBrowsable(EditorBrowsableState.Never)` configuration attribute.