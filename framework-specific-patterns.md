# `pytest` 301

* [Testing Flask applications](#flask)
* [Testing Django applications](#django)

## Flask

Vanilla Flask tests can quickly become a confusing jungle of context managers and cryptic jargon, without which you are plagued by `working outside of [ request / application ] context` errors.

`pytest-flask` is a `pytest` extension does the heavy lifting for you. All you need to do is define an `app` fixture.

To get started,

```bash

pip install pytest-flask

```

Add `pytest-flask` to your `requirements.txt` file.

Next, add an `app` fixture to the relevant `conftest.py` file or files. The pytest-flask docs provide a very basic example:

```python
@pytest.fixture
def app():
    app = create_app()
    return app
```

If your Flask app has a settings file, add a file containing the required settings to your test directory and load it into your app fixture by passing the location of your test config to the `config` keyword argument of `create_app`:

```python
@pytest.fixture
def app():
    # Assumes settings file called `tests/test_config.py`, adjust accordingly
    app = create_app(config='tests.test_config')
    return app
```

Once you complete this setup, you do not need to include the `app` fixture in any other fixtures or methods; it just lives there in the ether as context.

Meanwhile, `pytest-flask` also includes a built-in `client` fixture. Simply include it in tests where you’d like to GET or POST to Flask routes –

```python
def test_get_index(client):
    rv = client.get(‘/’)
    assert rv.status_code == 200
```

– use [a Flask method](http://flask.pocoo.org/docs/0.12/api/#useful-functions-and-classes), such as `url_for` –

```python
def test_builds_https(client):
    assert url_for(‘/’).startswith(‘https’)
```

– or complete a Flask session transaction.

```python
def test_delete_from_session(client):
    del session[‘foo’]
    session.modified = True
    assert ‘foo’ not in session
```

## Django

`pytest-django`, a plugin for pytest, simplifies the task of integrating pytest with Django apps.

First, install it.

```bash
pip install pytest-django
```

Then, add your freshly installed version of pytest-django to `requirements.txt`.

Most likely, your app has local/production, general, or custom settings. For testing, some content from these files should be included in `test_config.py` in your tests directory. Set up `test_config.py`, and tell Django where to find it:

```bash
export DJANGO_SETTINGS_MODULE=tests.test_config
```

You are ready to write a test! Create a test file (remember to use `test` as a filename prefix or suffix), and import pytest at the top:

```python
import pytest
```

Let’s start with basics – a simple URL test. The below outlines how to write a test for a detail view for a Cat model.

```python
import pytest

from django.urls import reverse

from my_app.models import Cat

@pytest.mark.django_db
def test_cat_detail(client):
    # Create a new cat
    kitten = Cat.create(name=’Teresa’)

    # Create and call the URL (with Django’s `reverse` function)
    url = reverse(‘detail’, kwargs={‘pk’: kitten.id})

    response = client.get(url)

    # Test the success of the response code
    assert response.status_code == 200
```

Your Django apps may have management commands, which may aid our application in significant ways – from sending emails to importing new content into your database. Such commands require tests. The below outlines how to create a database, call a management command, and test the results.

```python
import pytest
import sys
from unittest.mock import patch

from django.core.management import import_data

from my_app.models import Cat

# give the test function access to the test database, by marking it as such

@pytest.mark.django_db
# pass in a fixture that creates a database
def test_command(django_db_setup):
    with patch.object(Command, ‘import_data’) as mock_import:
        call_command(‘import_data’, stdout=sys.stdout)

    # `import_data` adds cats to the database – check that those cats exist.
    kittens = Cats.objects.all()

    assert len(kittens) > 0
```