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

Easily integrate `jshint` into your workflow, using one of three strategies:

* Spotcheck code! Copy-and-paste select lines of potentially error-raising JS in the [`jshint` online console](http://jshint.com/). 
* Enable `jshint` as a constant, nagging reminder of JS best practices. Add a JSHint plugin to your preferred code editor. At DataMade, many of us start on and continue using Sublime. For such Sublime user, install [JSHint Gutter](https://github.com/victorporof/Sublime-JSHint) through the Sublime package manager, and activate it by right clicking and selecting `JSHint --> Lint Code`. The results may be enlightening, but also exhausting: fix what needs fixing, and then clear the annotations with `CMD + esc`.
* 


## jasmine


