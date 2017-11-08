# Writing Tests, the DataMade Way

These are the principles, procedure and patterns that guide test writing at DataMade. Looking for something a little more didactic? [Skip to the tutorials](#tutorials).

## Testing principles

1. Tests address **what a method should do, given specific contexts**. This philosophy is called behavior-driven development, or BDD (supplementary reading [here](https://dannorth.net/introducing-bdd/) and [here](https://www.toptal.com/freelance/your-boss-won-t-appreciate-tdd-try-bdd)). BDD testing, as the name indicates, centralizes the behavior of the code: "this function should send an email," “this function should hyphenate a PIN,” “this function should return a queryset,” etc. BDD reduces the number of unhandled edge cases by requiring the developer to think about code functionality from a multidimensional perspective. Likewise, BDD rewards early communication and deliberate boundary setting with our clients, which helps us better understand and communicate the scope of our work and the time needed to complete it.

2. Testing is an **essential part** of dynamic applications. Include time for writing and running tests in the scope and statement of work for projects, where appropriate.

3. Tests **speed up the feedback loop** during development, enabling rapid iteration on a feature prior to deployment.

4. Tests and your codebase are **reciprocally beneficial**. In other words, a good test enables you to write more modular code, which enables you to write more modular tests, and so on.

5. Tests are conducted on **real data**, whenever possible, and realistic data otherwise.

## Testing procedure

**What should I test?**

Behavior-driven development (BDD) is based on an understanding of what should happen, given certain parameters. This principle is also useful to consider when writing tests: If a user does X, this application should do Y. Examples of test-worthy functionality include:

* Actions that alter state, e.g. session or database transactions
* Exception handling
* Dynamically displayed user-interface
* User interactions

Note that compatibility and style are also elements of testing. If a feature doesn’t work across environments and browsers, it doesn’t work. If the the code doesn’t adhere to agreed-upon style conventions, it isn’t merged. Use of `flake8` and `jshint` will get you well on your way to compatibility and style compliance.

Also note that it is only necessary to test your own code because languages and libraries (ideally) have tests of their own. For instance, you don't need to test that `print('foo')` prints "foo". (Seriously, [it's covered](https://github.com/python/cpython/blob/6f0eb93183519024cb360162bdd81b9faec97ba6/Lib/test/test_print.py).) On the other hand, if you write a method that prints "foo" if certain conditions are met, you should test that your method prints "foo" provided those conditions are met and doesn't print "foo" otherwise.

**How do I write tests?**

Practicing BDD means creating a "spec" for each feature prior to implementation. While some developers may prefer to spec on the fly or outside of a testing environment, it can also be useful to set up a test and outline the spec as a series of comments first. Development can then be done at the developer’s own pace – writing a piece of code or implementing the entire feature – then testing that it behaves as expected.

Whether or not you use tests to spec, document what you are testing in docstrings or comments, particularly where queries or clever assertions are involved.

Tests should operate independently of the outside world, and of each other. Use fixtures to create reusable data, whether it’s in a database or available through the fixture itself. Ideally, all non-database fixtures should be scoped to the function level. Document where fixtures exceed this scope, and be sure your tests return those fixtures to their original state, where applicable. For instance, if your fixture refers to a database table and your test alters that table, revert your changes at the end of your test.

**When should I run tests?**

Tests should be run as part of the continuous integration setup. (DataMaders: See [our internal guide to configuring deploy hooks](https://github.com/datamade/deploy-a-site/blob/master/Setup-deployment-hook.md#setup-codedeploy-hook-for-github--travis-ci).) Code that does not pass the tests is neither merged nor deployed.

In the event that a test fails, unless the functionality of the method has materially changed, fix the method, not the test.

## Tutorials

* [`pytest` 101](/intro-to-python-testing.md)
  * Organizing tests
  * Configuring tests
  * Running tests
* [`pytest` 201](/intermediate-python-testing.md)
  * Intro to fixtures
  * Parameterized fixtures
  * Database fixtures
    * Scope, or How to avoid confusing dependencies between your tests
  * mock
* [`pytest` 301](/framework-specific-patterns.md)
  * Testing Flask applications
  * Testing Django applications
* [Front-end testing](/intro-to-javascript-testing.md)
  * `jshint`
  * `jasmine`

## Examples

* [`dedupe`](https://github.com/dedupeio/dedupe) – test functionality of DataMade's fuzzy matching library
* [`django-councilmatic`](https://github.com/datamade/django-councilmatic/blob/master/councilmatic_core/tests.py) – test all endpoints in a Councilmatic instance are valid
* [`large-lots`](https://github.com/datamade/large-lots/tree/master/lots_admin/tests) - test Django management command in Large Lots sends notifications and updates the database accordingly
* [`occrp`](https://github.com/datamade/occrp-timeline-tool/tree/master/tests) – test Flask form submission and page content rendering
