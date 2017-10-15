# Front-End Language

Disclaimer: These are all work-in-progress ideas on what a strongly-typed HTML / CSS language might look like. I invite criticisms and the pointing out of flaws for this hypothetical language.

## Goals:

- A coupled replacement for Twig and SASS, combined into one compiler.
    - We are *not* thinking too hard about JavaScript integration with components yet.
- Fast and simple analysis of unused CSS selectors and optional optimization.
    - Optimization ideas:
        - Reduce CSS selector string sizes (where not marked for use in JavaScript)
            - ie. Transform ".Button_primary" to ".x45dt"
        - Completely remove unused rules from output and/or warn about them
- Easy to reuse pure components
- Can be used standalone (ie. generate static HTML) and can integrate seamlessly with other build systems (ie. Webpack, Laravel Mix, Brunch)
- (Maybe) Define and detect browser bugs at compile-time.
    - ie. If detect "display: inline-block;" requires a hack for IE6/IE7.

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

To avoid constantly increasing scope and never properly finish implementing features, the following link 
shows my plan for features and what version numbers they'll be at.

This will be updated as I go.

[Roadmap](ROADMAP.md)

## Rules
- Keywords: (These can be escaped with a \\, ie, if you want a HTML element called "if", do \\if)
    - import
    - Control flow:
        - if
        - for (do the Golang thing, one keyword for looping)
            - This will probably be in line with Silverstripe/Twig loops, which implicitly
              uses the scope of the current item, but not sure. Might have something like:
              "for MyList {" wherein you access properties with "it.Name"

              Reasoning for this is so that when reading the code, you immediately know 
              if the code your reading is in a "for" loop context. Also means you dont need special "top" or "up"
              variables to access parent scopes.
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
            // NOTE(Jake): Tells the compiler to not prefix this class with a namespace
            //             ie. without this, output CSS will be ".Button_is-active" rather than ".is-active"
            //
            //             You might want this for specific classes to make interfacing with JavaScript easier.
            //             ie. 
            //
            namespace := false
        }
    }

    button(class="js-component-button "+kind+" "+width, type=type) {
        children // will insert child nodes here
    }

    // Example HTML Output
    // <button class="Button_primary Button_full" type="button" />
}
```

```c
Normalize :: css {
    // Imagine a typical normalize.css file here
    // See "css_files" config below for example of how this gets included.
}
```

## Templates

```c
html {
    head {
        meta(name="viewport", content="width=device-width")
        meta(http-equiv="Content-Type", content="text/html; charset=utf-8")
        meta(http-equiv="X-UA-Compatible", content="IE=edge,chrome=1")
        script(type="text/javascript") {
            // NOTE: Since JavaScript parsing isn't going to be supported by the compiler yet, just pass
            //       in a HereDoc string of the JavaScript.
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

The idea is that the compiler will be able to output the CSS and the views in the form of PHP templates, React component views, etc.

```
HTML :: language {
    support_interop: false
}

PHP :: language {
    support_interop: true
}

Silverstripe :: language {
    support_interop: true
}

JavaScript_React :: language {
    support_interop: true
}

Elm :: language {
    support_interop: true
}
```