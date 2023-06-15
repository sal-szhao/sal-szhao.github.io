---
title: flask-tutorial-3
date: 2023-06-12 14:38:48
tags: [flask, tech]
---

## Testing web pages

`unittest` is used for automatic testing. A sample testing method is as following.

``` python
import unittest
from app import app, db, Movie, User

class WatchlistTestCase(unittest.TestCase):

    def setUp(self):
        app.config.update(
            TESTING=True,
            SQLALCHEMY_DATABASE_URI='sqlite:///:memory:'
        )
        db.create_all()
        user = User(name='Test', username='test')
        user.set_password('123')
        movie = Movie(title='Test Movie Title', year='2019')
        db.session.add_all([user, movie])
        db.session.commit()

        self.client = app.test_client()
        self.runner = app.test_cli_runner()

    def tearDown(self):
        db.session.remove()
        db.drop_all() 

    # Test whether app exists.
    def test_app_exist(self):
        self.assertIsNotNone(app)

    # Whether app is under testing.
    def test_app_is_testing(self):
        self.assertTrue(app.config['TESTING'])
```

The class should be inherited from `unittest.TestCase`. `setUp` will be called before the testing starts while `tearDown` will be called after the test is finished.

In the test mode, the database path is set to `sqlite:///:memory:`, which will create an in-memory database that has a faster speed to access.

`app.test_client()` is used to test templates and view functions using `get` while `app.test_cli_runner()` is used to test commands using `invoke`. If the command has click options, the user can use a list to pass command arguments into `invoke`.

``` python
def test_404_page(self):
    response = self.client.get('/nothing')  # target URL
    data = response.get_data(as_text=True)
    self.assertIn('Page Not Found - 404', data)
    self.assertEqual(response.status_code, 404)

def test_forge_command(self):
    result = self.runner.invoke(forge)
    self.assertIn('Done.', result.output)
    self.assertNotEqual(Movie.query.count(), 0)

 def test_forge_command_count(self):
    result = self.runner.invoke(forge, ['--count', '40'])
    self.assertEqual(Message.query.count(), 40)    
```


Testing can also simulate `post` actions to login as a specific user.

``` python
def login(self):
    self.client.post('/login', data=dict(
        username='test',
        password='123'
    ), follow_redirects=True)
```

However, for `WTForms` in `flask`, one additional code needs to be added to `setUp()`. Because `WTForms` will automatically add a CSRF token to the post, which will forbid the test code from posting.

``` python
app.config["WTF_CSRF_ENABLED"] = False
```

There is also another way of generating error pages using `abort`.

``` python
def test_500_page(self):
    from flask import abort

    @app.route('/500')
    def throw_500_error():
        abort(500)

    response = self.client.get('/500')
    data = response.get_data(as_text=True)
    self.assertIn('Internal Server Error', data)
    self.assertEqual(response.status_code, 500)
```

Then add the following code to the end of the test file.

``` python
if __name__ == '__main__':
    unittest.main()
```

To test the coverage of the testing code, install `coverage` package.
``` bash
$ conda install coverage
```

Then run to get the results of coverage.

``` bash
$ python -m coverage run --source=watchlist test_watchlist.py
```

Alternatively, one can use the following commnads to see more details about the coverage test.

``` bash
$ coverage report
$ coverage html
```

## Deploy

As the [tutorial](https://tutorial.helloflask.com/deploy/) suggests, the deployment can be conducted on [PythonAnywhere](https://www.pythonanywhere.com/). However, there is one **important** thing to notice here.

The `venv` environment created in PythonAnywhere should be at least `python 3.7`. But `python 3.10` will not work.

``` bash
$ python3.7 -m venv env
```

## Some Symbols in HTML

There are special symbols encoded in HTML ending with `;`.

``` html
&times;     # Close 
&uarr;      # Up arrow
&darr;      # Down arrow
```

## Flask Moment

Flask moment can be used to render `timestamps`. It can be used to generate `... minutes ago`.

First it needs initialization.

``` python
from flask_moment import Moment
moment = Moment(app)
```

Then inside the `<head>` section, the following codes are needed to use the extension.

``` jinja
{{ moment.include_moment() }}
```

Then to record time from now, use the following code.

``` jinja
{{ moment(message.time).fromNow(refresh=True) }}
```

`fromNow()` will calculate the time difference between `messsage.time` and `now()`. With `refresh` set to `True`, the difference will autmatically refresh itself. 

The output can be various depending on the time difference.

```
a few seconds ago
2 minutes ago
an hour ago
...
```

Set a field in the model to record the current timestamp by using `datetime.utcnow()`.

``` python
from datetime import datetime
time = db.Column(db.DateTime, default=datetime.utcnow)
```

## Faker

`faker` package can be used to generate fake messages to insert into the database for testing.

``` python
from faker import Faker

fake = Faker()

message = Message(
    name=fake.name(),
    body=fake.sentence(),
    timestamp=fake.date_time_this_year()
)

db.session.add(message)
db.session.commit()
```

For instance, `fake.name()` can generate a random fake name. 