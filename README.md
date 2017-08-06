# Front-End Language

Disclaimer: These are all work-in-progress ideas on what a strongly-typed HTML / CSS language might look like. I invite criticisms and the pointing out of flaws for this hypothetical language.

## Goals:

- A coupled replacement for Twig and SASS, combined into one compiler.
    - We are *not* thinking too hard about JavaScript integration with components yet.
- Fast and simple analysis of unused CSS selectors and optional optimization.
    - Optimization ideas:
        - Reduce CSS selector string sizes (where not marked for use in JavaScript)
        - Completely remove unused rules from output and/or warn about them
- Easy to reuse components using a functional programming paradigm.
- Can be used standalone (ie. generate static HTML) and can integrate seamlessly with other build systems (ie. Brunch, Webpack)
- Define and detect browser bugs at compile-time.
    - ie. If you're targetting IE8, it'll warn about CSS

## Parsing Logic:
- Semicolons (;) optional. This will be achieved by making newlines act as a statement terminator. (Like Golang)
- Colons (:) used to mean 'declare' and colon equals (:=) means infer type from right-hand expression. (Like Golang)
- Parameters can be seperated by , or \n, but not both at the same time.
- Strings can be represented with: 
    - " (regular strings)
    - ' (will either be unused, or work like C/Golang, ie. single character byte)
    - ` (for style/script data or inline HTML)
- Identifiers declared with $ are backend interopable variables / functions.
    - ie. $get_posts() in PHP will output `<?php get_posts(); ?>`
    - ie. $this.props.value in JavaScript will output `this.props.value`

## Roadmap plan:

To avoid constantly increasing scope and never properly finish implementing features, the following link 
shows my plan for features and what version numbers they'll be at.

This will be updated as I go.

[Roadmap](ROADMAP.md)

## Rules
- A component must begin with a captial letter.
    - Reasoning: Make it clear when using a regular tag vs a component, ie. "div" vs "Div"
- Keywords: (These can be escaped with a \\, ie, if you want a HTML element called "if", do \\if)
    - Top Level or Component
        - style (also a tagName)
        - html (also a tagName)
        - script (also a tagName)
        - styleflags
    - Top Level 
        - import
    - Component
        - properties
    - HTML Block:
        - children
    - Control flow:
        - if
        - for (do the Golang thing, one keyword for looping)
            - This will probably be in line with Silverstripe/Twig loops, which implicitly
              uses the scope of the current item.
    - Browser Def
        - vendorprefix
    - Types:
        - string
        - enum
        - int (64-bit integer)
        - float (64-bit floating point, a double)
        - bool
            - false
            - true
        - type\[\] (Array of type)
        - interop
            - this is essentially a $ variable that can't be known about 
              at compile time. Unfortunately a "mixed" type. 

## Reusable Components

```cpp
Button : component {
    Kind : enum {
        primary: "primary"
        secondary: "secondary"
    }

    Type : enum {
        button: "button"
        submit: "submit"
        reset: "reset"
    }

    Width : enum {
        auto
        full: "full-width"
    }
    
    // NOTE: properties default to first item in enum, unless explicitly set.
    properties {
        // NOTE(Jake): Might add a "private"/"readonly"/"unsupported" block so you can declare some properties
        //             as unsupported. This will mean users *can* change the values but it'll cause a compile 
        //             warning saying "[x] property is unsupported and future updates might break this component"
        namespace := 'btn' // (optional, will default to component name, ie. "Button")
        primaryColor := "red" // NOTE: Might want to make a `Colors` enum for standard lib.

        kind: Kind
        type: Type
        width: Width = full
    }

    style {
        // NOTE(Jake): Access component properties in a CSS4 variables way.
        var(--namespace) {

        }

        .primary {
            // ...
        }

        .secondary {
            // ...
        }

        .primary:hover,
        .primary:focus,
        .primary.is-active {
            // ...
        }
    }

    // Idea: Add special compile hints to CSS selectors, one idea might be to
    //       restrict CSS properties to a list of enumerated values to catch improperly
    //       set values.
    styleflags {
        .is-active {
            // NOTE(Jake): Tells the compiler to not optimize this out or change it's name. This is so
            //             the classname can be used in JavaScript and so the CSS rules aren't optimized out 
            //             if unused.
            dontModify := true
        }
    }

    html {
        classes := []
        classes[] = namespace
        classes[] = kind
        classes[] = width

        // NOTE: Implicitly explode array into string for "class" property.
        //       Might be exposed as function so you can explictily do "array.toClassString()"
        button(class = classes, type=type) {
            children // will insert child nodes here
        }
    }
}
```

```cpp
Normalize : component {
    properties {
        namespace := '' // do not namespace CSS selectors as this is a global
                        // stylesheet
    }

    style {
        // Imagine a typical normalize.css file here
    }
}
```

## Templates

```cpp
html {
    head {
        meta(name="viewport", content="width=device-width")
        meta(http-equiv="Content-Type", content="text/html; charset=utf-8")
        meta(http-equiv="X-UA-Compatible", content="IE=edge,chrome=1")
        script(type="text/javascript") {`
            var _gaq = _gaq || [];
            _gaq.push(['_setAccount', 'UA-XXXXX-X']);
            _gaq.push(['_trackPageview']);

            (function() {
            var ga = document.createElement('script'); ga.type = 'text/javascript'; ga.async = true;
            ga.src = ('https:' == document.location.protocol ? 'https://ssl' : 'http://www') + '.google-analytics.com/ga.js';
            var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(ga, s);
            })();
        `}
    }
    body {
        footerWidths := Button.Width.fullWidth

        div {
            Button(kind=primary, type=reset) {
                "Clear"
            }
            // NOTE(Jake): Because these params are for a `Button` component, you don't need 
            //             to do "Button.Kind.secondary" (though you could), you can just do "Kind.secondary"
            Button(kind=Kind.secondary, width=Width.fullWidth, type=Type.submit){
                "Submit"
            }
        }
        footer {
            Button(width=footerWidths) {
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
        - /components/
        - /templates/
        - config.fel
    - /css/
    - /templates/
```

## Config

```cpp
import (
    Normalize
)

// Idea: You can put a heap of Bootstrap components in a 'Bootstrap' folder
//       which will automatically namespace each object so you have to do "Bootstrap.Button".
//
//       However, I feel you generally just want to be able to use any component in your project without
//       worrying about namespaces for the most part, so not sure how necessary this would be.
//
namespaceDirectory := 'modules'

cssOutputDirectory := '../css'

// Idea: You can target a backend language to output to
backendLanguage := Language.PHP // ie. Language.HTML, Language.JavaScript

// NOTE(Jake): Where to output HTML / PHP / JavaScript, depending on `backendLanguage`
//             This is a 1-1 mapping, so if you made "Page.fel" it would output in /templates/Page.php
templateDirectory := {
    "templates": "../templates"
}

// Idea: Inspired by Brunch, declare where you want CSS of specific modules/components to be output to.
//
//       If you are actually using another build system with this like Webpack/Brunch, perhaps the CSS output
//       can be configured at that level.
//
cssFiles := [
    "normalize.css": [
        Normalize
    ],
    "main.css": [] // empty array implies put in all CSS that isn't placed elsewhere.
]

// Idea: Define target browsers so 'stylerules' can be applied to warn of browser bugs as well as generate
//       vendor prefixes for properties.
Browser := {
    InternetExplorer: 9,
    Edge: 
}
Browser.InternetExplorer = 9
Browser.Edge = 14
Browser.Safari = 8
Browser.SafariIOS = 10
Browser.Chrome = 60
Browser.ChromeAndroid = 59
Browser.Opera
Browser.Firefox = 54
Browser.Firefox = 54
```

## Browser Rules / Vendor Prefixes
```
if Browser.Safari < 9 {
    // NOTE(Jake): Declare vendor prefixes based on browser being targetted
    vendorprefix {
        transition: -webkit-transition
    }
}
```

## Language Definition

The language will come with a few standard language definition files to allow configuration of backend interoperation.

It's unclear how this should look so far so to begin with language support will probably be hardcoded into the code generation
part of the compiler.

```
HTML :: language {
    supportInterop: false
}

PHP :: language {
    supportInterop: true
}

JavaScript :: language {
    supportInterop: true
}
```