---
layout:       post
uuid:         73102c01-fccc-4774-9716-4c9c97ae43f8
categories:   coffeescript
tags:         [javascript, coffeescript, vim, sublime, atom]
title:        "Edit CoffeeScript like a Pro in 6 Minutes"
date:         2014-08-23
author:       
  name:       Ben Trombley
  twitter:    bentrombley
  github:     bentrombley
feature_img:  null
sitemap:
  lastmod:    2014-08-23T08:43:04
  priority:   0.5
  changefreq: monthly
  exclude:    'no'
---


<img src="/img/sublime-lint-errors.png" style="width: 100%" />
<small><b>Pictured Above:</b> See errors and inconsistencies in your code as you type.</small>

There are tons of articles explaining how great CoffeeScript is as a language (which is true), but few that actually talk about how you should set up your dev environment to write it.  As a CoffeeScript shop, this is how we do it at DataFox.

## Sublime Text Editor

To start, I tried to use my favorite tools, [vim](http://www.vim.org/) and [IntelliJ](http://www.jetbrains.com/idea/).  However, while IntelliJ is great at Python, Java, Scala, and PHP, its CoffeeScript plugin is woefully lacking.  Vim, meanwhile, takes extensive configuration to support advanced features like linting, and can be a polarizing choice when hiring engineers used to IDE or emacs.

Fortunately, [Sublime](http://www.sublimetext.com/) is a fanastic text editor with much of the power of an IDE without the bloat.  Plus, it comes with a solid Vim plugin that approaches the full functionality of the real thing.

## Set up Sublime for CoffeeScript (6 min)

### Install [Sublime Text 3](http://www.sublimetext.com/3) (60s)
Don't install Sublime Text 2 or the plugins won't work.

### Install [Package Control](https://sublime.wbond.net/installation) (30s)
[Package Control](https://sublime.wbond.net/installation) makes installing plugins virtually instantaneous.  Follow the instructions on the page.

### Install Plugins (120s)
Access the "Package Control: Install Package" command by opening the [command pallette](http://docs.sublimetext.info/en/sublime-text-3/extensibility/command_palette.html) (`shift-cmd-p` on Mac, `shift-ctrl-p` on Windows) and typing "install".

Then use the autocomplete to quickly install these plugins:

  - git
  - git gutter
  - Better CoffeeScript
  - sidebar enhancements
  - SublimeLinter
  - SublimeLinter-coffeelint

And include these plugins if you use any of these languages:

  - LESS
  - Handlebars
  - Jade

Sublime is awesome and will install the plugins without any restart.

### Update Your Settings (30s)

CoffeeScript relies on consistent indentation, so update your settings to enforce it (_and make sure your team does the same!_).  Indentation is an oddly personal matter, so modify as needed.  Edit the settings (`cmd+,`) and paste in:

    {
      "auto_indent": true,
      "color_scheme": "Packages/User/Monokai (SL).tmTheme",
      "detect_indentation": false,
      "file_exclude_patterns":
      [
      ],
      "ignored_packages":
      [
      ],
      "smart_indent": false,
      "tab_size": 2,
      "translate_tabs_to_spaces": true,
      "trim_trailing_white_space_on_save": true
    }

The important thing is to standardize your spacing/tabbing and disable the automatic indentation which can cause bugs.

### Setup Linting (120s)

CoffeeScript is a very "expressive" untyped language which really means it is very ambiguous language with lots of easy-to-make mistakes. Linting takes one minute to setup. Seriously, just do it.

#### Prerequisites
I assume you have Node and npm installed already.

#### Install Coffeelint
Note it must be installed globally for Sublime to call it:

      npm install -g coffeelint

Test that it works by running it on a file:

      coffeelint path/to/a/file.coffee

#### (Optional) Configure Lint Rules

Createa `coffeelint.json` file in your project by running:

      cd <project root>
      coffeelint --makeconfig > coffeelint.json

Edit the file to turn on/off the rules, which are all [clearly documented](http://www.coffeelint.org/#options).

#### (Optional) Fix nvm + zsh
You may need to also do these steps, taken from the [coffeelint documentation](https://github.com/SublimeLinter/SublimeLinter-coffeelint)

  1.  If you are using `nvm` and `zsh`, ensure that the line to load nvm is in `.zshenv` and not `.zshrc`.
  2.  In order for `coffeelint` to be executed by SublimeLinter, you must ensure that its path is available to SublimeLinter. Before going any further, please read and follow the steps in [“Finding a linter executable”](http://sublimelinter.readthedocs.org/en/latest/troubleshooting.html#finding-a-linter-executable) through “Validating your PATH” in the documentation.

On Mac, this means you should open terminal and type `which coffeelint` which will give you a path like **/usr/local/bin/**coffeelint.  Then type `echo $PATH` and if you don't see **/usr/local/bin** (or whatever you see) add it by editing `~/.bashrc` or `~/.zshrc` with the line `export PATH=$PATH:/usr/local/bin`.

#### (Optional) Enforce Linting as a Git Hook

To enforce your new linting rules, create a pre-commit git hook by editing `.git/hooks/pre-commit`:

    exitCode=0
    for file in `git diff --cached --name-only | grep "\.coffee$"`; do
      # ignore files that were deleted/moved as part of the commit
      if [ -e ${file} ]
      then
        coffeelint ${file}
        exitCode=$((${exitCode} + $?))
      fi
    done

    if [[ exitCode -ne 0 ]];
    then
      echo "Coffee linting has failed, please fix the error(s).  If this is an incorrect error, either fix our linting rules (in coffeelint.json) or in this case commit with the --no-verify flag." 1>&2;
    fi

This will run the linter on all edited files when you commit.  I don't believe in tying developers' hands, so you can always skip this rule by running `git commit --no-verify`.

It is also a good idea to symlink `.git/hooks` to a directory in your repo so you can easily share hooks across your team.

#### Restart Sublime

#### Customize SublimeLinter

Edit the SublimeLinter rules, by opening the context menu (right-click) and modifying:
    - Lint Mode > Background
    - Mark Style > No Column Highlights Line
    - Mark Style > Stippled Underline

This will lint your files as you work and show errors with red underlines (of course, you can change this).


#### Test It Out
Open a .coffee file and behold!



It took a lot of trial-and-error to arrive at this setup, so I hope it is helpful.  If you have any other tips or feedback please share them with us: ops [at] datafox.co.
