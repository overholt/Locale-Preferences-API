# Proposal: Locale Preferences API

## Abstract

This document proposes an extension to HTML's `Navigator` interface to enable
dynamic localization of content. The idea is to expose to script the language
tags that represents the user's locale preferences (akin to the language tags
that are normally sent with HTTP's `Accept-Languages` header).

Also proposed is a "`languageschange`" event, so that scripts can be notified if
the user changes the ordering of their preferred locales.

## Problem we are trying to solve

In order to support dynamic localization of content on the client-side,
developers need to have access to the user's locale  preferences. In user
agents, this is generally represented as an ordered list  of [BCP47] language
tags, which is shared with servers through the `Accept-Languages` HTTP header.

Traditionally, to access this list of language tags developers need to query a
server to tell them what the browser's language preferences are set to (i.e., by
reflecting the `Accept-Languages` HTTP header - and usually stripping away the
"q" values).  This has led to the creation of various xhr-based hacks and
workarounds on the client side. See: [JavaScript for detecting browser language
preference](http://stackoverflow.com/questions/1043339/javascript-for-
detecting-browser-language-preference) .

There are a number of issues with this work-around:

 * because of the reliance on making a HTTP request, the values are not
immediately available to script.

 * because of the reliance on making a XHR-based request, this becomes
impractical if the user is not connected to the network.

 * because of the reliance on HTTP requests, it's not possible to immediately
know if the user's preferred language order has changes (even though it is
rare - FireFox applications rely on this to be able to maintain the UI localized
without needing to reboot the device).

To overcome these limitations, and solely in Mozilla's FirefoxOS, developers are
relying on a  proprietary 
[mozSettings API](https://developer.mozilla.org/en-US/docs/Web/API/window.navigator.mozSettings) 
to get notified when the user's locale preferences change.

In order to address the issues described above, and to move away from having to
rely on a proprietary solution, this document proposes the following extensions
to the [HTML]'s Navigator interface.

## Acquiring the end-user's locale preferences

The end-user's locale preferences represents the end-user's preferred languages
and regional settings, which are derived from the operating system or directly
from the user agent. As there are numerous ways a user agent can derive the end-
user's preferred languages and regional settings, the means by which those
values are derived are beyond the scope of this document and left up to the
implementation.
 
## Extensions to Navigator interface

```WebIDL
partial interface Navigator : EventTarget {
   readonly attribute DOMString[]  languages;
            attribute EventHandler onlanguageschange; 
}
```

Note: We've received feedback that TC39 is not in favor of API's using frozen
/read-only arrays. Alternatives to the above attribute are:

 1. `sequence<DOMString> getLanguages()` method - thought this has been
    internally criticized as being "javaish". 

 2.  Willfully violate WebIDL's ban on using sequences on attributes, and make
     `languages` just return `sequence<DOMString>`. 

## The `languages` attribute

When getting, the languages attribute returns a read only platform Array
[WebIDL] of valid language tags in canonical form [BCP47]. The array is ordered
from most preferred to least preferred, where the first item is the language tag
that represents the user's most preferred language.

## Event handlers

If the user updates their locale preferences in such a way that it would cause
the ordering of language tags change, then the user agent MUST perform the
following steps. These steps use the DOM manipulation task source as the task
source. 

1. Let lang list be the updated list of preferred locales.

2. Queue a task to perform the following:

2.1 If the first item of the lang list is not the same value as the value of
the 'navigator' object's `language` attribute, update the `navigator` object's `
attribute` to be the first item lang list.

2.2 Update the values of `navigator`'s `languages` attribute so they are the
same as those in lang list.

2.3 Fire a simple event named "`languageschange`" at the `navigator` object.

## Privacy considerations

As with navigator.language, there are privacy implications in exposing the
user's language preferences, as it can potentially be used to infer both the
physical location (to at least a country level) and potentially the user's
ethnic background (in those that choose have explicitly selected more than one
language preference). These values can also be exploited, together, with other
data to uniquely identify users.

However, these values are already shared with servers with every HTTP request,
thus this API does not exacerbate the finger-printing situation.

Regardless, implementors are encouraged to reflect the value of
navigator.language unless the user has explicitly indicated that the site in
question is not allowed access to the information.

## Known usability issues

It is envisioned that the primary purpose for this API will be to take a list of
language-tags supported by an application and compare it with the list of
language-tags that represent the user's locale preferences.

Because of the nature of language tags, working with language tags can be
notoriously difficult - particularly when comparing two lists for changes.

See: https://bugzilla.mozilla.org/show_bug.cgi?id=889616

To make this API useful in practice currently requires a supporting i18n library
(e.g., [Mozilla's L20n: Localization 2.0 library ](https://github.com/l20n/l20n.js)). 

To make it possible to use this API on its own, Mozilla is discussing with TC-39
the possibility of exposing the LookupSupportedLocales and
CanonicalizeLanguageTag abstract algorithms as part of an extension of
[Ecma-402].

## References

* [BCP47] [Tags for Identifying Languages](http://tools.ietf.org/html/bcp47)
* [Ecma-402] [ECMAScript® Internationalization API Specification ](http://www.ecma-international.org/ecma-402/1.0/ECMA-402.pdf)

## Related Mozilla bugs

The following bugs motivated Mozilla to put together this proposal. The use
cases are have mainly been driven by FirefoxOS, though they've also come up
else where (e.g., in with Firefox Extensions).

* [bug 889335 - navigator.languages](https://bugzilla.mozilla.org/show_bug.cgi?id=889335)
* [Bug 780953 - Add language change event](https://bugzilla.mozilla.org/show_bug.cgi?id=780953)
* [Bug 889617 - Provide API for user requested language fallback](https://bugzilla.mozilla.org/show_bug.cgi?id=889617)
* [Bug 288670 - Use intl.accept_languages to choose the locale for a package if the current locale is unavailable](https://bugzilla.mozilla.org/show_bug.cgi?id=288670) 
* [Bug 562648 - Prioritized locale list for fallback of strings or add-ons](language/translation fall-back; fallback is always en-US)]
