# Roadmap

**Please Note:** None of the pre-alpha versions below are ready for public consumption, are potentially buggy and have lots of edges that haven't been ironed out yet.

## [v0.1](https://github.com/SilbinaryWolf/compiler-fel/tree/0.1)
- [x] Parse config.fel file
- [x] Parse a template `html` block.
- [x] Parse expressions in `html` block
    - ie. value := "test" + "other_string"
- [x] Generate HTML out to specified to `templateOutputDir`

## [v0.2](https://github.com/SilbinaryWolf/compiler-fel/tree/0.2)
- [x] Ensure lexing supports UTF-8 characters.
    - Need to research other languages UTF-8 support, see if any annoying characters are banned. ie. emoji
- [x] Update Scanner to store lexing errors (return token.EOF if hits one)

## [v0.3](https://github.com/SilbinaryWolf/compiler-fel/tree/0.3)
- [x] Add CSS support (lexer/parsing/output)

## [v0.4](https://github.com/SilbinaryWolf/compiler-fel/tree/0.4)
- [x] Add component support

## [v0.5](https://github.com/SilbinaryWolf/compiler-fel/tree/0.5)
- [x] Add basic optimization pass (ie. *some* CSS rules will be omitted if they do not exist)

## v0.6
- Change attribute storage of classes on HTMLNode to be a "classList", rework attributes.
- Add support for ":: css_rules" blocks (so you can disable namespacing on certain selectors, etc)
- Add more optimization support for CSS (improve attribute matching, tag name matching)

## v0.x
- Add CSS element and property checking based on configuration file
	- ie. if you use "font-family", you'll get no error, if you use "font-famlee", you will get "CSS property is not defined" error.
- Add if / for-loop support
- Re-write type system (current hardcoded ints, needs to be more flexible to allow for custom types)