Simple Authorization
====================

Before you use this snippet, I urge you to look at using `Flask-
Principal`_. It is a great piece of software, and is probably more
secure than this, and definitely better maintained. However, I found
that it was too much for my needs, and so I created this snippet.


Snippet
-------

This snippet is pretty simple. You just need to replace
get_current_user_role() with however you get the user's current role
and error_response() with however you want to notify the user that
they are not logged in. After you do that, you should be good to go.


::

    def requires_roles(*roles):
        def wrapper(f):
            @wraps(f)
            def wrapped(*args, **kwargs):
                if get_current_user_role() not in roles:
                    return error_response()
                return f(*args, **kwargs)
            return wrapped
        return wrapper




Usage
-----

Usage is equally as simple as the snippet itself. This is just a
decorator that you pass the required roles into. The required roles
can be any type of object, not just strings. Do note that if you use a
login extension such as Flask-Login, you should call it after the
login_required (or equivalent) decorator.


::

    @app.route('/user')
    @required_roles('admin', 'user')
    def user_page(self):
        return "You've got permission to access this page."
.. _Flask-Principal: http://packages.python.org/Flask-Principal/

