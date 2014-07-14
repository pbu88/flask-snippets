HTTP Digest Auth
================

For more sophisticated authentication needs, HTTP Digest is a ready
solution. To use it in your Flask app, start by extending the
`authdigest`_ contribution to werkzeug with Flask knowledge:


::

    from functools import wraps
    from werkzeug.contrib import authdigest
    import flask
    
    class FlaskRealmDigestDB(authdigest.RealmDigestDB):
        def requires_auth(self, f):
            @wraps(f)
            def decorated(*args, **kwargs):
                request = flask.request
                if not self.isAuthenticated(request):
                    return self.challenge()
    
                return f(*args, **kwargs)
    
            return decorated


The create a digest database to hold your user authentication data:


::

    authDB = FlaskRealmDigestDB('MyAuthRealm')
    authDB.add_user('admin', 'test')


Use the authDB.requires_auth instance method to wrap a view function:


::

    from flask import request, session
    
    @app.route('/')
    @authDB.requires_auth
    def auth():
        session['user'] = request.authorization.username
        return "<h1>Content for authenticated user</h1>"


Or use the instance of authDB directly:


::

    @app.route('/auth')
    def authApi():
        if not authDB.isAuthenticated(request):
            return authDB.challenge()
    
        session['user'] = request.authorization.username
        return "<h1>Content for authenticated user</h1>"


The `authdigest module`_ was brought to you by `Shane Holloway`_,
under the same license as Werkzeug and Flask.

- Updates -

2013 April: Submit pull requests or fork `flask-digestauth`_ from
BitBucket.
.. _http://localhost:8000/': http://localhost:8000/'
.. _https://github.com/shanewholloway/werkzeug/: https://github.com/shanewholloway/werkzeug/
.. _Shane Holloway: http://bitbucket.org/shanewholloway
.. _authdigest module: https://github.com/shanewholloway/werkzeug/blob/master/werkzeug/contrib/authdigest.py
.. _https://bitbucket.org/shanewholloway/flask-digestauth: https://bitbucket.org/shanewholloway/flask-digestauth

