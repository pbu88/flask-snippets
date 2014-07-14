Beaker session in Flask
=======================

This is an extension to snippet `Using Beaker session with Flask`_
that gives more seamless beaker session integration by using
SessionInterface.


::

    from flask import Flask, session
    from flask.sessions import SessionInterface
    from beaker.middleware import SessionMiddleware
    
    session_opts = {
        'session.type': 'ext:memcached',
        'session.url': '127.0.0.1:11211',
        'session.data_dir': './cache',
    }
    
    class BeakerSessionInterface(SessionInterface):
        def open_session(self, app, request):
            session = request.environ['beaker.session']
            return session
    
        def save_session(self, app, session, response):
            session.save()
    
    app = Flask(__name__)       
    
    @app.route('/')
    def index():
        if not session.has_key('value'):
            session['value'] = 'Save in session'    
            return "Session value set."
        else:
            return session['value']
        
    if __name__ == '__main__':
        app.wsgi_app = SessionMiddleware(app.wsgi_app, session_opts)
        app.session_interface = BeakerSessionInterface()
        app.run(debug=True)
.. _Using Beaker session with Flask: http://flask.pocoo.org/snippets/61/

