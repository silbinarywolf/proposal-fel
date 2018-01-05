# Front-End Language

Disclaimer: These are all work-in-progress ideas on what a strongly-typed HTML / CSS language might look like. I invite criticisms and the pointing out of flaws for this hypothetical language.

## Goals:

- Blazing fast and simple analysis of unused CSS selectors and **maybe** optional classname optimization.
    - Optimization ideas:
        - Completely remove unused CSS rules from output and/or warn about them
            - ie. If the element `<abbr>` is unused in the project and global CSS styling exists for `abbr {}`
                  it will be removed.
        - Reduce or hash CSS selector string sizes (where not marked for use in JavaScript or other frontend code)
            - ie. Transform ".Button_primary" to ".x45dt". This will reduce CSS filesize and potentially alleviate browser string allocations / comparisons.
- A coupled replacement for template engines (Twig, Pug) and CSS preprocessors (SASS, LESS, Stylus), combined into one compiler.
    - We are *not* thinking too hard about JavaScript integration with components yet. I have explored embedding [Goja](https://github.com/dop251/goja) and parsing JavaScript with it, so handling `:: javascript` definitions isn't out of the question. The path that will most likely be taken is outputting components as stateless views for React / Vue.js / similar.
- Easy to reuse pure components
- Can be used standalone (ie. generate static HTML) and can integrate seamlessly with other build systems (ie. Webpack, Laravel Mix, Brunch)
- (Maybe) Define and detect browser bugs at compile-time.
    - IE11    - Flex container not center nested elements if it has a min-height. User could be warned of this and told of workarounds via the compiler. (ie. wrap the container again) (source: https://github.com/philipwalton/flexbugs/issues/64)
    - IE6/IE7 - Auto apply patch for "display: inline-block;". (ie. "*display: inline; zoom: 1;") (source: https://stackoverflow.com/questions/6544852/ie7-does-not-understand-display-inline-block)

## Parsing Logic:
- Semicolons (;) optional. This will be achieved by making newlines act as a statement terminator. (Like Golang)
- Colons (:) used to mean 'declare' and colon equals (:=) means infer type from right-hand expression. (Like Golang)
- Strings can be represented with: 
    - " (regular strings)
    - """ (for heredoc strings, ie. style/script data or inline HTML)
    - ' (banned except for in CSS for now)
- Identifiers declared with $ are backend interopable variables / functions.
    - ie. $get_posts() in PHP will output `<?php get_posts(); ?>`
    - ie. $this.props.value in JavaScript will output `this.props.value`

## Roadmap plan:

To avoid constantly increasing scope and never finishing features, the following link 
shows my plan for features and what version numbers they'll be at.

This will be updated as I go.

[Roadmap](ROADMAP.md)

## Rules
- Keywords: (These can be escaped with a \\, ie, if you want a HTML element called "if", do \\if)
    - import
    - Control flow:
        - if
        - for (do the Golang thing, one keyword for looping)
    - Types:
        - string
        - enum
        - int (defaults to highest integer type for that operating system, most likely 64-bit)
        - float (defaults to highest floating point type for that operating system, most likely 64-bit)
        - bool
            - false
            - true
        - type\[\] (Array of type)
        - interop
            - this is essentially a $ variable that can't be known about 
              at compile time. Unfortunately a "mixed"/"any" type.

## Conventions
- A component should begin with a captial letter
    - Reasoning: Make it clearer when using a regular tag vs a component, ie. "button" vs "Button"
                 and less likely to clash with potential future elements.

## Reusable Components

```c
Button :: html {
    // NOTE: properties default to first item in enum, unless explicitly set.
    :: struct {
        kind: enum {
            "primary"
            "secondary"
        }
        type: enum {
            "button"
            "submit"
            "reset"
        }
        width: enum {
            "auto"
            "full"
        } = "full"
    }

    // NOTE: Currently it does not have access to `:: struct` variables as I'm not sure adding
    //       that complexity is a good idea yet.
    //
    //       Semicolons aren't required to end property statements currently, but that feature
    //       has added a bit of complexity to the CSS parser, so not sure if it's worth it.
    //
    :: css {
        test := "red"

        .primary {
            color: test
        }

        .secondary {
            // ...
        }

        .primary:hover,
        .primary:focus,
        .primary.is-active, {
            // ...
        }
    }

    // Idea: Tell the compiler to do certain operations / etc based on classnames.
    :: css_config {
        .js-component-button,
        .is-active {
            // NOTE(Jake): Tells the compiler to not prefix this class with a namespace or optimize the rule
            //             away if it's unused at compile-time.
            //
            //             ie. without this, output CSS will be ".Button__is-active" rather than ".is-active"
            //
            //             You might want this for specific classes to make interfacing with JavaScript easier.
            //             ie. jQuery(".Button").addClass("is-active")
            //
            modify := false
        }
    }

    button(class="js-component-button "+kind+" "+width, type=type) {
        children // will insert child nodes here
    }

    // Example HTML Output
    // <button class="Button_primary Button_full" type="button"></button>
}
```

```c
Normalize :: css {
    // Imagine a typical normalize.css file here
    // See "css_files" config below for example of how this gets included.
}
```

```c
Link :: html {
    :: struct {
        url: string
    }

    // NOTE: Only have wrapping <a> tag if url is a non-empty string.
    a(href=url) if url {
        children
    }
}

Link :: html {
    :: struct {
        url: string
    }

    // NOTE: Will have same HTML output as above Link example.
    if url {
        a(href=url) {
            children
        }
    } else {
        children
    }
}
```

```c
FontAwesomeIconName :: enum {
    // ... long list of icon names ...
    "facebook"
    "twitter"
    // ... more icon names ...
}

SocialLink :: struct {
    title: string
    url: string
    icon: FontAwesomeIconName
}

socialLinks := []SocialLink{
    {
        title: "Facebook",
        url: "www.facebook.com",
        icon: "facebook",
    },
    {
        title: "Twitter",
        url: "www.twitter.com",
        icon: "facebook",
    }
}

html {
    head{
        // ...
    }
    body {
        // Reasoning: 1) Doesn't change the scope so you need to use "Top"/"Up" variables
        //               like most programming languages.
        //
        //            2) When scanning/skimming code, spotting an "it" will immediately 
        //               hint to you that you're reading code that is in a for-loop.
        //

        // Iterator Loop
        for link := socialLinks {
            Link(url=link.url) {
                i(class="fa fa-"+link.icon)
                link.title // Print `title`
            }
        }
        // Iterator Loop, explicit index (i, zero-indexed) and iterator (socialLink) value
        for i, link := socialLinks {
            // NOTE: I do like template languages with "first" and "last", but
            //       I'm not convinced the syntactic-sugar is that helpful, and by forcing
            //       users to be explicit like below, over and over, it'll help solidify their
            //       knowledge of working with zero-based arrays.
            if i == 0 || i == len(socialLinks)-1 {
                div(class="clearfix"){}
            }
            Link(url=link.url) {
                i(class="fa fa-"+link.icon)
                link.title // Print `title`
            }
        }
        // Traditional Loop
        for i := 0; i < len(socialLinks); i++ {
            link := socialLinks[i]
            Link(url=link.url) {
                i(class="fa fa-"+link.icon)
                link.title // Print `title`
            }
        }
        // Not sure if anyone would *ever* need a while(true)-like loop for templates.
        for true {
            Link(url="www.google.com") {
            }
        }
    }
}
```

## Component output to TypeScript

This is simple example of how component outputting might work. 
The idea is your view components can be output to the language / virtual-dom framework of your choice.

```c
/**
 * This was automatically generated by FEL compiler.
 * Do not edit.
 */

import * as React from 'react';

interface Props {
    url: string
    children: React.ReactNode
}

export function Link(props: Props) {
    // NOTE(Jake): 2018-01-05
    //
    // The code below will most likely generate "React.createElement" calls 
    // rather than JSX/HTML and track a result node variable ("n")
    // so "wrapped element" if-statements can be handled without
    // code duplication.
    //
    // tl;dr: This output wont be as readable but file size / performance should
    //        be optimal.
    //
    if (url) {
        return (
            <a href={url}>
                { this.children }
            </a>
        )
    }
    return this.children
}
```

## Templates

These are files that can be output 1-1 in the folder of the developers choosing.
The use case for this functionality is SilverStripe, Wordpress and Drupal templates.

```c
html {
    head {
        meta(name="viewport", content="width=device-width")
        meta(http-equiv="Content-Type", content="text/html; charset=utf-8")
        meta(http-equiv="X-UA-Compatible", content="IE=edge,chrome=1")
        script(type="text/javascript") {
            // NOTE: Since JavaScript parsing isn't going to be supported by the compiler yet, just pass
            //       in a HereDoc string of the JavaScript.
            //
            //       We are using """ (Python-style) instead of ` (Golang-style) because JavaScript/ES6 is using `
            //       for template literals: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals
            //
            //       It's also way less likely to collide with a real giant chunk of raw text. I can't imagine any real-world
            //       case where someone uses """.
            //
            """
            var _gaq = _gaq || [];
            _gaq.push(['_setAccount', 'UA-XXXXX-X']);
            _gaq.push(['_trackPageview']);

            (function() {
            var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
            ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
            var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
            })();
            """
        }
    }
    body {
        footer_widths := "full"

        div {
            Button(kind=primary, type=reset) { 
                "Clear" 
            }
            Button(kind="secondary", width="full", type="submit"){
                "Submit"
            }
        }
        footer {
            Button(width=footer_widths) {
                "Open Something Cool!"
            }
        }
    }
}
```

## Example code structure in directory

```
- /project_folder/
    - /fel/
        - /includes/
        - /templates/
        - config.fel
    - /css/
    - /templates/
```

## Config

```c
// Idea: You can put a heap of Bootstrap components in a 'Bootstrap' folder
//       which will automatically namespace each object so you have to do "Bootstrap.Button".
//
//       However, I feel you generally just want to be able to use any component in your project without
//       worrying about namespaces for the most part, so not sure how necessary this would be.
//
namespace_directory := 'modules'

css_output_directory := '../css'

// Idea: Define a directory to output components as React/Vue.js/Elm stateless/pure components
//       so they can be used in web apps.
component_view_language = 'typescript'
component_view_framework = 'react'
component_view_output_directory = '../javascript/views'

// Idea: You can target a backend language to output to
backend_language := Language.PHP // ie. Language.HTML, Language.JavaScript

// NOTE(Jake): Where to output HTML / PHP / JavaScript, depending on `backendLanguage`
//             This is a 1-1 mapping, so if you made "Page.fel" it would output in /templates/Page.php
template_directory := {
    "templates": "../templates"
}

// Idea: Inspired by Brunch, declare where you want CSS of specific modules/components to be output to.
//
//       If you are actually using another build system with this like Webpack/Brunch, perhaps the CSS output
//       can be configured at that level.
//
css_files := [
    "normalize.css": [
        Normalize
    ],
    "main.css": [] // empty array implies put in all CSS that isn't placed elsewhere.
]

// Idea: Define target browsers so 'css_rules' can be applied to warn of browser bugs as well as generate
//       vendor prefixes for properties.
Browser := {
    Internet_Explorer: 9,
    Edge: 14,
    Safari: 8,
    Safari_IOS: 10,
    Chrome: 60,
    Chrome_Android: 59,
    Opera: 30,
    Firefox: 54,
}
```

## Browser Rules / Vendor Prefixes
```
if Browser.Safari < 9 {
    // NOTE(Jake): Declare vendor prefixes based on browser being targetted
    :: css_vendor_prefix {
        transition: -webkit-transition
    }
}
```

## Language Definition

The language will come with a few standard language definition files to allow configuration of backend interoperation.

It's unclear how this should look so far so to begin with language support will probably be hardcoded into the code generation
part of the compiler.

The idea is that the compiler will be able to output the views in the form of PHP templates, React component views, etc.
I've put together a simplistic idea of how it might work below, but this will need to be battle-tested against real languages.

```
HTML :: language {
    support_interop: false
}

PHP :: language {
    support_interop: true
    if_begin: "if (%conditional) {"
    if_end: "}"
    for_begin: "for (%begin; %conditional; %statement) {"
    for_end: "}"
    for_it_begin: "foreach (%array as %it) {"
    for_it_end: "}"
    for_index_and_it_begin: "foreach (%array as %index => %it) {"
    for_index_and_it_begin: "}"
    for_continue: "continue;"
    for_break: "break;"
}

Silverstripe :: language {
    support_interop: true
    if_begin: "<% if %conditional %>"
    if_end: "<% end_if %>"
    //
    // No equivalents for <% loop %>.
    //
    // Might just target PHP / Twig, then write a Silverstripe-Twig template
    // module.
    //
    // Alternatively, extend Silverstripe template language to allow for-loop
    //
    for_begin: ""
    for_end: ""
    for_it_begin: ""
    for_it_end: ""
    for_index_and_it_begin: ""
    for_index_and_it_begin: ""
    for_continue: ""
    for_break: ""
}

JavaScript_React :: language {
    support_interop: true
    if_begin: "{ !!(%conditional) &&"
    if_end: "}"
    for_begin: "for (%begin; %conditional; %statement) {"
    for_end: "}"
    // NOTE: This will probably end up generating "for (%it of $array)" instead in
    //       an idiomatic way so you can control the loop with "continue"/"break"
    for_it_begin: "{ %array.map((%it) => {"
    for_it_end: "}) }"
    for_index_and_it_begin: "{ %array.map((%it, %index) => {"
    for_index_and_it_begin: "}) }"
    for_continue: ""
    for_break: ""
}

Elm :: language {
    support_interop: true
}
```