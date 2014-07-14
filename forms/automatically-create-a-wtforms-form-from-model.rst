Automatically create a WTForms Form from model
==============================================

There are some helpful `extensions`_ for many popular model mappings
(SQLAlchemy, App Engine, and Django) bundled with `WTForms`_ that
allow you to easily create a `wtforms.form.Form` from a model.

If using the `Flask-WTForms`_ extension, however, you'll want to make
sure you pass the base_class as `flaskext.wtf.Form` from the Flask
extension so the created form includes all the `API improvements`_
from the extension (ex. `form.validate_on_submit()`).


::

    from flaskext.wtf import Form
    from wtforms.ext.appengine.db import model_form
    from models import MyModel
    
    MyForm = model_form(MyModel, Form)


Note: In this example I am using App Engine's models and thus using
`model_form()` from `wtforms.ext.appengine.db`. If using SQLAlchemy,
you will want to use `model_form()` from `wtforms.ext.sqlalchemy.orm`.
This is currently not documented on WTForms extensions page, but you
can see it's been available since 0.5 (`source`_ | `changelog`_).


Customize created form
----------------------

You will want to refer the WTForms extension `docs`_ for more details,
but one nice item to mention is you can customize the created form at
creation. Here is an example of adding an extra validator to one of
the fields


::

    from flaskext.wtf import Form
    from wtforms.ext.appengine.db import model_form
    from wtforms import validators
    from models import MyModel
    
    MyForm = model_form(MyModel, Form, field_args = {
        'name' : {
            'validators' : [validators.Length(max=10)]
        }
    })


Note: `model_form()` will add validators automatically to the created
Form based on your model definitions, such as if a field is required.


Usage example
-------------

Here is a full example using a created form.


::

    from flaskext.wtf import Form
    from wtforms.ext.appengine.db import model_form
    from models import MyModel
    
    @app.route("/edit<id>")
    def edit(id):
        MyForm = model_form(MyModel, Form)
        model = MyModel.get(id)
        form = MyForm(request.form, model)
    
        if form.validate_on_submit():
            form.populate_obj(model)
            model.put() 
            flash("MyModel updated")
            return redirect(url_for("index"))
        return render_template("edit.html", form=form)
.. _API improvements: http://packages.python.org/Flask-WTF/#api-changes
.. _Flask-WTForms: http://packages.python.org/Flask-WTF

