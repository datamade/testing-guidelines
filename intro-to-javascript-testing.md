# Intro to JavaScript Testing

* [The right style](#style)
* [jshint](#jshint)
* [jasmine](#jasmine)

## The right style

JavaScript, as a language itself, does not have too many opinions. Functional, reliable, "correct" JS comes in a variety of colors and shapes: in fact, the number of ways to [declare a function](https://www.bryanbraun.com/2014/11/27/every-possible-way-to-define-a-javascript-function/) reaches double digits.  

Its flexibility can cause confusion, headaches, and pure rage. JS invites hard-to-answer questions, like, "Am I doing this right?", "Am I doing this the best way possible?" – and the "right way" may encompass enough solutions to ruin an afternoon. 

So, narrow your options! As you write code and tests-for-your-code, check your JS against a style guide. Consistent style makes code easier to test and debug, while also minimizing cross-browser errors – a considerable pain point in JS. 

At DataMade, we recommend [our fork of the Node.js style guide](https://github.com/datamade/node-style-guide), a simple overview of best JS practices identified by [Felix Geisendörfer](http://felixge.de/). The [Crockford conventions](http://javascript.crockford.com/code.html) nicely complement the Node.js style guide: in it, [Douglas Crockford](https://en.wikipedia.org/wiki/Douglas_Crockford) gives a no-nonsense discussion of how to write "publication quality" JavaScript. 

## jshint

An impeccable style guide, however, does not prevent human error. Every coder has at least one horror story about an hours-long hunt for a missing semicolon that caused the (sometimes silent) crash of an entire program. 

[`jshint`](http://jshint.com/about/) can be the hero in these stories. `jshint` analyzes JavaScript line-by-line and reports typos and common bug-causing mistakes.  

Easily integrate `jshint` into your workflow using one of three strategies:

* Spotcheck code! Copy-and-paste select lines of potentially error-raising JS in the [`jshint` online console](http://jshint.com/). 
* Enable `jshint` as a constant, nagging reminder of JS best practices: add a JSHint plugin to your preferred code editor. At DataMade, many of us start on and continue using Sublime. For those Sublime users, install [JSHint Gutter](https://github.com/victorporof/Sublime-JSHint) through the Sublime package manager, and activate it by right clicking and selecting `JSHint --> Lint Code`. The results may be enlightening, but also exhausting: fix what needs fixing, and then clear the annotations with `CMD + esc`.
* Install the `jshint` [command line interface](http://jshint.com/docs/cli/), and integrate it with the regular running of your test suite. Run `npm install -g jshint` to install JSHint globally. Then, in terminal, tell JSHint to lint your files. 

The `jshint` cli has several options. Most simply, the `jshint` command accepts a specific file as a parameter:
 
``` bash
# Inspect this JS file!
jshint my_unique_app/static/js/custom.js

# Discouraging output...
my_unique_app/static/js/custom.js: line 14, col 51, Missing semicolon.
my_unique_app/static/js/custom.js: line 18, col 48, Missing semicolon.
my_unique_app/static/js/custom.js: line 52, col 34, Missing semicolon.

3 errors
```

Alternatively, JSHint can recursively discover all JS files in a directory and return a report: `jshint .` The results, in this case, may not be useful, since JSHint lints third-party libraries, too. For greater precision, add a configuration file in the root of your app, called `.jshintignore`. Here, tell JSHint to ignore, let's say, the directory where you store external libraries.

```
my_unique_app/static/js/lib/*
```

The JSHint cli allows for automated testing with Travis. Include in your `.travis.yml` file instructions for installing `jshint` and directive(s) for running the linter:

```
...

install:
- pip install --upgrade pip
- pip install --upgrade -r requirements.txt
- npm install -g jshint

...

script: 
- pytest tests
- jshint my_unique_app/static/js/custom.js

...
``` 

## jasmine


