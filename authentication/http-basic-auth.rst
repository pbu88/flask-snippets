HTTP Basic Auth
===============

For very simple applications HTTP Basic Auth is probably good enough.
Flask makes this very easy. The following decorator applied around a
function that is only available for certain users does exactly that:


::

    from functools import wraps
    from flask import request, Response
    
    
    def check_auth(username, password):
        """This function is called to check if a username /
        password combination is valid.
        """
        return username == 'admin' and password == 'secret'
    
    def authenticate():
        """Sends a 401 response that enables basic auth"""
        return Response(
        'Could not verify your access level for that URL.\n'
        'You have to login with proper credentials', 401,
        {'WWW-Authenticate': 'Basic realm="Login Required"'})
    
    def requires_auth(f):
        @wraps(f)
        def decorated(*args, **kwargs):
            auth = request.authorization
            if not auth or not check_auth(auth.username, auth.password):
                return authenticate()
            return f(*args, **kwargs)
        return decorated


To use this decorator, just wrap a view function:


::

    @app.route('/secret-page')
    @requires_auth
    def secret_page():
        return render_template('secret_page.html')


If you are using basic auth with mod_wsgi you will have to enable auth
forwarding, otherwise apache consumes the required headers and does
not send it to your application: `WSGIPassAuthorization`_.
.. _WSGIPassAuthorization: http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIPassAuthorization

