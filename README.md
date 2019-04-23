# Front-End Language Reliability Tool

**Disclaimer:** This intent of this document is to discuss ways to improve upon existing build tools in a pragmatic but positive manner. If anything is unclear or if you have any questions, please raise them as a Github issue.

## What are the primary problems we want to solve?

The loose decoupled nature of HTML and CSS makes it difficult to safely remove unused rules and determine what rules may conflict with each other. Not only that, but various browsers vendors such as Internet Explorer and Safari either lag behind the latest standards by half a decade or implement a feature erroneously, which pushes additional development effort to the website or web app developer.

We don't only want this tool to solve the above problems, but solve them well. This means a few things:

- It needs to be blazing fast. CSS preprocessors like SASS aren't very complex but seem to compile significantly slower than other build tools like the TypeScript compiler. For example, on a Windows machine, SASS might take 4 seconds to recompile in "watch mode" for a reasonably large project using a Bootstrap-sized framework.
- It needs to be 100% backwards compatible. If it's installed globally on a developers machine, it should be easy to install the next version (eg. 1.1) but also be able to run the tool at an older version (eg. 1.0). The Go programming language claims to offer this feature using a ["-lang" flag](https://golang.org/doc/go1.12#compiler).
- The tool should not rely on the user having Python, NodeJS or any other third-party dependency installed, it should work standalone. However, it should also be easy to integrate with existing build systems such as Webpack. We might want the Webpack version to be a JavaScript / WASM build of the tool as building native binaries like NodeSASS can cause problems with build servers and compose poorly with other tools like [Cypress](https://www.cypress.io/).
- It should ideally always have at least 2-3 deeply invested maintainers so that the project isn't halted by a single author leaving the project.

## Who do we want to solve these problems for?

We want to solve this problem in a way that will be a good fit for both large JavaScript build tool projects as well as for a time and budget-constrained work.

If you aren't aware, to get an idea of the time constraints some web developers are under, especially beginners, Im talking about projects that have about 45-90 hours of _total_ budget. That means a CMS backend with modules or plugins needs to be setup, the frontend needs to be built and then finally all that code needs to be put on a remote server in 2 weeks.

Another consideration is quick quoted work that can take anywhere between 1 to 6 hours. If you quote a client with 6 hours of work, then you most likely aren't going to be able to prioritize HTML and CSS cleanliness if you underestimated the effort. However, having a tool that just tells you what's unused or that could just cleanup unused rules for you, would save a lot of time and improve readability.

## What are some existing solutions to these problems?

Before considering building yet another tool, we should look around and see if there's anything out there that will be able to solve the problems we have.

One common thing I've listed as a "Con" below is the reliance on a JavaScript build tools. The reason for this is that in my experience when doing budget-constrained work, you don't have time to deal with the inertia and unreliability that comes from using these tools.

### Determine what CSS rules are actually used by a project

Here are a list of tools or resources that help assist you in finding what CSS rules are being used.

#### Chrome DevTools Audit

A tool built-in to the Chrome web browser which can help you discover unused CSS styles.

**Pros**

- 100% accuracy

**Cons:**

- Requires loading each page and all its possible variants, ie. if a client can modify a banner to have different styling, that variant needs to be tested. This takes time!

#### Typed CSS modules or similar

Tools that integrate with a JavaScript Build System and generate identifiers so that when a CSS rule is removed, a variable will become undefined thats tied to that CSS rule.

**Implementations:**

- [https://github.com/seek-oss/css-modules-typescript-loader](https://github.com/seek-oss/css-modules-typescript-loader)
- [https://github.com/Quramy/typed-css-modules](https://github.com/Quramy/typed-css-modules)

**Pros:**

- Generate constants from CSS files so that your JavaScript build will fail if you're attempting to use a rule that was removed.

**Cons:**

- Assumes you're building an application or using a build tool. Not helpful for those with legacy HTML/CSS code or those building Wordpress sites.

#### PurifyCSS

**Pros:**

- Can process HTML, JS and PHP to see what classes are used/unused.

**Cons:**

- [Project is no longer maintained.](https://github.com/purifycss/purifycss/issues/213)
- Assumes you're using a build tool like Gulp, Grunt or Webpack. Would require the user have Node installed.
- Requires manually specifying the files you want to scan. Ideally, we want something that will "just work".

### Stop CSS rules from conflicting

Here are a list of practices or tools that were designed to stop CSS classnames from conflicting with each other and that also keep [CSS specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity) low, which is helps reduce cognitive complexity.

#### CSS Modules

**Pros**

- Stops CSS conflicting by namespacing
- Keep CSS specificity low
- Not a process. The tool automatically namespaces your rules for you, less room for error.

**Cons**

-  Requires a JavaScript build tool like Webpack

#### [BEM Convention](http://getbem.com/)

**Pros**

- Stops CSS conflicting by namespacing
- Keep CSS specificity low

**Cons**

- Unintuitive and hard to understand. We base this on personal experience where  either a client has provided frontend where BEM wasn't followed properly or where teams with not much experience in frontend get it wrong. We don't see this as a failing of developer, but rather the unintuitive aesthetic of BEM itself.

### What HTML patterns or CSS features have erroneous behaviour in certain browsers

Here are a list of tools or resources that help assist you in finding what HTML is valid and what CSS rules are supported.

#### W3C Markup Validation Service

A [website](https://validator.w3.org/) that tells you whether your HTML is correct or not.

**Pros**

- Tells you whether your HTML is valid or not. 
- Historically, this was very good at helping you figure out why Internet Explorer 6, 7 or 8 wasn't working, usually due to a unclosed or incorrectly closed tag.

**Cons**

- Not built-in. Requires you provide a live URL or paste HTML per-page.

#### CanIUse

A website that will tell you what features are either partially or fully supported in a browser.

**Pros**

- Informs you of partial implementations that may differ or have buggy behaviour when compared to the W3C spec.

**Cons**

- Not a built-in tool, which makes it easy to accidentally use a bleeding edge CSS feature without realizing.
  - **Side Note:** Manual testing in your target browsers would help avoid this but unfortunately we have rarely seen developers test in those browsers either due to lack of physical ownership (iPhone), lack of knowledge on how to test older Internet Explorer in a VM or laziness.

## How do we solve this problem

Now that we've defined at least some problems, let's looks at some things that our tool will need to be able to solve.

### Nested CSS

In a legacy HTML or CSS project, nested CSS is definitely going to exist in the project. By nested CSS, we mean a selector such as ".myElement .myOtherElement". Due to the nature of various legacy systems and also with how Wordpress works, this tool is going to need to be able to reason about how templates include other templates so that it is able to build an accurate model of the varying HTML structures to see if that CSS rule is used or not.

This becomes more complicated as different backend configurations could completely change what CSS classes get rendered. If a user has a website wherein a client is configuring CSS classes directly and storing them in a database, we are limited in how much an external tool can reason about what is used / unused. 

ie.
```php
<div class="<?php echo $myClasses; ?>"></div>
```

_$myClass came from a MySQL database or something that we have no control over._

One solution to this problem is that a user could add additional code so that a tool could reason about what classes exist in scope.

```php
<?php
$classesAvailable = [
  “button” => true,
  “button-primary” => true,
];
// Split string of classes into an array of class names.
$classes = preg_split('/\s+/', $myClasses, -1, PREG_SPLIT_NO_EMPTY);
$myClassesValidated = “”;
foreach ($classes as $class) {
  if (isset($classesAvailable[$class])) {
    $myClassesValidated .= $class;
  }
}
?>
<div class=”<?php echo $myClassesValidated; ?>”>
</div>
```

_By doing the more verbose approach above, we've given the compiler enough information to know that $myClassesValidated can only have up to 2 classes, "button" and "button-primary"._

Another solution is that within our CSS, we simply add a note that states that we don't want the compiler to check if that rule is used or not. This would also be useful for stateful CSS classes that the tool might not be able to reason about. Ie. JavaScript applied classes

```css
// fel:unchecked
.button {
  display: block;
}
```

_Linting tools such as TypeScript Lint allow you to ignore rules certain properties / patterns with a code comment._

Finally, we could avoid needing to get an accurate model of the HTML by forcing developers to use something like [BEM](http://getbem.com/introduction/) or [CSS Modules](https://github.com/css-modules/css-modules), wherein they keep rules simply by only targeting 1 class without nesting.

### Built-in and custom rules

There should be built-in rules to stop users from using incorrect HTML tags. For example, using an anchor link as a button should be banned by default, ie.
```
<a href="#" class="btn"></a>
```
Would recommend you switch to:
```
<button type="button" class="btn"></button>
```
or at the very least, apply the [button role](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/button_role).
```
<a href="#" class="btn" role="button"></a>
```

There could also either be built-in or user-defined CSS rules that assist with:
- Warning the use of [viewport height units](https://bugs.webkit.org/show_bug.cgi?id=141832), as they don't work as expected on iOS.
- Warning the use of position: absolute / relative without defining an explicit z-index. We've personally found that when you don't at least explicitly give a z-index of 0, you get unpredictable rendering order across browsers.

### Framework / Language Agnostic

**Use cases for our tool**

- A Wordpress project that is made up of PHP, JavaScript and CSS/SASS files.
- A TypeScript React application that is made up of TypeScript files and pre-processed CSS files (most likely SASS as it seems to the most popular)
- A SilverStripe project is going to be up of PHP, SS templates (it's own template format!), JavaScript and CSS.
- A Go project made up of Go files, HTML templates (handlebar pre-processing), JavaScript and CSS.

To keep code written in this tool stable and reliable with minimal complexity, simply reading or validating multiple file formats seems out of the question. New features are being added to PHP, JavaScript and TypeScript all the time which could break the parser of our hypothetical tool. If this tool is to have strong backwards compatibility, it will most likely need its own format for templating that can at least target PHP, JavaScript and TypeScript. Ideally it would give it's users the ability to target whatever esoteric template language it needs to.

However, just because we may need our own template parser, that doesn't mean it needs to be its own special esoteric syntax or language. Infact, just using HTML with something simple like [Mustache](https://mustache.github.io/) template syntax might be all we want.