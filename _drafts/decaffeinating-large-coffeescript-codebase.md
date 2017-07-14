---
layout: post
title: Decaffeinating a Large CoffeeScript Codebase Without Losing Sleep
date: 2017-07-13
author: Kevin Gao
categories: javascript
uuid: 60588087-597e-4502-8f05-223752c08dfb
---

When DataFox was first founded in 2013, [CoffeeScript][]<sup>†</sup> was chosen over ECMAScript 5 for improved developer productivity.
It did a great job at smoothing over some of the [bad parts of JavaScript](https://arcturo.github.io/library/coffeescript/07_the_bad_parts.html) and provided syntax to make common patterns more idiomatic: class definitions, fat arrow functions, and more.
It’s now 2017 and a lot has changed in the world of JavaScript, many of these changes being improvements to the language itself.

Over the course of two months, the engineering team at DataFox has been able to successfully convert our main web application codebase (~1400 files, ~200,000 lines) from CoffeeScript to ECMAScript 2015 (ES2015) without losing any productivity.
This post outlines our conversion process and provides some practical tips for those of you in the same situation.

† CoffeeScript refers to CoffeeScript 1.x unless otherwise stated.

## Motivation

Before jumping into the how, let’s first take a look at _why_ we decided to undertake this project.

As mentioned above, ES2015 (also known as ES6) introduced a number of new [language features](http://es6-features.org/).
Many of these new features overlap with those that originally attracted us to CoffeeScript.
Even more important than the current feature set is the formalization of the [TC39 process](https://tc39.github.io/process-document/).
This process has streamlined the creation, development, and acceptance of new [language proposals](https://github.com/tc39/proposals), allowing the language to continue evolving.

Apart from the language, the JavaScript ecosystem has seen huge growth.
Tooling, in particular, has seen a large amount of development.
While many tools in the ecosystem can work perfectly fine with either JavaScript or CoffeeScript, here are a few benefits we've gotten from switching to ES2015.
- Built-in static analysis for editors like [Visual Studio Code][] can make reading and writing code much easier. Support for [automatic type acquisition](https://code.visualstudio.com/docs/languages/javascript#_automatic-type-acquisition) and projects like [DefinitelyTyped][] reduce friction for getting type analysis.
- [ESLint][] for powerful and pluggable JavaScript linting.
- [jscodeshift][] makes [refactoring large codebases](https://www.toptal.com/javascript/write-code-to-rewrite-your-code) much more efficient.
- [Prettier][] takes the [bikeshedding](http://bikeshed.org/) discussions out of play when it comes to code formatting.
- [Webpack][] and [rollup.js][] are code bundlers that take advantage of [statically analyzing ES2015 modules](http://exploringjs.com/es6/ch_modules.html#static-module-structure) to enable tree shaking (dead code elimination).
- Node.js v6 uses a version of V8 that [supports essentially all of the ES2015 spec](http://node.green/), aside from implementing a ES2015 module loader.
- Ember CLI works best with modern JavaScript.

At the end of the day, it's not about which programming language is better or how many tools you have in your toolchain; it's about the people.
Although we like to think choice of language shouldn't be that big of a deal, it still plays a part in recruiting and onboarding.
Just going by numbers, there is significantly higher [usage](http://githut.info/), [interest, and satisfaction](http://stateofjs.com/2016/flavors/) for modern JavaScript over CoffeeScript.
Thanks to the larger community, there is already an abundance of modern JavaScript expertise that has been distilled into resources such as books, tutorials, and Stack Overflow answers.
This can be invaluable for onboarding those who are new to the language and increases the chances of a new hire being able to hit the ground running, especially given prior experience.
Plus, to be truly proficient at CoffeeScript, you will inevitably need to understand what the underlying JavaScript is doing.

## Tools Used for Conversion

For context, our team was converting a monorepo consisting of many sub-projects that support our main web application.
These projects ranged from backend Node.js to frontend Ember 1.x projects - all written in CoffeeScript.
In order to convert CoffeeScript files to ES2015, we used the following tools:

- [bulk-decaffeinate][]/[decaffeinate][]: Converts the CoffeeScript code to ES2015. `bulk-decaffeinate` is a wrapper around `decaffeinate`, `eslint`, and `jscodeshift`.
- [ESLint][]: Linter for ES2015 code that we run with `--fix` to automatically fix many lint errors.
- [jscodeshift][]: Parses the ES2015 abstract syntax tree (AST) and performs code transformations to apply our own code style.
- [DataFoxCo/jscodemods][]: These are codemodification scripts we wrote to be used with `jscodeshift`. They will read in ES2015 code, parse the AST, filter for specific patterns, transform the code, and finally output the transformed code to fix correctness/stylistic issues. See the README in that repository for more detailed information on each of the transformations.
  - [AST Explorer][] is a great tool for prototyping and writing your own codemods.

I also want to give a nod to Bugsnag's [depercolator](https://github.com/bugsnag/depercolator), which similarly wraps `decaffeinate` but is specifically designed for React codebases.
In addition to decaffeinate, it wraps `cjsx-transform`, `react-codemod`, and `prettier-eslint` (a wrapper for `prettier && eslint --fix`).
We were not aware of this tool when we had started out conversion process; however, if we had a React codebase, we most likely would have used it!
Check out [their blog post](https://blog.bugsnag.com/converting-a-large-react-codebase-from-coffeescript-to-es6/) on their journey.

## Before Going Cold Turkey

### Be a Resource for Your Team

Make sure you get a sense for the scope of work required to convert your codebase.
This will enable you to be a resource to your team before, during, and after the conversion process.
To be honest, the actual conversion process is quite fast; almost all the work is entirely in preparation and cleanup.
That being said, it is critically important to make sure that your team is actually happy with the end result and that you end up in a better state than you were to begin with.
Here are a few items you may want to check off as part of your personal preparation:

- Take inventory of just how much CoffeeScript you have. Use a tool like [`cloc`](https://github.com/AlDanial/cloc) to count files and real lines of code: `cloc --vcs=git --include-lang=CoffeeScript`.
- Measure your test coverage - hopefully it's high! High test coverage can give you and your team more confidence in doing the conversion.
- Identify and familiarize yourself with the various build tools in your codebase and continuous integration pipeline that deal with compiling CoffeeScript.
- Identify any ignorefiles (e.g. `.gitignore`, `.eslintignore`) or other configuration files (e.g. `.arclint`, git hooks) you may have in your codebase that you may need to modify.
- If you haven't already, [learn ES2015](https://babeljs.io/learn-es2015/) and learn it well.
- Write some projects in ES2015.
- Install [decaffeinate][] and try converting some representative files in your codebase.
  - `npm i -g decaffeinate` or `yarn global add decaffeinate` to install the `decaffeinate` binary globally.
  - Read through [decaffeinate documentation](https://github.com/decaffeinate/decaffeinate/tree/master/docs).
  - `decaffeinate example.js files.js`.
  - Identify correctness or style issues in generated code that should be fixed.
    - Some of these style issues can be fixed by using `--loose*` options on `decaffeinate`.
    - Other issues you may want to try fixing with `jscodeshift` codemods.
    - Spend some time on this now to try to smooth out any edge cases you may come across in your code; it will save you time in the long run.
- Make sure you are able to create an equally good, but hopefully better, developer experience for writing modern JavaScript compared to CoffeeScript.

### Get Buy-in from Your Team

While doing all the personal preparation you can is great, the most important thing is to get buy-in from your team and any leadership necessary.
The time you spend converting is time not spent building; though, my hope is that sharing our experiences will reduce the amount of time you spend.
More than the time spent converting, though, is the switching cost of every engineer needing to become familiar and productive with ES2015.
Although this cost may be high in the short-term, for our team it was a worthwhile investment.

Here are some ideas to lessen the switching cost and to get buy-in from your team:

- Give a Lunch and Learn on modern JavaScript. Here's [a fiddle](https://jsbin.com/vipugul/34/edit?js,console,output) I put together and presented to my team many months before even considering converting. The fiddle goes over some of the history of JavaScript and has code demos for some of the more exciting new features.
- Identify real problems and value gained by switching from CoffeeScript to ES2015. For instance, our team has a number of Ember.js frontend applications with a custom Grunt build process, but we plan to switch to Ember CLI which uses modern JavaScript by default. While there are [precompilers](https://github.com/kimroen/ember-cli-coffeescript) to handle using CoffeeScript, there doesn't appear to be too many in the community taking that approach. This surfaces in the current lack of support for the latest Ember CLI and CoffeeScript 2.
- Work with your team to establish a ES2015+ style guide and enforce it with ESLint rules. [Airbnb's style guide](https://github.com/airbnb/javascript) is a great starting point. Be careful that some of the styles enforced are actually newer than what's in the ES2015 spec (e.g. comma-dangle in function parameter lists).
- Demo some of the shiny tools that work with modern JavaScript and show off the better developer experience.
- (Try to) keep up with news in the JavaScript community. Subscribe to [JavaScript Weekly][] and share interesting articles and videos with your team.
- Convert a smaller, less critical project and run it in production for a while.
- Write up documentation for how to convert code from CoffeeScript to ES2015. This blog post should give you a great start.

## The Conversion Process

This is an excerpt of our internal documentation that we developed on how to perform the conversion of a project.
By far the biggest pain in the actual conversion process was dealing with bound subclass methods which were used extensively throughout our codebase.
We provide our programmatic solution for resolving this below.
For reference, you can also view the [official decaffeinate conversion guide](https://github.com/decaffeinate/decaffeinate/blob/master/docs/conversion-guide.md).

### Prerequisites

#### Dependencies

Before performing decaffeinate, make sure you've installed the following globally:

```shell
$ npm install -g bulk-decaffeinate decaffeinate jscodeshift eslint@^3.19.0
$ git clone git@github.com:DataFoxCo/jscodemods.git ~/jscodemods
```

#### eslintrc

This process also assumes that your project already has a `.eslintrc` (or `.eslintrc.js`) configured (may be in the parent path).
If you don't have a `.eslintrc` file, it might be good to configure one before running `bulk-decaffeinate`.
Otherwise, you will have to do more work later to fix lint issues.

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

But, you then need to fix all places that reference those methods as unbound functions (e.g. `_.map(someArray, @myMethod, callback)`.
We wrote a [bind-iteratee-and-callback-methods](https://github.com/DataFoxCo/jscodemods#bind-iteratee-and-callback-methods) codemod to perform this binding automatically after the decaffeinate process.
The codemod supports some commonly used higher order functions like those of `Array.prototype`, underscore, lodash, and async.js.
Since it would be incredibly hard to know if the method actually needs to be bound, this may result in some superfluous binding.

While we were okay with binding for conversion purposes, we prefer to use arrow functions in newly written ES2015 code:

```javascript
// BAD
_.map([], this.myMethod, callback);

// OKAY
_.map([], this.myMethod.bind(this), callback);

// BETTER
const mapSomething = (something) => this.myMethod(something);
_.map([], mapSomething, callback);
```

† `decaffeinate@2` would fail to convert this code by default unless using `--enable-babel-constructor-workaround` or `--allow-invalid-constructors`.
However, `decaffeinate@3` will convert it using the Babel constructor workaround by default, but our preference is still to fix these cases as described below.

#### Problems with `grunt-neuter`

We ran into issues when `decaffeinate` created helper methods like `__guard__` and `__guardMethod__` in our generated files (see below re those helper methods).
The issue here only occurs if we have a mixture of code _before_ `require`s to be used for `neuter`-ing within the same file.
Due to the way `grunt-neuter` [puts code into closures](https://github.com/trek/grunt-neuter#example), the helper functions end up getting defined _after_ the neutered code, so the definitions of the helper methods and the usage of the helper methods end up in different closures.

The solution to this problem is to make sure that, in cases where both `require`s and code occur in the same file, the `require`s occur at the top before any code.

#### ES5 "Classes" Extending ES2015 Classes

When going through and converting projects, work from the outside in.
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

† CoffeeScript 2 actually does compile to ES2015 classes, so it should be interoperable.
For a discussion on why you should use `decaffeinate` and not CoffeeScript 2 for your conversion, see [this GitHub issue](https://github.com/decaffeinate/decaffeinate/issues/1130).

### The Conversion Process

Here are some high-level tips that we took advantage of to make the conversion process go smoothly:

- When choosing what parts of code to convert first, start with projects from the outside in to handle the class extension problem above.
- Tests are a great place to start converting. They have the benefit of being able to test the correctness of the conversion for you, and we've found that generated code is generally pretty clean and idiomatic.
- In general, it may be easier to start from smaller projects and move to bigger ones.
- When you're converting each project, communicating with your team is key to make sure that people don't have changes in flight to code that is about to be converted. Consider holding mini code moratoriums to prevent cases like that from happening.
- Parallelize your efforts by having different individuals manage the conversion of each individual sub-project. This also gives everyone the chance to be familiar with the process and the generated ES2015 code. Ideally, the individuals working on converting each piece should be familiar with the existing code already to some degree.
- The steps below include running a number of `jscodeshift` codemods as part of `bulk-decaffeinate`. However, you may choose to run those independently after converting the files; use `git add --patch` to do some spot-checking as you commit.
- Set a deadline for when you should finish converting everything by. Keep tabs on your process as you go along.
- Plan a celebration for once you've converted all your code!
- Celebrate!

Following the above advice, we were able to convert our ~200,000 lines of actual CoffeeScript code (not counting whitespace and comments), spread out across ~1400 files.
Plus, we were able to continue working on feature building at about the same pace as previously during and after the conversion.

#### Step by Step Instructions for Converting a Project

Below are the step by step instructions provided to our team members that we wanted to share.
It will use `~/myproject` as the example project.

1. Disable any coffee compilers that are running.
1. Make sure you are on a clean git branch.
1. Update eslintignore files if necessary. For example `~/myproject/.eslintignore` may contain `src/**/*.js`. Remove this line in order for the later step of `eslint --fix` to run correctly. Commit this.
1. Change directories into the project you want to run bulk-decaffeinate on.

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
1. Update the build steps in dev and production to no longer use compile CoffeeScript to ES5.
1. Make sure your project still runs correctly!
1. Update ignorefiles, git hooks, etc.
1. Review your changes with someone.
1. Once your changes are approved, land your changes, but do NOT squash your changes.

## Post-conversion

At this point, all your CoffeeScript files should have been converted to ES2015.
The post-processing done in the conversion process should have performed all the auto-fixable changes.
However, there are still steps you may have to do manually.
It is up to your discretion whether or not to do these immediately.

### `__guard__`, `__guardMethod__`, etc.

`decaffeinate` will create helper methods like `__guard__`.
You may want to replace these with sensible JavaScript.
If you are using a library like `lodash`, you can replace `__guard__`, `__guardMethod__`, and `__range__` with `_.get`, `_.invoke`, `_.range` respectively.
Many times, though, this situation is often due to bad code that was originally too liberally using the CoffeeScript existential operator `?`.
It can often be refactored to not require deeply nested existence checks.

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

## Some Well-deserved Rest

In just two months, there is not a single line of CoffeeScript remaining in our codebase.
With all that coffee out of the system, it's time to get some well-deserved rest.
Of course, we can't rest too much.
Although we've already fully fixed the style issues in over a third of our converted files, we are remaining diligent about cleaning files as we come across them.

While it's impossible to see the future and know exactly where CoffeeScript, ECMAScript, and the next cool language will stand, we have already gotten value out of doing this conversion.
As it stands today, modern JavaScript still has a huge amount of inertia and continues to be a target language for languages like CoffeeScript, TypeScript, Kotlin, and many more.
We are confident that the dividends of doing this conversion will continue to pay off even as we explore new technology.


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
