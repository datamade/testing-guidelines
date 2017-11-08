# `pytest` 201

* [Fixtures](#fixtures)
* [Parameterized fixtures](#parameterizing-fixtures)
* [Database fixtures](#database-fixtures)
  * [Scope, or How to avoid confusing dependencies between your tests](#scope-or-how-to-avoid-confusing-dependencies-between-your-tests)
* [mock](#mock)

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

Note: You may create additional `conftest.py` files with special fixtures in your topical subdirectories. Test modules within the subdirectory will then have access to both global and special fixtures. This a great way to stay organized in complex testing situations.

## Parameterizing fixtures

Behavior-driven development hinges on the question, "How should my code behave, given a certain context?" In our math example, we only test one context: summing two positive integers. What if one or both integers is negative? What if they’re floats? What if they’re not numbers at all? 

Lucky for us, the `pytest` framework can test multiple contexts with one fixture – testing sugar, known as parameterized fixtures. To parameterize your fixture, first create an iterable containing your test cases. Then, pass your iterable to the `params` keyword argument of the `fixture` decorator. Tests that include your fixture will now run once for every parameter.

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

Note: You can write cumulative fixtures by defining one fixture, then defining another fixture that includes (accepts as an argument) the first one.

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

Many of our web applications include databases. When they do, it’s important to test your SQL or ORM queries to ensure you’re accessing – or updating – the information you expect. To do that, we need to define a database fixture.

First, define a test database connection string in `test_config.py`.

```python
DB_OPTS = {
    host=’localhost’,
    database=’cooltestdatabase’,
    username=’postgres’,
    password=’’,
    port=5432,
}

DB_FMT = 'postgresql+dedupe://{username}:{password}@{host}:{port}/{database}'
DB_CONN = DB_FMT.format(**DB_OPTS)
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
from .test_config import DB_OPTS, DB_CONN

@pytest.fixture(scope='session')
def database(request):
    pg_host = DB_OPTS.get("host")
    pg_port = DB_OPTS.get("port")
    pg_user = DB_OPTS.get("username")
    pg_db = DB_OPTS["database"]
    init_postgresql_database(pg_user, pg_host, pg_port, pg_db)
```

DataMaders: If you need to add extensions to your test database, see [the `database` fixture in `dedupe-service`](https://github.com/datamade/dedupe-service/blob/master/tests/conftest.py).

### Scope, or How to avoid confusing dependencies between your tests

Notice the scope keyword argument in the `fixture` decorator above. Scope determines how long the context your fixture creates, sticks around. In `pytest`, there are three options:

* `session` – Fixture is run once for the entire test suite
* `module` – Fixture is run once per test module
* `function` – Fixture is run every time a test includes it

**Fixtures are function-scoped by default.** You should use this default unless you have a compelling reason. For example, it would be silly to re-create a database for every test. It is appropriate to use the `session` scope for fixtures that establish context that you aren’t going to change over the course of your testing, such as creating a database, initializing an application, or inserting immutable dummy data.

**Be cautious!** Changes made to broadly scoped fixtures persist for all other tests in that session or module. This can lead to confusing circumstances where you are unsure whether your test is failing or the fixture context you expect was changed by a previous test.

To diagnose these unintended dependencies, run your tests in a random order with [`pytest-randomly`](https://pypi.python.org/pypi/pytest-randomly). Simply:

```bash
pip install pytest-randomly
```

Then run your tests as normal and behold! They will be shuffled, and you will be kept honest. (Toggle random functionality with the `--randomly-dont-reorganize` flag if you need a run in sequential order.)

If you define a fixture that creates a table or inserts data that you intend to alter in your test, **use the `function` scope** and write a finalizer.

A finalizer is a method, defined within a fixture and decorated with `@request.addfinalizer`, that is run when that fixture falls out of scope. In effect, finalizers should undo your fixture. If you insert data, remove the data; if you create a table, delete the table; and so on. Let's add a finalizer that drops the database to the `database` fixture from before.

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

Even better, reconsider whether tests need alter the database at all. If you are testing a method that accepts the result of a query, do you need to create a table, populate it with dummy data, and query it just so you can run the test? Probably not! Read on for an introduction to testing with `mock`.

## mock

If a test must retrieve data from an API endpoint before processing that data, it is not a modular test. `mock` is a `unittest` submodule that allows you to remove external dependencies from your test suite by creating mocked versions of the methods or objects that rely on them.

When a method or object is mocked, it is replaced by `Mock` object. This object records whether and how the patched method or object is called rather than executing the underlying code. To return to our example, you can mock your data retrieval method and test that it is called with the arguments you expect without ever touching the API.

For more on `mock`, see [the quickstart](https://docs.python.org/3/library/unittest.mock.html#quick-guide) in the docs and [this excellent tutorial](https://www.toptal.com/python/an-introduction-to-mocking-in-python).

At DataMade, we like [`pytest-mock`](https://github.com/pytest-dev/pytest-mock/), a `pytest` extension that exposes the `mock` API to your tests with the `mocker` fixture.
