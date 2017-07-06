# Front-End Language

Disclaimer: These are all work-in-progress ideas on what a strongly-typed HTML / CSS language might look like. I invite criticisms and the pointing out of flaws for this hypothetical language.

## Goals:

- Fast and simple analysis of unused CSS selectors and optional optimization.
    - Optimization ideas:
        - Reduce CSS selector string sizes (where not marked for use in JavaScript)
        - Completely remove unused rules from output and/or warn about them
- Easy to reuse components using a functional programming paradigm.
- Can be used standalone (ie. generate static HTML) and can integrate seamlessly with other engines / templating code.
- Define and detect browser bugs at compile-time.

## Parsing Logic:
- Semicolons (;) optional. This will be achieved by making newlines act as a statement terminator. (Like Golang)
- Colons (:) used to mean 'declare' and colon equals (:=) means infer type from right-hand expression. (Like Golang)
- Parameters can be seperated by , or \n, but not both at the same time.
- Strings can be represented with: 
    - " (regular strings)
    - ' (will either be unused, or work like C/Golang, ie. single character byte)
    - ` (for style/script data or inline HTML)
- Variables are prefixed with $.
    - This decision is to avoid name clashes with constants/enums and variables.

## Rules
- A component must begin with a captial letter.
    - Reasoning: Make it clear when using a regular tag vs a component, ie. "div" vs "Div"
- Keywords: (These can be escaped with a \\, ie, if you want a HTML element called "if", do \\if)
    - Top Level or Def
        - style (also a tagName)
        - html (also a tagName)
        - script (also a tagName)
        - stylerules
        - config 
    - Top Level 
        - import
    - Def
        - properties
    - HTML Block:
        - children
    - Control flow:
        - if
        - for (do the Golang thins, one keyword for looping)
    - Types:
        - string
        - enum
        - int (64-bit integer)
        - float (64-bit floating point, a double)
        - bool
            - false
            - true
        - type\[\] (Array of type)

## Reusable Components

```cpp
Button : component {
    config {
        namespace := 'btn' // (optional, will default to component name, ie. "Button")
        // NOTE: Might want to make a `Colors` enum for standard lib.
        primaryColor := "red"
    }

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
        $kind: Kind
        $type: Type
        $width: Width = full
    }

    style {`
        .$config.namespace {

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
    `}

    // Idea: Add special compile hints to CSS selectors, one idea might be to
    //       restrict CSS properties to a list of enumerated values to catch improperly
    //       set values.
    stylerules {
        .is-active {
            // Idea: This will keep the selector name in-tact and 
            //       not attempt deadcode elimination / throw warnings.
            //       A better name might be "alwaysKeep" or "usedExternally".
            inJavaScript := true
        }
    }

    html {
        $classes := []
        $classes[] = $config.namespace
        $classes[] = $kind
        $classes[] = $width

        // NOTE: Implicitly explode array into string for "class" property.
        //       Might be exposed as function so you can explictily do "array.toClassString()"
        button(class = $classes, type=$type) {
            children // will insert child nodes here
        }
    }
}
```

```cpp
Normalize : component {
    config {
        namespace := '' // do not namespace CSS selectors
    }

    // Imagine a typical normalize.css file here
    style {`
    `}
}
```

## Templates

```cpp
html {
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
            $footerWidths := Button.Width.fullWidth

            div {
                Button(kind=primary, type=reset) {
                    "Clear"
                }
                Button(kind=secondary, width=fullWidth, type=submit){
                    "Submit"
                }
            }
            footer {
                Button(width=$footerWidths) {
                    "Open Something Cool!"
                }
            }
        }
    }
}
```

## Config

```cpp

import (
    Normalize
)

config {
    // Idea: You can put a heap of Bootstrap components in a 'Bootstrap' folder
    //       which will automatically namespace each object so you have to do "Bootstrap.Button".
    //
    //       However, I feel you generally just want to be able to use any component in your project without
    //       worrying about namespaces for the most part, so not sure how necessary this would be.
    //
    namespaceDir := 'modules'

    cssOutputDir := 'app/css/'

    // Idea: Inspired by Brunch, declare where you want CSS of specific modules/components to be output to.
    cssFiles := [
        'normalize.css' := [
            Normalize
        ],
        'main.css' := [] // empty array implies put in all CSS that isn't placed elsewhere.
    ]

    // Idea: Define target browsers so 'stylerules' can be applied to warn of browser bugs as well as generate
    //       vendor prefixes for properties.
    targetBrowsers := Browser.IE >= 10 && Browser.Safari > 8

    // Idea: CSS preprocessor middleware, this will:
    //          - Naively replace "$config.namespace" with your namespace string. ie. .$config.namespace {} becomes .btn {}
    //          - Run it through an external library or executable.
    //          - Take output CSS and parse into AST for FEL, so it can deadcode check, etc. 
    //
    //       This idea might be completely moot if we just make the FEL CSS language really good on its own merits.
    //       Problems with this include:
    //          - Can't easily redistribute a component that requires SASS/LESS/Stylus/etc.
    //          - Not sure how to pass configuration values to compiler SASS. ie. I can't configure a Bootstrap buttons primary color.
    //          - Slow down compilation time.
    //          - Unable to properly handle error messages, could cause JavaScript Transpilation-levels of obtuse-errors.
    //
    cssMiddleware := [
        sass_compiler
    ]
}

```
