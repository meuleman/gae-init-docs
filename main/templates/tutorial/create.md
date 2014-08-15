{% raw %}

There are a few things that we need to do in order to start adding new
contacts. In short we're going to create a **Form**, a **Handler** and a **Template**.


### Form

Create a new file `contact.py` in the `main` directory and add the following code 
that will be responsible for validating the user's input.

```python
from flask.ext import wtf

class ContactUpdateForm(wtf.Form):
  name = wtf.StringField('Name', [wtf.validators.required()])
  email = wtf.StringField('Email', [wtf.validators.optional(), wtf.validators.email()])
  phone = wtf.StringField('Phone', [wtf.validators.optional()])
  address = wtf.TextAreaField('Address', [wtf.validators.optional()])
```

For more information regarding the form validation refert to
[Flask-WTForms](http://flask.pocoo.org/docs/patterns/wtforms/).


### Handler

After creating the form we have to create a handler for it. 
Add the following code into the `contact.py` file.

```python
import flask
import auth
import model
from main import app

@app.route('/contact/create/', methods=['GET', 'POST'])
@auth.login_required
def contact_create():
  form = ContactUpdateForm()
  if form.validate_on_submit():
    contact_db = model.Contact(
        user_key=auth.current_user_key(),
        name=form.name.data,
        email=form.email.data,
        phone=form.phone.data,
        address=form.address.data,
      )
    contact_db.put()
    return flask.redirect(flask.url_for('welcome'))
  return flask.render_template(
      'contact_create.html',
      html_class='contact-create',
      title='Create Contact',
      form=form,
    )
```

#### Let's take a closer look at this new snippet

```python
import flask
import auth
import model
from main import app
```

This includes the needed imports for the Handler, just put
them in the beginning of the `contact.py` file with the rest of the
imports.

```python
@app.route('/contact/create/', methods=['GET', 'POST'])
```

The route and the methods that we are going to use to communicate with the webserver.
`GET` is to serve (or pull/obtain/get) the html form from the server and `POST` is to
submit (or push/post) the data to the server.

For more information refer to Flask documentation on
[routing](http://flask.pocoo.org/docs/quickstart/#routing).

```python
@auth.login_required
```

The purpose of this is to make sure that whoever is entering
this URL will be already signed in so we can make use of the `user_key`
variable of the authenticated user. If the user is not logged in, he/she will be
redirected to the sign-in page and then back to this URL.

### Template

After creating the form and a handler, we are going to need a template
to be able to obtain the user data to fill the form. 
Basically, this can be thought of as a user interface to the form we created earlier.
Create a new file `contact_create.html` in the `templates` directory
and paste the following code there:

```html
# extends 'base.html'
# import 'macro/forms.html' as forms

# block content
  <div class="page-header">
    <h1>{{title}}</h1>
  </div>
  <div class="row">
    <div class="col-md-4">
      <form method="POST" action=".">
        <fieldset>
          {{form.csrf_token}}

          {{forms.text_field(form.name, autofocus=True)}}
          {{forms.email_field(form.email)}}
          {{forms.text_field(form.phone)}}
          {{forms.textarea_field(form.address)}}

          <button type="submit" class="btn btn-primary btn-block">
            Create Contact
          </button>
        </fieldset>
      </form>
    </div>
  </div>
# endblock
```


### Finalise

Before we can start testing the creation of new contacts in our phonebook 
there are a couple of things we have to complete.


#### Import contact.py

We need to refer to the `contact.py` file in the `main.py` file,
because otherwise the Flask application won't be able to figure out the
routing rules. Include the following line bellow the rest of the imports
in the `main.py` file.

```python
import contact
```


#### Adding a link on the top bar

In order for the user to find his/her way around the phonebook and enable them to add contacts to it, 
we have to create a link to the top navigation bar.

Add the lines `2 - 4` inside the `<ul class="nav">...</ul>` element that you
will find in the `header.html` file that is located in the `templates/bit`
directory.

```html
<ul class="nav navbar-nav">
  <li class="{{'active' if html_class == 'contact-create'}}">
    <a href="{{url_for('contact_create')}}"><i class="fa fa-file"></i> Create Contact</a>
  </li>
  ...
</ul>
```


### Testing

If your development server is still running, then by visting the
[http://localhost:8080/](http://localhost:8080/)
you should notice the Create Contact link in the top navigation bar and you should be able
to actually start creating new contacts!

Create a couple of new contacts and also try to add a contact without
a name, or try to enter an invalid email and then press the **Create Contact**
button.

Since the contact list is not visible in the application yet, you can visit the
Google App Engine's admin console to make sure that the contacts are being
added, by visiting the admin console:
[http://localhost:8081/datastore](http://localhost:8081/datastore?kind=Contact).

{% endraw %}
