Complex Validation with Flask-WTF
=================================

Sometimes you are in the situation where you need to validate a form
with custom logic that can not necessarily be reduced to a validator
on a single field. A good example are login forms where you have to
make sure a user exists in the database and has a specific password.

Thankfully Flask-WTF makes this very easy since you can hook into the
whole validation process:


::

    from flask.wtf import Form, TextField, PasswordField, validators
    from myapplication.models import User
    
    
    class LoginForm(Form):
        username = TextField('Username', [validators.Required()])
        password = PasswordField('Password', [validators.Required()])
    
        def __init__(self, *args, **kwargs):
            Form.__init__(self, *args, **kwargs)
            self.user = None
    
        def validate(self):
            rv = Form.validate(self)
            if not rv:
                return False
    
            user = User.query.filter_by(
                username=self.username.data).first()
            if user is None:
                self.username.errors.append('Unknown username')
                return False
    
            if not user.check_password(self.password.data):
                self.password.errors.append('Invalid password')
                return False
    
            self.user = user
            return True


Here we pull the user from the database in the general validation
step, validate username and password by hand and attach errors to the
individual fields if something goes wrong. We then also keep the user
object around so that we can use it in a view:


::

    from flask import flash, redirect, url_for, session, render_template
    
    @app.route('/login', methods=['GET', 'POST'])
    def login():
        form = LoginForm()
        if form.validate_on_submit():
            flash(u'Successfully logged in as %s' % form.user.username)
            session['user_id'] = form.user.id
            return redirect(url_for('index'))
        return render_template('login.html', form=form)


Login forms are perfectly paired with the `secure redirect form`_ from
the snippet database which redirects a user back to the page they came
from.
.. _https://gist.github.com/mickey06/4770903: https://gist.github.com/mickey06/4770903
.. _secure redirect form: /snippets/63

