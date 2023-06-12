---
title: flask-tutorial-2
date: 2023-06-12 11:01:09
tags: [flask, hexo]
---

## Database

`SQLAlchemy` will turn a database/table into a python class. First install `flask-sqlalchemy` to integrate `sqlalchemy` and `flask`.

``` bash
$ conda install flask-sqlalchemy==2.5.1 sqlalchemy==1.4.47
```

### Initialization

``` python
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
db = SQLAlchemy(app)
```

Besides, the configuration parameters needs to be set as well. 

``` python
WIN = sys.platform.startswith('win')
if WIN:
    prefix = 'sqlite:///'
else:
    prefix = 'sqlite:////'

app.config['SQLALCHEMY_DATABASE_URI'] = prefix + os.path.join(os.path.dirname(app.root_path), os.getenv('DATABASE_FILE', 'data.db'))
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False 
``` 

The first config is the path is the storage path of the database. `os.getenv` method will get environment variable first. If it doesn't exist, then use the second value in the parenthesis.

### Create tables

``` python
class User(db.Model): 
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(20)) 
```

Each class should inherit `db.Model`. Each field should be instantiated by `db.Column()`. Additional parameters for `db.Column()` include `primary_key`, `nullable`, `unique`, `default`. Common fields include `db.Integer`, `db.String (size)`, `db.Text`, `db.DateTime`, `db.Float`, `db.Boolean`.


### Command registration

Users can create customized flask commands by using `click`.

``` python
import click

@app.cli.command() 
@click.option('--drop', is_flag=True, help='Create after drop.')
def initdb(drop):
    if drop:
        db.drop_all()
    db.create_all()
    click.echo('Initialized database.')    # print to stdout in cmd
``` 

`db.create_all()` will create all the table schema in the database. `db.drop_all()` will drop everything in the database. `click.echo` will print the output to `stdout`.

### Query

There are manifold ways of selecting entries from tables in the database.

``` python
>>> movie = Movie.query.first()
>>> movie.title
'Leon'
>>> Movie.query.all()
[<Movie 1>, <Movie 2>]
>>> Movie.query.count()
2
>>> Movie.query.get(1)  # Get record whose primary_key=1
<Movie 1>
>>> Movie.query.filter_by(title='Mahjong').first()  # Get record where title field is 'Mahjong'.
<Movie 2>
>>> Movie.query.filter(Movie.title=='Mahjong').first()
<Movie 2>
>>> db.session.query(Movie).filter_by(title='Mahjong').all()
<Movie 2>
```

Here `filter_by` or `filter` will return all the entries that match the filter condition by calling method `.all()`.



### Insertion, update and deletion

``` python
# Insert an entry to the table.
user = User(name=name)
db.session.add(user)
db.session.commit()

# Update an entry in the table.
movie = Movie.query.get(2)
movie.title = 'WALL-E'
movie.year = '2008'
db.session.commit()

# Delete an entry from the table.
movie = Movie.query.get(1)
db.session.delete(movie)
db.session.commit()
```

`commit()` needs to be called everytime the database is changed.


## Form posts

Create a new form post.

``` python
<p>{{ movies|length }} Titles</p>
<form method="post">
    Name <input type="text" name="title" autocomplete="off" required>
    Year <input type="text" name="year" autocomplete="off" required>
    <input class="btn" type="submit" name="submit" value="Add">
</form>
```

`<input>` should specify `name` field.

To deal with `post` requests, the `app.route` decorator should be changed.

``` python
from flask import request

@app.route('/', methods=['GET', 'POST'])
def index():
    if request.method == 'POST':
        # Get form values.
        title = request.form.get('title') 
        year = request.form.get('year')
```

`request.form` is used to retrieve the form data.


## Flash Messages

`flash` is used in view functions to send messages to templates. 

``` python
from flask import flash

flash('Item Created.')
```

`get_flashed_messages()` can be used in templates to get messages sent by view functions.

``` python
{% for message in get_flashed_messages() %}
    <div class="alert">{{ message }}</div>
{% endfor %}
```

We need to set a new config here. The secret key will protect the application from being attacked by external codes.

``` python
app.config['SECRET_KEY'] = os.getenv('SECRET_KEY', 'dev')
```

## Redirect

`redirect` will make a new request to the target url.

``` python
return redirect(url_for('index')) 
```


## User Authentication

### Password hash codes

`werkzeug` can be used to generate hash codes from plain English.

``` python
from werkzeug.security import generate_password_hash, check_password_hash

class User(db.Model):
    def set_password(self, password):
        self.password_hash = generate_password_hash(password) 

    def validate_password(self, password):
        return check_password_hash(self.password_hash, password)
```

### Initialization
`flask-login` package can be used to realize user authentication.

``` bash
$ conda install flask-login
```

Then the `flask-login` can be initialized.

``` python
from flask_login import LoginManager

login_manager = LoginManager(app)

@login_manager.user_loader
def load_user(user_id):
    user = User.query.get(int(user_id))
    return user
```

Besides, `User` class model should inherit from `UserMixin` to enable authentication features. 

``` python
from flask_login import UserMixin

class User(db.Model, UserMixin):
...
```

### User login and logout

Then the user can be logged in and logged out using the following code,

``` python
from flask_login import login_user, logout_user

login_user(user)
logout_user()
```

`@login_required` decorator can be used for `view protections`. To be more specific, the page can only be accessed when logged in.

``` python
@app.route('/logout')
@login_required
def logout():
    logout_user() 
```

`current_user.is_authenticated` will be `true` if user has logged in, which can be used for `template content protections`.

``` python
{% if current_user.is_authenticated %}{% endif %}
```

