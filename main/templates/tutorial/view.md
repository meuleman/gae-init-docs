{% raw %}
In order to view details of individual contacts we again have to perform 
two main tasks: create the **Handler** that will retrieve the contact from 
the datastore and the **Template** to view the results in the browser.
And of course we need to be able to select from the contact list a 
single user whose details we're interested in.

### Handler

Add the following code to the `contact.py` file:

```python
@app.route('/contact/<int:contact_id>/')
@auth.login_required
def contact_view(contact_id):
  contact_db = model.Contact.get_by_id(contact_id)
  if not contact_db or contact_db.user_key != auth.current_user_key():
    flask.abort(404)
  return flask.render_template(
      'contact_view.html',
      html_class='contact-view',
      title=contact_db.name,
      contact_db=contact_db,
    )
```

### Template

After creating the handler, we are going to need a template to be able to 
present a contact's personal detail page. Create a new file `contact_view.html` 
in the `templates` directory and paste the following code there:

```html
# extends 'base.html'

# block content
  <div class="page-header">
    <h1>{{title}}</h1>
  </div>
  <p>{{contact_db.email}}</p>
  <p>{{contact_db.phone}}</p>
  <p>{{contact_db.address}}</p>
  <hr>
  <p>
    <a href="{{url_for('contact_list')}}">Back</a>
  </p>
# endblock
```

It's hard to test this template right now because we have to manually construct  
the URL for the user we're interested in.
So in the next section we'll add a link to make it easier to access.

### Finalise

As a final touch in order to view the contacts we'll modify the contact list
so when you click on a name you're able to view the contact's details page.

Find the line that renders the name of the contact and replace it with the following:

```html
...
<td>
  <a href="{{url_for('contact_view', contact_id=contact_db.key.id())}}">
    {{contact_db.name}}
  </a>
</td>
...
```

Now visit the contact list
([http://localhost:8080/contact/](http://localhost:8080/contact/)),
and you should be able to click on the name of each contact and view their
properties.

{% endraw %}
