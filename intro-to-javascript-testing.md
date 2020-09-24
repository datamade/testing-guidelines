# Intro to JavaScript Testing

* [The right style](#the-right-style)
* [Testing your logic](#testing-your-logic)

## The right style

JavaScript, as a language itself, does not have too many opinions. Functional, reliable, error-free JS comes in a variety of colors and shapes: in fact, the number of ways to [declare a function](https://www.bryanbraun.com/2014/11/27/every-possible-way-to-define-a-javascript-function/) reaches double digits. The flexibility of JS can create headaches, inspire pure rage, and ruin an afternoon.

Narrow your options! As you write code and tests-for-your-code, check your JS against a style guide. Consistent style makes code easier to test and debug, while also minimizing cross-browser errors – a considerable pain point in JS.  

An impeccable style guide, however, does not prevent human error. Every coder has at least one horror story about an hours-long hunt for a missing semicolon that caused the (sometimes silent) crash of an entire program. At DataMade, we recommend [ESLint](https://eslint.org/) as a way to enforce a coherent JS syntax and automate the hunt for typos.

We integrate `ESLint` into our workflow as a step in continuous integration—typically, when any commit is pushed to a feature branch. This means that we catch syntax errors early, but are not stymied during local development as we introduce new code and troubleshoot. For an example of what this looks like, see the Gatsby app cookiecutter template. Notable files:

- `package.json`: This setup requires installation of the package `eslint-config-react-app`. This is also where you can define the `yarn test` command used in `test.yml`
- `.eslintrc.js`: This file defines the syntax rules ESLint will follow. It's based on the `react-app` defaults, with minor modifications.
- `.github/workflows/test.yml`: This Âworkflow instructs Github to run the ESLint tests on every push.

## Testing your logic

The right style and perfect syntax make for clean, readable, debuggable JavaScipt. Yet, testing JS functionality requires something distinct. For this, we recommend [Jasmine](https://jasmine.github.io/2.0/introduction.html), a testing framework for behavior-driven development.

### Python integration

Jasmine [plays nicely with Django and Flask apps](https://jasmine.github.io/setup/python.html). In the virtualenv of your app, install Jasmine and initialize a project:

```
# Install
pip install jasmine

# Initialize a project
jasmine-install
```

`jasmine-install` creates a `spec` repo, where all testing specs and the configuration file (i.e., `jasmine.yml`) live. Before writing and running tests, tell Jasmine where to find relevant files by specifying, most importantly, the `src_files` and `src_dir` attributes:

```
...

src_files:
  - js/*

...

src_dir: 'my_app/static'

...
```

Then, in terminal, run:

```bash
jasmine
```

Visit `http://127.0.0.1:8888`, and view your test results.

Of course, without any tests, you should see relatively sparse output, something along these lines: "finished in 0.008s No specs found". Let's change that by writing a test! Open a JS file, and create a simple function – but not within `$(document).ready`. Why? `$(document).ready` [hides the functions inside a closure](http://bittersweetryan.github.io/jasmine-presentation/#slide-17).

```javascript
function makeTheTruth() {
    return true;
};
```

Then, make a spec file:

```bash
touch spec/javascripts/custom_spec.js
```

Open the newly created spec file, and add a test:

```javascript
describe("Test custom.js", function() {
  it("returns true", function () {
      var result = makeTheTruth();
      expect(result).toBe(true);
  });
});
```

Reload `http://127.0.0.1:8888`, and the browser should show the number of tests and the number of failures: "finished in 0.007s 1 spec, 0 failures".

### Resources to consider for advanced integrated testing

At DataMade, we are actively trying to improve our testing protocols. We hope to consider more robust ways of Jasmine integration with our python apps, as suggested by these sources:

* [django-jasmine](https://github.com/jakeharding/django-jasmine)
* [django.js for Jasmine views](http://djangojs.readthedocs.io/en/latest/test.html)
