---
layout:       post
uuid:         60588087-597e-4502-8f05-223752c08dfb
categories:   javascript
tags:         [javascript, coffeescript, webpack]
title:        Decaffeinating a Large CoffeeScript Codebase Without Losing Sleep
date:         2017-07-18
author:       
  name:       Kevin Gao
  twitter:    sudowork
  github:     sudowork
feature_img:  null
sitemap:
  lastmod:    2017-07-18T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---

In 2013, DataFox chose [CoffeeScript][]<sup>†</sup> over ECMAScript 5 because it improved developer productivity.
It smoothed over many of the [bad parts of JavaScript](https://arcturo.github.io/library/coffeescript/07_the_bad_parts.html) and provided syntax to make common patterns more idiomatic: class definitions, fat arrow functions, and more.
Four years later, a lot has changed in the world of JavaScript, including many improvements to the language itself.

Over the course of two months, the engineering team at DataFox has been able to successfully convert our main web application codebase (~1400 files, ~200,000 lines) from CoffeeScript to ECMAScript 2015 (ES2015) while continuing to ship new features.
This post outlines our conversion process and provides some practical tips.

<sup>†</sup>CoffeeScript refers to CoffeeScript 1.x unless otherwise stated.

## Why Spend the Effort?

Because ES2015 (also known as ES6) introduced a number of new [language features](http://es6-features.org/), including those that originally attracted us to CoffeeScript, we felt that switching would simplify our development and thus improve developer productivity.
Additionally, the formalization of the [TC39 process](https://tc39.github.io/process-document/), which streamlines the creation, development, and acceptance of new [language proposals](https://github.com/tc39/proposals), gave us confidence that the language will continue evolving.

The JavaScript ecosystem has seen huge growth, especially in tooling.
While many tools in the ecosystem work perfectly fine with either JavaScript or CoffeeScript, here are a few benefits that we'd only get by switching to ES2015.

- Built-in static analysis for editors like [Visual Studio Code][] make reading and writing code much easier. Support for [automatic type acquisition](https://code.visualstudio.com/docs/languages/javascript#_automatic-type-acquisition) and projects like [DefinitelyTyped][] reduce friction for getting type analysis, which reduces bugs and gives higher confidence when refactoring.
- [ESLint][] for powerful and pluggable JavaScript linting.
- [jscodeshift][] makes [refactoring large codebases](https://www.toptal.com/javascript/write-code-to-rewrite-your-code) much more efficient.
- [Prettier][] takes the [bikeshedding](http://bikeshed.org/) discussions out of play when it comes to code formatting.
- [Webpack][] and [rollup.js][] are code bundlers that take advantage of [statically analyzing ES2015 modules](http://exploringjs.com/es6/ch_modules.html#static-module-structure) to enable tree shaking (dead code elimination).
- Node.js v6 uses a version of V8 that [supports essentially all of the ES2015 spec](http://node.green/), aside from implementing a ES2015 module loader.
- Ember CLI works best with modern JavaScript.

We also see benefits in recruiting and onboarding new engineers ([we're hiring!](https://www.datafox.com/company/careers/)).
Just going by numbers, there is significantly higher [usage](http://githut.info/), [interest, and satisfaction](http://stateofjs.com/2016/flavors/) for modern JavaScript over CoffeeScript.
Thanks to the larger community, modern JavaScript expertise is more accessible in distilled resources like books, tutorials, and Stack Overflow answers, allowing newcomers to JavaScript to hit the ground running.
Plus, true proficiency in CoffeeScript requires an understanding of the underlying JavaScript.

In summary, we felt confident that transitioning our codebase to modern Javascript would help us to build faster.

## Tools Used for Conversion

For context, our team was converting a monorepo consisting of many sub-projects that support our main web application.
These projects ranged from backend Node.js to frontend Ember 1.x projects - all written in CoffeeScript.
In order to convert CoffeeScript files to ES2015, we used the following tools:

- [bulk-decaffeinate][]/[decaffeinate][]: Converts the CoffeeScript code to ES2015. `bulk-decaffeinate` is a wrapper around `decaffeinate`, `eslint`, and `jscodeshift`.
- [ESLint][]: Linter for ES2015 code that we run with `--fix` to automatically fix many lint errors.
- [jscodeshift][]: Parses the ES2015 abstract syntax tree (AST) and performs code transformations to apply our own code style.
- [DataFoxCo/jscodemods][]: These are code modification scripts (codemods) we wrote to be used with `jscodeshift`. They read in ES2015 code, parse the AST, filter for specific patterns, transform the code, and finally output the transformed code to fix correctness/stylistic issues. See the README in that repository for more detailed information on each of the transformations.
  - [AST Explorer][] is a great tool for prototyping and writing your own codemods.

If you have a React codebase, check out Bugsnag's [depercolator](https://github.com/bugsnag/depercolator).
In addition to decaffeinate, it wraps [`cjsx-transform`](https://github.com/jsdf/coffee-react-transform), [`react-codemod`](https://github.com/reactjs/react-codemod), and [`prettier-eslint`](https://github.com/prettier/prettier-eslint) (a wrapper for `prettier && eslint --fix`).
We were not aware of this tool when we had started out conversion process; however, if we had a React codebase, we most likely would have used it!
Check out [their blog post](https://blog.bugsnag.com/converting-a-large-react-codebase-from-coffeescript-to-es6/) detailing their journey.

## Before Going Cold Turkey

### Know What You're Doing

Make sure you get a sense for the scope of work required to convert your codebase.
This will enable you to be a resource to your team before, during, and after the conversion process.
Because the actual conversion process is relatively quick, most of the work is in preparation and cleanup.
Here are a few items you'll want to check off as part of your personal preparation:

- Take inventory of just how much CoffeeScript you have. Use a tool like [`cloc`](https://github.com/AlDanial/cloc) to count files and real lines of code: `cloc --vcs=git --include-lang=CoffeeScript`.
- Measure your test coverage - hopefully it's high! High test coverage will give you and your team confidence that you haven't introduced bugs during the conversion.
- Identify and familiarize yourself with the various build tools in your codebase and continuous integration pipeline that deal with compiling CoffeeScript.
- Identify any ignorefiles (e.g. `.gitignore`, `.eslintignore`) or other configuration files (e.g. `.arclint`, git hooks) you have in your codebase that you'll need to modify.
- If you haven't already, [learn ES2015](https://babeljs.io/learn-es2015/) and learn it well.
- Write some projects in ES2015.
- Install [decaffeinate][] and try converting some representative files in your codebase.
  - `npm i -g decaffeinate` or `yarn global add decaffeinate` to install the `decaffeinate` binary globally.
  - Read through [decaffeinate documentation](https://github.com/decaffeinate/decaffeinate/tree/master/docs).
  - `decaffeinate example.js files.js`.
  - Identify correctness or style issues in generated code that should be fixed.
    - Some of these style issues can be fixed by using `--loose*` options on `decaffeinate`.
    - Other issues you may want to try fixing with `jscodeshift` codemods.
    - Spend time ironing out any edge cases you may come across in your code; it will save you time in the long run.

### Get Buy-in from Your Team

Before you get started, be sure to get buy-in from your team and leadership.
The time you spend converting code is time not spent building new features or fixing bugs.
You'll also need to help your engineers get familiar with ES2015.
Be sure to set aside time for training on best practices with ES2015.

Here are some strategies that worked well for our team:

- Host a Lunch and Learn on modern JavaScript. Here's [a fiddle](https://jsbin.com/vipugul/34/edit?js,console,output) I put together and presented to my team many months before even considering converting. The fiddle goes over some of the history of JavaScript and has code demos for some of the more exciting new features.
- Identify real problems that are fixed by switching from CoffeeScript to ES2015. We have a number of Ember.js frontend applications that use custom Grunt build processes, but we plan to switch our legacy apps to Ember CLI which uses modern JavaScript by default. There is little community support for using CoffeeScript with Ember CLI, and it didn't make sense for us to maintain multiple build systems when there was an easy way to standardize our process.
- Work with your team to establish a ES2015+ style guide and enforce it with ESLint rules. [Airbnb's style guide](https://github.com/airbnb/javascript) is a great starting point. Be careful; some of the styles enforced are actually newer than what's in the ES2015 spec (e.g. comma-dangle in function parameter lists).
- Demo how developer experience is improved by using modern JavaScript.
- Subscribe to [JavaScript Weekly][] and share interesting articles and videos with your team.
- Convert a smaller, less critical project and run it in production as a proof of concept.
- Write up documentation for how to convert code from CoffeeScript to ES2015. This post should give you a good start!

## The Conversion Process

This is an excerpt of our internal documentation that we developed on how to perform the conversion of a project.
By far the biggest pain in the actual conversion process was dealing with bound subclass methods which were used extensively throughout our codebase.
We provide our programmatic solution for resolving this [below](#bound-methods-on-subclasses).
For reference, you can also view the [official decaffeinate conversion guide](https://github.com/decaffeinate/decaffeinate/blob/master/docs/conversion-guide.md).

### Prerequisites

#### Dependencies

Before performing decaffeinate, make sure you've installed the following globally:

```shell
$ npm install -g bulk-decaffeinate decaffeinate jscodeshift eslint
$ git clone git@github.com:DataFoxCo/jscodemods.git ~/jscodemods
```

#### eslintrc

This process assumes that your project already has a `.eslintrc` (or `.eslintrc.js`) configured (may be in the parent path).
If you don't have a `.eslintrc` file, configure one before running `bulk-decaffeinate`.
Otherwise, you will have to do more work later to fix lint issues due to unnecessarily disabled rules and subsequent refactoring.

### Issues Preventing Conversion

#### `this` in Subclass Constructors before `super`

ES2015 does not allow references to `this` in a constructor before calling `super()`.
CoffeeScript, however, does allow this, preventing conversion.

```coffeescript
class Foo extends Bar
  constructor: ->
    @doSomething()
    super()

  doSomething: -> console.log(42)
```

Assuming it does not affect the logic, you must change the constructor to:

```coffeescript
class Foo extends Bar
  constructor: ->
    super()
    @doSomething()

  doSomething: -> console.log(42)
```

#### Bound Methods on Subclasses

For similar reasons as above, decaffeinate will not correctly convert code of subclasses that have bound (`=>`) methods<sup>†</sup> due to the presence of `this.myMethod.bind(this);` code in the constructor.
Code like the following will not convert:

```coffeescript
class Foo extends Bar
  baz: 42
  buzz: => console.log(@baz)
```

In order to fix this, you should use an unbound method:

```coffeescript
class Foo extends Bar
  baz: 42
  buzz: -> console.log(@baz)
```

Unfortunately, this cannot be done blindly because there may be some callers that rely on the method being bound.
For example, the if you did `[1].forEach(new Foo().buzz)` using the unbound method, it will log `undefined` instead of `42`.

You can fix the methods themselves with:

```shell
$ cd <directory to fix>
$ find . -name "*.coffee" | xargs perl -pi -e 's/^(  \w+:.*)=>$/\1->/g'
```
which will replace all `method: (...) =>` with `->` that are not indented.

But, you then need to fix all places that reference those methods as unbound functions (e.g. `_.map(someArray, @myMethod)`.
We wrote a [bind-iteratee-and-callback-methods](https://github.com/DataFoxCo/jscodemods#bind-iteratee-and-callback-methods) codemod to perform this binding automatically after the decaffeinate process.
The codemod supports some commonly used higher order functions like those of [`Array.prototype`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/prototype), [underscore](http://underscorejs.org/#collections), [lodash](https://lodash.com/docs/), and [async.js](https://caolan.github.io/async/docs.html).
Since it would be incredibly hard to know if the method actually needs to be bound, this may result in some superfluous binding.

While we were okay with binding for conversion purposes, we prefer to use arrow functions in newly written ES2015 code.
Arrow functions make the expected arguments explicit at the call site and offer a more natural method of `bind`ing `this`.

```javascript
// BAD
_.map([], this.myMethod);

// OKAY
_.map([], this.myMethod.bind(this));

// BETTER
const mapSomething = (something) => this.myMethod(something);
_.map([], mapSomething);
```

<sup>†</sup>`decaffeinate@2` would fail to convert this code by default unless using `--enable-babel-constructor-workaround` or `--allow-invalid-constructors`.
However, `decaffeinate@3` will convert it using the Babel constructor workaround by default, but our preference is still to fix these cases as described above.

#### Problems with `grunt-neuter`

We ran into issues when `decaffeinate` created helper methods like `__guard__` and `__guardMethod__` in our generated files ([see below](#__guard__-__guardmethod__-etc)).
When we have a mixture of code _before_ `require`s to be used for `neuter`-ing within the same file,  `grunt-neuter` [puts code into closures](https://github.com/trek/grunt-neuter#example), but the helper functions end up getting defined _after_ the neutered code.
This results in the helper methods and their callers being in different closures.

The solution to this problem is to make sure that, in cases where both `require`s and code occur in the same file, the `require`s occur at the top before any code.

#### ES5 "Classes" Extending ES2015 Classes

When going through and converting projects, work from the outside in: derived classes must be converted before the associated superclasses.
This is because an ES5 "class" will not be able to extend an ES2015 class properly.
For example, the normal CoffeeScript compiler<sup>†</sup> will compile `class Foo extends Bar` to:

```javascript
var Foo,
  __hasProp = {}.hasOwnProperty,
  __extends = function(child, parent) { for (var key in parent) { if (__hasProp.call(parent, key)) child[key] = parent[key]; } function ctor() { this.constructor = child; } ctor.prototype = parent.prototype; child.prototype = new ctor; child.__super__ = parent.prototype; return child; };
Foo = (function(_super) {
  __extends(Foo, _super);
  Foo.name = 'Foo';
  function Foo() {
    return Foo.__super__.constructor.apply(this, arguments);
  }
  return Foo;
})(Bar);
```

If `Bar` is a proper ES2015 class (e.g. `class Bar {}`), then you will get the following runtime error when trying to instantiate `new Foo()`: `TypeError: Class constructor Bar cannot be invoked without 'new'`.
This is due to the line `Foo.__super__.constructor.apply(this, arguments)`.

On the other hand, an ES2015 class should be able to extend a CoffeeScript compiled ES5 class, so working from the outside in should be fine.

<sup>†</sup>CoffeeScript 2 actually does compile to ES2015 classes, so it should be interoperable.
For a discussion on why you should use `decaffeinate` and not CoffeeScript 2 for your conversion, see [this GitHub issue](https://github.com/decaffeinate/decaffeinate/issues/1130).

### The Conversion Process

Here are some tips that we used to make the conversion process go smoothly:

- When choosing what parts of code to convert first, start with projects from the outside in to handle the class extension problem above.
- Start by converting tests to ES2015. They have the benefit of being able to test the correctness of the conversion for you, and we've found that generated code is generally pretty clean and idiomatic.
- In general, it may be easier to start from smaller projects and move to bigger ones. This gives you an opportunity to catch issues on smaller surface area.
- When you're converting each project, make sure to coordinate with your team, so that you aren't converting code that is actively being worked on. Consider holding mini code moratoriums to prevent cases like that from happening.
- Parallelize your efforts by having different people manage the conversion of each individual sub-project. The added benefit of this approach is that it gives everyone the chance to be familiar with the process and the generated ES2015 code. Ideally, the people working on each project should be familiar with the existing code already.
- The steps below include running a number of `jscodeshift` codemods as part of `bulk-decaffeinate`. However, you may choose to run those independently after converting the files; use `git add --patch` to do some spot-checking as you commit.
- Set a deadline for when you should finish converting everything by. Keep tabs on your process as you go along.
- Celebrate once you've converted all your code!

By following these practices, we were able to convert ~200,000 lines of actual CoffeeScript code (not counting whitespace and comments), spread across ~1400 files, all while continuing to build new features.

#### Step by Step Instructions for Converting a Project

Below are the step by step instructions provided to our team members that we wanted to share.
It will use `~/myproject` as the example project.

1. Disable any coffee compilers that are running.
1. Make sure you are on a clean git branch.
1. Update eslintignore files if necessary. For example `~/myproject/.eslintignore` may contain `src/**/*.js`. Remove this line in order for the later step of `eslint --fix` to run correctly. Commit this.
1. Change directories into the project you want to convert.

    ```shell
    $ cd ~/myproject/src
    ```

1. Make sure you have your desired `.eslintrc.js` settings put in place for your project.
1. Create the `bulk-decaffeinate.config.js` file in the project directory and update `fileFilterFn` if necessary:

    ```javascript
    // ~/myproject/src/bulk-decaffeinate.config.js
    const path = require('path');
    
    module.exports = {
      decaffeinateArgs: [
        // decaffeinate@3 settings
        '--loose',
        '--disable-babel-constructor-workaround',
        '--disallow-invalid-constructors',
        // We actually used decaffeinate@2 during our conversion with these settings:
        // '--keep-commonjs',
        // '--prefer-const',
        // '--loose-for-expressions',
        // '--loose-for-of',
        // '--loose-includes',
      ],
      // NOTE: You should update this fileFilterFn
      fileFilterFn(path) {
        return true; // most usual case
      },
      jscodeshiftScripts: [
        path.join(process.env.HOME, 'jscodemods/decaffeinate/fix-existential-conditional-assignment.js'),
        path.join(process.env.HOME, 'jscodemods/decaffeinate/fix-for-of-statement.js'),
        // Warning: Fixing the implicit return assignment is a potentially unsafe transformation.
        // It is possible that some code relies on return this.foo = 42; however, that is a bad practice.
        path.join(process.env.HOME, 'jscodemods/decaffeinate/fix-implicit-return-assignment.js'),
        path.join(process.env.HOME, 'jscodemods/decaffeinate/fix-multi-assign-class-export.js'),
        path.join(process.env.HOME, 'jscodemods/decaffeinate/remove-coffeelint-directives.js'),
        path.join(process.env.HOME, 'jscodemods/decaffeinate/bind-iteratee-and-callback-methods.js'),
        path.join(process.env.HOME, 'jscodemods/transforms/use-strict.js'),
      ],
    };
    ```

1. Check to see if `bulk-decaffeinate` will apply cleanly:

    ```shell
    $ bulk-decaffeinate check
    ```

1. If necessary, run this perl command to replace all bound methods with unbound methods. Note: This will result in most likely broken code which will be fixed by the `jscodemods/decaffeinate/bind-iteratee-and-callback-methods.js` codemod. The codemod is not 100% accurate, but should get almost every case with some false positives.

    ```shell
    $ find . -name "*.coffee" | xargs perl -pi -e 's/^(  \w+:.*)=>$/\1->/g'
    ```

1. Convert the files! This will end up: [saving the original files as `*.original.coffee`, renaming `*.coffee` files to `*.js`, creating commit, running `decaffeinate` on all files, create commit, run jscodemods, run `eslint --fix`, create commit](https://github.com/decaffeinate/bulk-decaffeinate#what-it-does).

    ```shell
    $ bulk-decaffeinate convert
    ```

1. You may want to run `eslint --fix your files here` one more time because bulk-decaffeinate may have caused some lint errors by putting comments directly above `'use strict';`.
1. Update the build steps in dev and production to no longer compile CoffeeScript to ES5.
1. Make sure your project still runs correctly!
1. Update ignorefiles, git hooks, etc.
1. Review your changes with someone.
1. Once your changes are approved, land them **without squashing** the commits. If you squash them, you lose the ability to track git history from before the conversion.

## Post-conversion

At this point, all your CoffeeScript files should have been converted to ES2015.
The post-processing done in the conversion process should have performed all the auto-fixable changes.
However, there are still steps you may have to do manually.
It is up to your discretion whether or not to do these immediately.

### `__guard__`, `__guardMethod__`, etc.

`decaffeinate` will create helper methods like `__guard__`.
You may want to replace these with idiomatic JavaScript.
If you are using a library like `lodash`, you can replace `__guard__`, `__guardMethod__`, and `__range__` with `_.get`, `_.invoke`, `_.range` respectively.

```javascript
// BAD
const name = __guard__(req.body.filters != null ? req.body.filters.company : undefined, x => x.name) != null ? __guard__(req.body.filters != null ? req.body.filters.company : undefined, x => x.name) : '';
const id = __guardMethod__((typeof maybeObj !== 'undefined' && maybeObj !== null ? maybeObj._id : undefined), 'toString', o => o.toString());
const indices = __range__(0, xs.length, true);

// OKAY
const name = _.get(req.body, 'filters.company', '');
const id = _.invoke(maybeObj, '_id.toString');
const indices = _.range(0, xs.length);
```

Many times, though, this situation arises from poorly written code that was too liberal in using the CoffeeScript existential operator `?`.
It can often be refactored to not require deeply nested existence checks.

```javascript
// Original Decaffeinated Code
const searchCompanyRoute = function(req, res, next) {
  const searchParams = {
    name: __guard__(req.body.filters != null ? req.body.filters.company : undefined, x => x.name),
    url: __guard__(req.body.filters != null ? req.body.filters.company : undefined, x1 => x1.url)
  };
  if (!searchParams.name && !searchParams.url) {
    return next(new Error('Must provide at least company name or url'));
  }
  return CompanySearchService.search(searchParams, function(err, results) { /* ... */ });
};
function __guard__(value, transform) {
  return (typeof value !== 'undefined' && value !== null) ? transform(value) : undefined;
}
```

```javascript
// After some light refactoring
// NOTE: It would be better to use some sort of validation middleware
const searchCompanyRoute = function(req, res, next) {
  const hasCompanyFilters = req.body.filters && req.body.filters.company;
  if (!hasCompanyFilters) {
    return next(new Error('Must provide company filters.'));
  }

  const { name, url } = req.body.filters.company;
  const hasNameOrUrl = name || url;
  if (!hasNameOrUrl) {
    return next(new Error('Must provide at least company name or url'));
  }

  const searchParams = { name, url };
  CompanySearchService.search(searchParams, function(err, results) { /* ... */ });
};
```

### Disabled ESLint Rules

There are some ESLint errors that cannot be automatically fixed with `--fix`.
These errors will be ignored on a per-file basis at the top of each file.
You will see a message like:

```javascript
/* eslint-disable
    consistent-return,
    no-case-declarations,
    no-console,
    no-param-reassign,
    no-undef,
    no-unused-vars,
    one-var,
    radix,
*/
// TODO: This file was created by bulk-decaffeinate.
// Fix any style issues and re-enable lint.
```

At some point, you will want to fix these errors and re-enable the lint rules.

## Coffee in Our Veins, Not Code

In just two months, there is not a single line of CoffeeScript remaining in our codebase.
With all that coffee out of the system, it's time to get some well-deserved rest... but not too much.
Although we've already fully fixed the style issues in over a third of our converted files, we will continue to clean files as we come across them.

While it's impossible to see the future and know exactly where CoffeeScript, ECMAScript, and the next cool language will stand, this conversion has already been incredibly valuable.
Today, modern JavaScript still has a huge amount of inertia and a vibrant community.
It continues to be a target language for languages like CoffeeScript, TypeScript, Kotlin, and many more.
Though the technology landscape constantly changes, the dividends of this conversion will continue to pay off in the foreseeable future.

Our team is always looking to responsibly adopt new technology that improves the product and developer experience.
Constant Learning and Empathy are just two of the values that make up our [engineering culture]({{ site.baseurl }}{% link _posts/2017-04-30-engineering-culture-at-datafox.md %}).
If this sounds like a team you'd like to be a part of, [we're hiring](https://www.datafox.com/company/careers/)!

<!-- Links -->
[CoffeeScript]: http://coffeescript.org/
[Visual Studio Code]: https://code.visualstudio.com/
[DefinitelyTyped]: https://github.com/DefinitelyTyped/DefinitelyTyped
[ESLint]: http://eslint.org/
[jscodeshift]: https://github.com/facebook/jscodeshift
[Prettier]: https://github.com/prettier/prettier
[Webpack]: https://webpack.js.org/
[rollup.js]: https://rollupjs.org/
[bulk-decaffeinate]: https://github.com/decaffeinate/bulk-decaffeinate
[decaffeinate]: http://decaffeinate-project.org/
[DataFoxCo/jscodemods]: https://github.com/DataFoxCo/jscodemods
[AST Explorer]: https://astexplorer.net/
[JavaScript Weekly]: http://javascriptweekly.com/
