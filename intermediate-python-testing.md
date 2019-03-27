# `pytest` 201

* [Fixtures](#fixtures)
* [Parameterized fixtures](#parameterizing-fixtures)
* [Database fixtures](#database-fixtures)
  * [Scope, or How to avoid confusing dependencies between your tests](#scope-or-how-to-avoid-confusing-dependencies-between-your-tests)
* [mock](#mock)
* [requests-mock](#requests-mock)

## Fixtures

Fixtures are [explained in the `pytest` docs](https://docs.pytest.org/en/latest/fixture.html) as "a fixed baseline upon which tests can reliably and repeatedly execute." More simply, fixtures are methods that establish context or generate values for the tests you’re about to run.

As opposed to monolithic `setUp` and `tearDown` methods sometimes used in unit testing, fixtures allow you to create just enough context to run your tests. Smart fixture use makes your tests simultaneously faster and more thorough.

Creating a fixture is a two-step process. First, define a method that returns something in `conftest.py`. Then, add the `@pytest.fixture` decorator. That’s it! Fixtures defined in `tests/conftest.py` are available to all tests, whether they’re in the root `tests/` directory or a topical subdirectory. 

To use your new fixture, pass the fixture name as an argument to your test. You can then access the value/s your fixture returns through the fixture name.

For example, let’s say you want to do some math, but you want to make sure your computer is smart enough to do it. To avoid the arduous task of typing more numbers than necessary, let’s create a `numbers` fixture that returns two integers.

**`conftest.py`**

```python
import pytest

@pytest.fixture
def numbers():
    return 1, 5
```

Next, let’s use our numbers fixture to test whether the `+` operator does what it purports to do.

**`test_math.py`**

```python
def test_addition(numbers):
    a, b = numbers
    assert a + b == sum([a, b])
```

Finally, run your test.

```bash
(gut) that-cat-over-there:schmesting Hannah$ pytest -v

========================================= test session starts =========================================

platform darwin -- Python 3.5.2, pytest-3.2.1, py-1.4.34, pluggy-0.4.0 -- /Users/Hannah/.virtualenvs/gut/bin/python3.5

cachedir: .cache

rootdir: /Users/Hannah/hackz/datamade/schmesting, inifile:

plugins: flask-0.10.0

collected 1 item

tests/test_blah.py::test_addition PASSED

====================================== 1 passed in 0.01 seconds =======================================
```

Success!

<sup>**Note:** You may create additional `conftest.py` files with special fixtures in your topical subdirectories. Test modules within the subdirectory will then have access to both global and special fixtures. This a great way to stay organized in complex testing situations.</sup>

## Parameterizing fixtures

Behavior-driven development hinges on the question, "How should my code behave, given a certain context?" In our math example, we only test one context: summing two positive integers. What if one or both integers is negative? What if they’re floats? What if they’re not numbers at all? 

Lucky for us, the `pytest` framework can test multiple contexts with one fixture – testing sugar, known as parameterized fixtures. To parameterize your fixture, first create an iterable containing your test cases. Then, pass your iterable to the `params` keyword argument of the `fixture` decorator. Tests that include your fixture will now run once for every parameter. 

<sup>**Note:** the below example uses [`pytest.raises` as a context manager](https://docs.pytest.org/en/latest/assert.html#assertions-about-expected-exceptions) to test the `TypeError` exception.</sup>

**`conftest.py`**

```python

import pytest

# Contains values for a, b, and the error we expect or None
cases = [
    (1, 5, None), # two positive integers
    (-10, 3, None), # one positive integer, one negative integer
    (-29, -32, None), # two positive integers
    (0.45, 100, None), # one integer, one decimal
    (495.325, 99.3, None), # two decimals
    ('foo', 5, TypeError), # one string, one int
]

@pytest.fixture(params=cases)
def numbers(request):
    # Access the parameter through request.param
    return request.param
```

**`test_math.py`**

```python
import pytest
def test_addition(numbers):
    a, b, err = numbers
    if err:
        with pytest.raises(err):
            assert a + b == sum([a, b])
    else:
        assert a + b == sum([a, b])
```

```bash
(gut) that-cat-over-there:schmesting Hannah$ pytest -v

========================================= test session starts =========================================

platform darwin -- Python 3.5.2, pytest-3.2.1, py-1.4.34, pluggy-0.4.0 -- /Users/Hannah/.virtualenvs/gut/bin/python3.5

cachedir: .cache

rootdir: /Users/Hannah/hackz/datamade/schmesting, inifile:

plugins: flask-0.10.0

collected 6 items

tests/test_blah.py::test_addition[numbers0] PASSED

tests/test_blah.py::test_addition[numbers1] PASSED

tests/test_blah.py::test_addition[numbers2] PASSED

tests/test_blah.py::test_addition[numbers3] PASSED

tests/test_blah.py::test_addition[numbers4] PASSED

tests/test_blah.py::test_addition[numbers5] PASSED

====================================== 6 passed in 0.02 seconds =======================================
```

Success times 6!

<sup>**Note:** You can write cumulative fixtures by defining one fixture, then defining another fixture that includes (accepts as an argument) the first one.</sup>

```python
@pytest.fixture
def foo():
    return ‘foo’

@pytest.fixture
def bar(foo):
    # returns "foo!!!"
    return ‘{0}!!!’.format(foo)
```

If two or more fixtures that make up your cumulative fixtures are parameterized, any tests that include them will run for all possible combinations of the parameters. For example, if `foo` is run for parameters A, B, and C, and `bar` is run for parameters 1, 2, and 3, tests that include `bar` will run for A1, A2, A3, B1, B2, B3, C1, C2, and C3 – nine times in all.

## Database fixtures

Many of our web applications include databases. When they do, it’s important to test your SQL or ORM queries to ensure you’re accessing – or updating – the information you expect.

**If you are writing a Django app,** you get much of the following for free. Head over to [`pytest 301`](/framework-specific-patterns.md) to learn more. If you're new to `pytest`, [don't miss the introduction to fixture scope](#scope-or-how-to-avoid-confusing-dependencies-between-your-tests) below.

**For roll-your-own database testing (i.e., you're using Flask),** we need to define database fixtures manually.

First, define a test database connection string in `test_config.py`.

```python
DB_OPTS = {
    host=’localhost’,
    database=’cooltestdatabase’,
    username=’postgres’,
    password=’’,
    port=5432,
}
```

`pytest_postgresql` is a `pytest` extension that provides convenience methods for creating and dropping your test database.

To get started,

```bash
pip install pytest_postgresql
```

and add pytest_postgresql to your `requirements.txt` file.

Then, import the convenience methods, `init_postgresql_database` and `drop_postgresql_database`, and use them to create a `database` fixture in your `conftest.py` file.

**`conftest.py`**

```python
from pytest_postgresql.factories import init_postgresql_database, drop_postgresql_database
from .test_config import DB_OPTS

@pytest.fixture(scope='session')
def database(request):
    pg_host = DB_OPTS["host"]
    pg_port = DB_OPTS["port"]
    pg_user = DB_OPTS["username"]
    pg_db = DB_OPTS["database"]

    init_postgresql_database(pg_user, pg_host, pg_port, pg_db)
```

### Scope, or How to avoid confusing dependencies between your tests

Notice the scope keyword argument in the `fixture` decorator above. Scope determines the lifespan of the context created by a fixture. In `pytest`, there are three options:

* `session` – Fixture is run once for the entire test suite
* `module` – Fixture is run once per test module
* `function` – Fixture is run every time a test includes it

Fixtures are function-scoped by default. You should use this default unless you have a compelling reason. For example, it would be silly to re-create a database for every test. It is appropriate to use the `session` scope for fixtures that establish context that you aren’t going to change over the course of your testing, such as creating a database, initializing an application, or inserting _immutable_ dummy data.

**Be cautious!** Changes made to broadly scoped fixtures persist for all other tests in that session or module. This can lead to confusing circumstances where you are unsure whether your test is failing or the fixture context you expect was changed by a previous test.

To diagnose these unintended dependencies, you can run your tests in a random order with [`pytest-randomly`](https://pypi.python.org/pypi/pytest-randomly). Simply:

```bash
pip install pytest-randomly
```

Then run your tests as normal and behold! They will be shuffled by default, and you will be kept honest.

If you need to run your tests in sequential order, toggle random functionality with the `--randomly-dont-reorganize` flag. If you need to run your tests in the same random order, say for debugging a tricky failure, grab the `--randomly-seed=X` value from the top of your last run –

```
============================================================= test session starts =============================================================
platform darwin -- Python 3.5.2, pytest-3.2.1, py-1.4.34, pluggy-0.4.0 -- /Users/Hannah/.virtualenvs/dedupe/bin/python3.5
cachedir: .cache
Using --randomly-seed=1510172112
```

– and use it to run the tests again.

```bash
pytest --randomly-seed=1510172112
```

#### Finalizers

If you define a fixture that creates a table or inserts data that you intend to alter in your test, **use the `function` scope** and write a finalizer.

A finalizer is a method, defined within a fixture and decorated with `@request.addfinalizer`, that is run when that fixture falls out of scope. In effect, finalizers should undo your fixture. If you insert data, remove the data; if you create a table, delete the table; and so on. Let's add a finalizer to our `database` fixture.

**`conftest.py`**

```python
@pytest.fixture(scope='session')
def database(request):

    # define fixture

    @request.addfinalizer
    def drop_database():
        # Last arg is postgres version
        drop_postgresql_database(pg_user, pg_host, pg_port, pg_db, 9.6)
```

If a more broadly scoped fixture is unavoidable, **clean up any changes** made to the fixture context at the end of every test that uses it.

Even better, reconsider whether tests need alter the database at all. If you are testing a method that accepts the result of a query, do you need to create a table, populate it with dummy data, and query it just so you can run the test? Probably not! Insert the query result into a fixture, and use the fixture to run your method directly.

Working with more than one layer of state-dependent function calls? There's probably a way around it! Read on for an introduction to testing with `mock`.

## mock

`mock` is a `unittest` submodule that allows you to remove external dependencies from your test suite by coercing the methods that rely on them to return state _you_ control.

At DataMade, we like [`pytest-mock`](https://github.com/pytest-dev/pytest-mock/), a `pytest` extension that exposes the `mock` API to your tests with the `mocker` fixture.

Let's say you have a method that calls a method that queries your database and returns a result, then performs some operation on the returned result.

**`foo.py`**
```python
def get_result():
    # query the database and return result

def do_a_thing():
    results = get_result()

    for result in results:
        # do a thing
```

With `mock`, you can tell `get_result` to return a pre-determined list of things, instead of querying a database as written. This allows you to bypass the getting of the result (e.g., you don't need to create a database with a table to query) so you can focus on testing whether your operation does what you expect.

**`test_foo.py`**
```python
def test_do_a_thing(mocker):
    mocker.patch('foo.get_result', return_value=[1, 2, 3])

    do_a_thing()

    # does things on [1, 2, 3]

    # test things were done as expected
```

Tantalizing, right? 

You can also use `mock` to raise exceptions to test error handling, return canned website responses without hitting a live endpoint (as we do [in Metro](https://github.com/datamade/la-metro-councilmatic/blob/master/tests/test_events.py#L117)), or simply turn off state-altering parts of your code that aren't relevant to the test at hand. Finally, `mock` keeps track of whether and how mocked methods are called, so you can test how your code is used (called _n_ times, or with this or that argument), without necessarily having to run it.

For more on how to use `mock`, see [the quickstart](https://docs.python.org/3/library/unittest.mock.html#quick-guide) in the `mock` documention, as well as [this excellent tutorial](https://www.toptal.com/python/an-introduction-to-mocking-in-python). Meanwhile, here are a few of our own hard-won lessons from early `mock` use.

#### Mock methods where they're used, not defined.

Returning to our example, let's say our code were instead organized like so:

**`utils.py`**
```python
def get_result():
    # query the database and return result
```

**`tasks.py`**
```python
from utils import get_result

def do_a_thing():
    results = get_result()

    for result in results:
        # do a thing
```

If we want to mock `get_result` in a test, we must patch it in the `tasks` module, where it's used, not where it's defined:

**`test_tasks.py`**
```python
def test_do_a_thing(mocker):
    mocker.patch('tasks.get_result', return_value=[1, 2, 3])

    do_a_thing()

    # does things on [1, 2, 3]

    # test things were done as expected
```

#### Mocking classes is unconventional, but not impossible.

Per [the `mock` documentation](https://docs.python.org/3/library/unittest.mock.html#unittest.mock.patch):

> Patching a class replaces the class with a MagicMock instance. If the class is instantiated in the code under test then it will be the return_value of the mock that will be used.

This means you first need to create a `MagicMock` instance "spec'ed" to the class you are mocking, with any methods you need to override (i.e., to return a value or raise an Exception), overridden.

```python
mocked_class = MagicMock(spec=CLASS_YOU_ARE_MOCKING)

mocked_class.this_method.side_effect = AttributeError
mocked_class.that_method.return_value = 'nyan nyan nyan'
```

<sup>**Note:** Spec'ing means your mock object shares attributes and methods with your class. If you try to access an attribute or call a method incorrectly, i.e., without positional arguments, on a spec'ed mock object, it will raise an exception. This is in contrast to an unspec'ed mock object, which will let  you access any attribute or call any function you want without complaint. Spec'ing ensures your tests align with the API in your code base and stay in line as it changes (because your mock objects will break if they fall out of date).</sup>

Then, you need to patch the class you'd like to mock, and set its `return_value` to the `MagicMock` we just made.

```python
mock_handle = mock.patch('path.to.CLASS_YOU_ARE_MOCKING')
mock_handle.return_value = mocked_class
```

## requests-mock

Sometimes, your app sends requests to external URIs. And sometimes, these requests either have very little to do with the needs of your tests or fully interfere with them. [`requests-mock`](https://requests-mock.readthedocs.io/en/latest/overview.html) leaps into action!

`requests-mock` sets up [a transport adapter](http://docs.python-requests.org/en/latest/user/advanced/#transport-adapters) (a mechanism which defines how to handle HTTP requests). With a `requests-mock` adapter, you can load data into responses from specified URIs. How to use it? `requests-mock` comes with [multiple patterns for instantiating the `requests_mock.Mocker` class](https://requests-mock.readthedocs.io/en/latest/mocker.html#activation). For instance, use it as a context manager:

```
import requests
import requests_mock

def test_get_resp():
    with requests_mock.Mocker() as m:
        m.get("http://propane.org", text="that boy ain't right")
        rsp = requests.get("http://propane.org").text

        assert rsp.text == "that boy ain't right"
``` 

Your app might hit variations of the same URI, however. `requests-mock` saves you the tedium of mocking each path. You can define an adaptor that matches multiple paths, [e.g., with Regex](https://requests-mock.readthedocs.io/en/latest/matching.html#regular-expressions). Let's return to the example above, but build out some of its functionality.

**`utils.py`**
```python
def get_result():
    # query the database and return result

def scrape_an_endpoint(endpoint):
    # get data from https://api.com/v1/{endpoint}

def scrape_another_endpoint(endpoint):
    # get data from https://api.com/v1/{endpoint}
```

**`tasks.py`**
```python
from utils import get_result

def do_a_thing():
    results = get_result()

    for result in results:
        # get data and do a thing 
        data = scrape_an_endpoint(result.id)
        more_data = scrape_an_endpoint(result.id)
```

We can mock `get_result` in a test, as described above. However, `do_a_thing` now makes actual calls to an external, possibly unstable API. Such data could likely interfere with the predictability of `test_do_a_thing`. We have two options: (1) mock the results of every `scrape_an_endpoint` and `scrape_another_endpoint`, or (2) mock the requests to `https://api.com/v1`. The former can be tedious, particularly if you need to mock loads of utility functions. The latter option, happily, requires minimal effort with `requests-mock`:

**`test_tasks.py`**
```python
import pytest
import requests_mock

def test_do_a_thing(mocker):
    with requests_mock.Mocker() as m:
        # here, we tell the requests-mock adaptor to match
        # any URL with "api.com/v1," i.e., different endpoints,
        # https and http, query params, etc.
        matcher = re.compile('api.com/v1')
        m.get(matcher, json={}, status_code=200)

        mocker.patch('tasks.get_result', return_value=[1, 2, 3])

        do_a_thing()

        # does things on [1, 2, 3]

        # test things were done as expected
```

Need more? Check out [the tests for `scrapers-us-municipal`](https://github.com/datamade/scrapers-us-municipal/blob/master/tests/lametro), which makes healthy use of `requests-mock`.
