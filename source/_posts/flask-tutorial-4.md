---
title: flask-tutorial-4
date: 2023-06-14 11:21:02
tags: [flask, tech]
---

## Flask-WTF

### WTForms
Flask can make forms implemented as a class by using `Flask-WTF`. The form classes can be stored in `forms.py`

``` python
from flask_wtf import FlaskForm
from wtforms import StringField, TextAreaField, SubmitField
from wtforms.validators import DataRequired, Length, ValidationError
        
class MessageForm(FlaskForm):
    name = StringField('Name', validators=[
                DataRequired(), 
                Length(1, 20)])
    submit = SubmitField('Submit')
```

The class should inherit from `FlaskForm`. Each input is defined as a field in the class. There are various `validators` to restrict the input format. 

### Customized validator

It is also possible to define a customized validator.

``` python
def command_check(form, field):
    if field.data != u'宝塔镇河妖':
        raise ValidationError('The captcha value must be "宝塔镇河妖".')

class MessageForm(FlaskForm):
    password = StringField(u'天王盖地虎', validators=[  
                    DataRequired(),
                    command_check])
```

### Bootstrap and Flask

To render the form in the HTML templates, `bootstrap` is needed. There are two versions of methods to render the forms.

The first one is to install `flask-bootstrap`. 

``` jinja
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}
```

The second is to install `bootstrap-flask`. Note that this package can only be installed through `pip` inside `conda`.

```jinja
{% from 'bootstrap/form.html' import render_form %}
{{ render_form(form) }}
```

`flask-bootstrap` will conflict with `bootstrap-flask` so only one package can be installed and used. But both packages needs initialization as following,

``` python
from flask_bootstrap import Bootstrap5
bootstrap = Bootstrap5(app)
```

## Bootstrap jquery class

There are useful `jqueries` that can be easy implemented by `Bootstrap` classes.

`data-dismiss` can be used to delete elements within specific classes. 

``` jinja                
<button type="button" class="close" data-dismiss="alert">&times;</button>
```

`data-toggle` set to `tooltip` can be used to display tooltips when the mouse is hovering certain items.

``` jinja
 <p data-toggle="tooltip" data-placement="top"
    data-timestamp="{{ message.time.strftime('%Y-%m-%dT%H:%M:%SZ') }}"
    data-delay="500"> {{ moment(message.time).fromNow(refresh=True) }}</p>
```

However, these functions can only take effect if the following script is placed at first among all the scripts.

``` jinja
<script type="text/javascript" src="{{ url_for('static', filename='js/jquery-3.2.1.slim.min.js') }}"></script>
```