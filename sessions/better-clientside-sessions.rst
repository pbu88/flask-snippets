Better Client-side sessions
===========================

Flask by default uses the Werkzeug provided 'secure cookie' as session
system. It works by pickling the session data, compressing it and
base64 encoding it.


Motivation
----------

This was a reasonable decision at the time but now there are better
modules available and if you are fine with another external dependency
you could start using `itsdangerous`_ instead of the secure cookie
which works the same but has a couple of advantages:

1. the implementation comes from the django signing module and was
heavily reviewed in terms of the crypto used. 2. it can be used for
much more than just signing cookies 3. it uses JSON instead of pickle
by default. 4. it could be implemented client side to read the session
data from JavaScript. 5. it does not rely on Python specifics so your
proxy server could be able to read it.


Implementation
--------------

Starting with Flask 0.8 adding support is easy due to the newly
introduced session interface:


::

    from werkzeug.datastructures import CallbackDict
    from flask.sessions import SessionInterface, SessionMixin
    from itsdangerous import URLSafeTimedSerializer, BadSignature
    
    
    class ItsdangerousSession(CallbackDict, SessionMixin):
    
        def __init__(self, initial=None):
            def on_update(self):
                self.modified = True
            CallbackDict.__init__(self, initial, on_update)
            self.modified = False
    
    
    class ItsdangerousSessionInterface(SessionInterface):
        salt = 'cookie-session'
        session_class = ItsdangerousSession
    
        def get_serializer(self, app):
            if not app.secret_key:
                return None
            return URLSafeTimedSerializer(app.secret_key, 
                                          salt=self.salt)
    
        def open_session(self, app, request):
            s = self.get_serializer(app)
            if s is None:
                return None
            val = request.cookies.get(app.session_cookie_name)
            if not val:
                return self.session_class()
            max_age = app.permanent_session_lifetime.total_seconds()
            try:
                data = s.loads(val, max_age=max_age)
                return self.session_class(data)
            except BadSignature:
                return self.session_class()
    
        def save_session(self, app, session, response):
            domain = self.get_cookie_domain(app)
            if not session:
                if session.modified:
                    response.delete_cookie(app.session_cookie_name,
                                       domain=domain)
                return
            expires = self.get_expiration_time(app, session)
            val = self.get_serializer(app).dumps(dict(session))
            response.set_cookie(app.session_cookie_name, val,
                                expires=expires, httponly=True,
                                domain=domain)


To activate this session interface all you have to do is to change the
`session_interface` attribute on your application:


::

    app.session_interface = ItsdangerousSessionInterface()
.. _https://github.com/maxcountryman/flask-login/issues/31: https://github.com/maxcountryman/flask-login/issues/31
.. _itsdangerous: http://pypi.python.org/pypi/itsdangerous

