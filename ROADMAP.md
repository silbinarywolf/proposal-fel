# Roadmap

## v0.1 - Done
- [x] Parse config.fel file
- [x] Parse a template `html` block.
- [x] Parse expressions in `html` block
    - ie. value := "test" + "other_string"
- [x] Generate HTML out to specified to `templateOutputDir`

## v0.2
- [x] Ensure lexing supports UTF-8 characters.
    - Need to research other languages UTF-8 support, see if any annoying characters are banned. ie. emoji
- [x] Update Scanner to store lexing errors (return token.EOF if hits one)

## v0.3
- Add CSS support (lexer/parsing/output)

## v0.4 
- Add component support

## v0.5
- Add basic optimization pass (ie. CSS rules will be omitted if they do not exist)

## v0.x
- Add CSS element and property checking based on configuration file
	- ie. if you use "font-family", you'll get no error, if you use "font-famlee", you will get "CSS property is not defined" error.

