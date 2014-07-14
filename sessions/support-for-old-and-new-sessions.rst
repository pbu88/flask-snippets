Support for Old and New Sessions
================================

If you want to write an extension that supports Flask 0.7 session
monkey patching and the new 0.8 session interface we recommend this
glue code:


::

    try:
        from flask.sessions import SessionMixin, SessionInterface
    except ImportError:
        class SessionInterface(object):
            pass
    
        class SessionMixin(object):
            def _get_permanent(self):
                return self.get('_permanent', False)
            def _set_permanent(self, value):
                self['_permanent'] = bool(value)
            permanent = property(_get_permanent, _set_permanent)
            del _get_permanent, _set_permanent
    
            # you can use a werkzeug.datastructure.CallbackDict
            # to automatically update modified if you want, but
            # it's not a requirement.
            new = False
            modified = True
    
    
    class MySession(dict, SessionMixin):
         pass
    
    
    class MySessionInterface(object):
    
        def open_session(self, app, request):
            # load the session and return it.
            return MySession()
    
        def save_session(self, app, session, response):
            # save the session
            ...
    
    
    def init_my_extension(app):
        if not hasattr(app, 'session_interface'):
            app.open_session = lambda r: \
                app.session_interface.open_session(app, r)
            app.save_session = lambda s, r: \
                app.session_interface.save_session(app, s, r)
        app.session_interface = MySessionInterface()


The minimum interface expected is that open session returns an object
that implements this:

1. it has a `permanent` attribute. The mixing automatically stuffs
that into the session itself. 2. it either has `modified`
automatically set to `True` or it tracks assignments to the dict. 3.
it has a `new` attribute which however is not required to be set to
`True` for new sessions but if possible, it should be supported.

Generally it is however recommended that extensions do not attempt to
support 0.7 for new session backends due to the added complexity.

