CSRF Protection
===============

A common technique against `CSRF`_ attacks is to add a random string
to the session, and check that string against a hidden field in the
POST.


::

    @app.before_request
    def csrf_protect():
        if request.method == "POST":
            token = session.pop('_csrf_token', None)
            if not token or token != request.form.get('_csrf_token'):
                abort(403)
    
    def generate_csrf_token():
        if '_csrf_token' not in session:
            session['_csrf_token'] = some_random_string()
        return session['_csrf_token']
    
    app.jinja_env.globals['csrf_token'] = generate_csrf_token        


And then in your template:


::

    <form method=post action="">
        <input name=_csrf_token type=hidden value="{{ csrf_token() }}">
.. _CSRF: http://en.wikipedia.org/wiki/CSRF
.. _http://docs.djangoproject.com/en/dev/ref/contrib/csrf/#rejected-requests: http://docs.djangoproject.com/en/dev/ref/contrib/csrf/#rejected-requests

