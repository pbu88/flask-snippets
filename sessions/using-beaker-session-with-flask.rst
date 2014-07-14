Using Beaker session with Flask
===============================

The following is a simple application that shows how to bootstrap
beakers session middleware and access the session variables. The
example uses memcached as the backend.


::

    from flask import Flask, request
    from beaker.middleware import SessionMiddleware
    
    session_opts = {
        'session.type': 'ext:memcached',
        'session.url': '127.0.0.1:11211',
        'session.data_dir': './cache',
    }
    
    app = Flask(__name__)       
    
    @app.route('/')
    def index():
        session = request.environ['beaker.session']
        if not session.has_key('value'):
            session['value'] = 'Save in session' 
            session.save()   
            return "Session value set."
        else:
            return session['value']
        
    if __name__ == '__main__':
        app.wsgi_app = SessionMiddleware(app.wsgi_app, session_opts)
        app.run(debug=True)


You might want to use beaker in the situation where you have multiple
Flask applications, e.g. load balanced applications, that need to have
access to a shared session storage, which beaker can provide.
.. _http://flask.pocoo.org/snippets/121/: http://flask.pocoo.org/snippets/121/

