---
title: flask-tutorial-1
date: 2023-06-11 19:14:23
tags: [flask, tech]
---

This series of post will introduce the details of how to implement a basic flask web page. The tutorial I'm following is given [here](https://tutorial.helloflask.com/).

## Preparations

To begin with, create a `.gitignore` file to exclude unnecessary files when pushing to Github. 

``` bash
$ nano .gitignore
```

The file content should be as following.

```
*.pyc
*~
__pycache__
.DS_Store
.env
*.db
htmlcov/
.coverage
```

The whole tutorial will be conducted in `conda` environment for easier package management.


## Manage Environment Variable

First `python-dotenv` needs to be installed, which will make `flask` automatically read environment variable from `.flaskenv` and `.env` file in the root directory.

``` bash
$ conda install python-dotenv
```

Then in the file `.flaskenv`, we need to write the following code.

``` 
FLASK_DEBUG=1
FLASK_APP=watchlist
```

The first line setting `FLASK_DEBUG` to 1 will enable debug mode. In this mode, refreshing the browser will display the changes once the codes are modified. Otherwise the whole application needs to be rerun.

The default web application file is `app.py`. If the actual name is `hello.py`, then `FLASK_APP` needs to be set as `hello.py`. In this case, *watchlist* is the package name of the web application.


## Run the Application.

``` python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Welcome to My Watchlist!'
```

Then in the root directory,

``` bash
$ flask run
```


## View Functions

View functions are often used to deal requests given by template pages, often needed to be bound with urls by `app.route()`.

### URL rules

One view function can be registered multiple URLs.

``` python
@app.route('/')
@app.route('/index')
@app.route('/home')
def hello():
    return 'Welcome to My Watchlist!'
```

Variables can be integrated in urls. `escape()` can escape special characters in the url. Also, users can specify the variable type in the url by calling `<int:xxx>`.

``` python
from markupsafe import escape

@app.route('/user/<string:name>')
def user_page(name):
    return f'User: {escape(name)}'
```

It is also possible to generate url from a view function.

``` python
from flask import url_for

print(url_for('hello'))
# /
print(url_for('user_page', name='peter'))
# /user/peter
print(url_for('test_url_for', num=2))
# /test?num=2
```



## Templates


### Template writing
The HTML page rendering is often written in `Jinja2`.


``` jinja
<h1>{{ username }}的个人主页</h1>
{% if bio %}
    <p>{{ bio }}</p>  {# Indent is not a must #}
{% else %}
    <p>Self introduction.</p>
    <p>{{ movies|length }} Titles</p>
{% endif %}
```

- `{{ ... }}` to introduce variable in the template.
- `{% ... %}` to mark special condition sentence.
- `{# ... #}` to write annotations.

There exists a way of dealing with variables inside the HTML code -- `{{ variable|filter }}`. An exmaple is `{{ movies|length }}`, which is equivalent to `len(movies)` in python.

### Template rendering
Then to render the page, one can follow the following code using `render_template()`.

``` python
from flask import Flask, render_template

@app.route('/')
def index():
    return render_template('index.html', name=name, movies=movies)
```

Variables in the templates can be passed as keyword variables.

### Context view functions

For variables that will be used in multiple templates, define a `app.context_processor` decorator will come in handy.

``` python
@app.context_processor
def inject_user():  # Can take an arbitrary function name.
    user = User.query.first()
    return dict(user=user)  # same as return {'user': user}
```

The function should return `dict` objects. The variable `user` can be directly used in all the template files without being passed as keyword variables when rendering templates.

### Handling errors

Use `app.errorhandler()` decorator to handle error pages.

``` python
@app.errorhandler(404)
def page_not_found(e):
    user = User.query.first()
    return render_template('404.html', user=user), 404
```

Apart from returning the rendered template, the error code needs to be returned as well. For normal view functions, the default error code is `200` indicating success, which does not need to specify.


### Template inheriting

To do inheritance in templates, a `base.html` needs to be written at first. 

``` jinja
...
<head>
    ...
    {% block head %}{% endblock %}
    ...
</head>
<body>
    ...
    {% block content %}{% endblock %}
    ...
</body>
```

The child templates can insert content into `block head` & `block content` as following,

``` jinja
{% extends 'base.html' %}

{% block content %}
    <p>{{ movies|length }} Titles</p>
{% endblock %}
```

The first line is to declare which template to inherit. Then writing contents inside `block content` will add contents to `block content` in the base template.


## Static files

Files like images or gifs should be stored in `static` folder. The code for importing images is displayed as following,

``` html
<img src="{{ url_for('static', filename='foo.jpg') }}">
```