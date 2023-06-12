---
title: flask-tutorial-3
date: 2023-06-12 14:38:48
tags: [flask, python]
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

`app.test_client()` is used to test templates and view functions using `get` while `app.test_cli_runner()` is used to test commands using `invoke`.

``` python
def test_404_page(self):
    response = self.client.get('/nothing')  # target URL
    data = response.get_data(as_text=True)
    self.assertIn('Page Not Found - 404', data)
    self.assertIn('Go Back', data)
    self.assertEqual(response.status_code, 404)

def test_forge_command(self):
    result = self.runner.invoke(forge)
    self.assertIn('Done.', result.output)
    self.assertNotEqual(Movie.query.count(), 0)    
```


Testing can also simulate `post` actions to login as a specific user.

``` python
def login(self):
    self.client.post('/login', data=dict(
        username='test',
        password='123'
    ), follow_redirects=True)
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