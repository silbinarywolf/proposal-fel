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

## [v0.6](https://github.com/SilbinaryWolf/compiler-fel/tree/0.6)
- [x] Add support for ":: css_rules" blocks (so you can disable namespacing on certain selectors, etc)
- [x] Add more optimization support for CSS (improve attribute matching, tag name matching)
- [x] Add tests for css optimization + setup Travis CI

## v0.7
- Add if / for-loop support

## v0.x
- Re-write type system 
	- Currently hardcoded enum/ints, needs to be more flexible to allow for custom types, ie. map[string]TypeInfo
- Improve `data/html_query.go` HasSelectorPartMatch() function.
	- Add support for more attribute operators (update `parser/typecheck.go` to enforce only supported attribute operators)
- Update data.HTMLNode to be 1-1 with JavaScript property names.
	- https://developer.mozilla.org/en-US/docs/Web/API/Element
		- className
		- nextElementSibling
	- For data attributes, store as property `dataset`. (maps to JavaScript)
	- For undefined attributes, store them in the Attributes map
		- We'll probably enforce checking if attributes exist by default based on a hardcoded list (that'll be updated to be configurable later.)
- Namespacing tag names with HTML/CSS.
	- Currently this will not automatically replace an "a { color:red; }" css rule with ".MyComponent__a-tag" or anything on the
	  HTML/CSS side of things. We probably want to do this.
- Add CSS element and property checking, initially hardcoded (to be eventually replaced so users can define the properties if need-be)
	- ie. if you use "font-family", you'll get no error, if you use "font-famlee", you will get "CSS property is not defined" error.
