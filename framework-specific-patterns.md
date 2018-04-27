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

Meanwhile, `pytest-flask` also includes a built-in `client` fixture. Simply include it in tests where you'd like to GET or POST to Flask routes –

```python
def test_get_index(client):
    rv = client.get('/')
    assert rv.status_code == 200
```

– use [a Flask method](http://flask.pocoo.org/docs/0.12/api/#useful-functions-and-classes), such as `url_for` –

```python
def test_builds_https(client):
    assert url_for('/').startswith('https')
```

– or complete a Flask session transaction.

```python
def test_delete_from_session(client):
    del session['foo']
    session.modified = True
    assert 'foo' not in session
```

## Django

[`pytest-django`](https://pytest-django.readthedocs.io/en/latest/), a plugin for pytest, simplifies the task of integrating pytest with Django apps.

### Setup

First, install it.

```bash
pip install pytest-django
```

Then, add your freshly installed version of pytest-django to `requirements.txt`.

Most likely, your app has local/production, general, or custom settings. For testing, some content from these files should be included in `test_config.py` in your tests directory. Set up `test_config.py`:

**`tests/test_config.py`** (YMMV)
```python
import os


SECRET_KEY = 'some secret key'

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'storages',
    ... YOUR APPS ...
    'debug_toolbar',
    'django.contrib.postgres',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'debug_toolbar.middleware.DebugToolbarMiddleware',
]

ROOT_URLCONF = 'YOUR_PROJECT.urls'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'YOUR_DATABASE',
        'USER': '',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

STATIC_URL = '/static/'

BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "YOUR_PROJECT", "static"),
)
```

Then, tell Django where to find it:

```bash
export DJANGO_SETTINGS_MODULE=tests.test_config
```

### Interacting with the database

`pytest-django` handles the work of creating and tearing down your database, for you. (By default, it's called `test_` plus the name of the database you defined in `test_config.py`)

However, `pytest-django` also treats the database as lava: Your tests will fail if they try to interact with it. This is great if you agree that the database is lava. At DataMade, we aren't so sure.

To grant a test access to your database, use the `django_db` mark.

```python
import pytest

@pytest.mark.django_db
def test_cat_detail():
    # a test with access to the database
```

Now you are ready to write a test!

Among [other very useful fixtures](http://pytest-django.readthedocs.io/en/latest/helpers.html), `pytest-django` includes a `client` fixture providing you access to the [Django test client](https://docs.djangoproject.com/en/dev/topics/testing/tools/#the-test-client), so you can GET and POST to routes, to your heart's content. Let's use the `client` fixture to test the detail route for a `Cat` object.

```python
import pytest

from django.urls import reverse

from my_app.models import Cat

@pytest.mark.django_db
def test_cat_detail(client):
    # Create a new cat
    kitten = Cat.create(name='Teresa')

    # Create and call the URL
    url = reverse('detail', kwargs={'pk': kitten.id})
    response = client.get(url)

    # Test the success of the response code
    assert response.status_code == 200
```

### Transactional context

But wait! Doesn't creating a new `Cat` object without cleaning it up violate our standard to return test context to its original state, every time? Nope! `pytest-django` runs all database operations in a transaction, then rolls them back at the end of the test.

It is important to note that there are two, distinct types of transactional context. The default **does not insert any data into the database**. Objects you create in the ORM will be accessible via the ORM, but if the code you are testing runs raw SQL against the database, your new object will not be there.

If the code under test queries the database directly, you can configure Django to push new data to the database with the `transaction` argument.

```python
@pytest.mark.django_db(transaction=True)
def test_that_pushes_data_to_the_database():
    Cat.objects.create(name='Felix')

    with connection.cursor() as cursor:
        cursor.execute('''
            SELECT * FROM cats WHERE name = 'Felix'
        ''')
```

If you're interested in the mechanics of Django's transactional test context, our very own Jean Cochrane [wrote an excellent blog post](https://jeancochrane.com/blog/django-test-transactions) that's well worth your time!

#### Special case: Fixtures

If you need to push data to the database in a fixture, use the `django_db` mark as above, and include the `transactional_db` fixture in your fixture.

```python
@pytest.mark.django_db(transaction=True)
def a_fixture(transactional_db):
    # data is pushed to the database
```

### Model object fixtures

The pattern used above is great for one test, but creating the same model instance over and over becomes wearisome, fast, especially when your models have many fields to populate, and many of them don't need to change between tests. That's where fixtures come in.

Parameterization is great if you have a standard set of test cases across all tests that use a fixture, but it's not so great if your test cases vary. Luckily, we can create a fixture that "accepts" parameters on a case-by-case basis by yielding a factory that accepts those parameters and uses them to create and return your slightly custom object.

The pattern goes like this:

**`conftest.py`**
```python
import pytest

from your_app.models import Cat


@pytest.fixture
@pytest.mark.django_db
def cat():
    class CatFactory():
        def build(self, **kwargs):
            cat_info = {
                'name': 'Seymour',
                'color': 'orange',
                'favorite_food': 'tuna',
            }

            cat_info.update(kwargs)

            return Cat.objects.create(**cat_info)

    return CatFactory()
```

First, we define a factory class, `CatFactory`, with a `build` method that accepts unspecified `kwargs`. The `build` method defines standard dummy attributes, `cat_info`, for the model we'd like to build. It then `update`s the dummy data with any `kwargs` passed to the `build` method, and creates and returns the object.

Use the fixture like this.

**`test_cat.py`**
```python
def test_cat(cat):
    basic_cat = cat.build()  # Cat Seymour, loves tuna
    custom_cat = cat.build(name='Darlene', favorite_food='chicken')  # Cat Darlene, loves chicken
```

This becomes even more useful when your models have foreign keys. Rather than having to create an instance of the model to populate the foreign key, you can just include the model object fixture and call its build method.

**`conftest.py`**
```python
import pytest

from your_app.models import Cat, Owner


@pytest.fixture
@pytest.mark.django_db
def owner():
    class OwnerFactory():
        def build(self, **kwargs):
            owner_info = {
                'name': 'Hannah',
                'age': 26,
            }

            owner_info.update(kwargs)

            return Owner.objects.create(**owner_info)

    return OwnerFactory()

@pytest.fixture
@pytest.mark.django_db
def cat(owner):
    class CatFactory():
        def build(self, **kwargs):
            cat_info = {
                'name': 'Seymour',
                'color': 'orange',
                'favorite_food': 'tuna',
                'owner': owner.build(),
            }

            cat_info.update(kwargs)

            return Cat.objects.create(**cat_info)

    return CatFactory()
```

# TO-DO: Expound real quick on MGMT commands.

### Management commands

Your Django apps may have management commands, which may aid our application in significant ways – from sending emails to importing new content into your database. Such commands require tests. The below outlines how to create a database, call a management command, and test the results.

```python
import pytest
import sys

from django.core.management import import_data

from my_app.models import Cat

@pytest.mark.django_db
def test_command(mocker):
    with mocker.patch.object(Command, 'import_data') as mock_import:
        call_command('import_data', stdout=sys.stdout)

    # `import_data` adds cats to the database – check that those cats exist.
    kittens = Cats.objects.all()

    assert len(kittens) > 0
```
