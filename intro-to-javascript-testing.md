# Intro to JavaScript Testing

* [The right style](#style)
* [jshint](#jshint)
* [jasmine](#jasmine)

## The right style

JavaScript, as a language itself, does not have too many opinions. Functional, reliable, "correct" JS comes in a variety of colors and shapes: in fact, the number of ways to [declare a function](https://www.bryanbraun.com/2014/11/27/every-possible-way-to-define-a-javascript-function/) reaches double digits.  

Its flexibility can cause confusion, headaches, and pure rage. JS invites hard-to-answer questions, like, "Am I doing this right?", "Am I doing this the best way possible?" – and the "right way" may encompass enough solutions to ruin an afternoon. 

So, narrow your options! As you write code and tests-for-your-code, check your JS against a style guide. Consistent style makes code easier to test and debug, while also minimizing cross-browser errors – a considerable pain point in JS. 

At DataMade, we recommend [our fork of the Node.js style guide](https://github.com/datamade/node-style-guide), a simple overview of best JS practices identified by [Felix Geisendörfer](http://felixge.de/). The [Crockford conventions](http://javascript.crockford.com/code.html) nicely complement the Node.js style guide: in it, [Douglas Crockford](https://en.wikipedia.org/wiki/Douglas_Crockford) gives a no-nonsense discussion of how to write "publication quality" JavaScript. 


