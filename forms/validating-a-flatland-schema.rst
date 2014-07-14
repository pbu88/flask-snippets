Validating a Flatland Schema
============================

Given this example Flatland schema,


::

    from flatland import Form, String
    
    class SignInForm(Form):
        username = String
        password = String


The correct way to create an element instance from a Flask request is
as follows:


::

    if request.method in ['POST', 'PUT']:
        form = SignInForm.from_flat(request.form.items(multi=True))


A little wordy, so you might want to write a little helper function:


::

    def form_from_request(schema):
        return schema.from_flat(request.form.items(multi=True))


Now you can simply do this in a view:


::

    if request.method in ['POST', 'PUT']:
        form = form_from_request(SignInForm)


An alternative could be to subclass Form and add a validate_on_submit
method similar to what Flask-WTF does, so that you can refactor out
the request.method check.

