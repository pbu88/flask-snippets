Lazy SQLAlchemy setup
=====================

If you are using the new Flask configuration together with the
application factory pattern, one thing you will want to do if using
SQLAlchemy is initialize a SQLAlchemy session for different
requirements. For example, for unit tests you don't want to use the
production database.

Furthermore you may need to use a SQLAlchemy session outside the
request scope, for example in the shell.

The scoped_session function, which provides a thread-safe session,
expects a factory function. Normally we would use sessionmaker :


::

    from sqlalchemy import create_engine
    from sqlalchemy.orm import scoped_session, sessionmaker
    
    engine = create_engine("sqlite:///myapp.db")
    
    db_session = scoped_session(sessionmaker(bind=engine))


In order to initialize SQLAlchemy dynamically however we need to pass
in a factory function that does not require a ready engine instance.
For this we can use create_session :


::

    from sqlalchemy import create_engine
    from sqlalchemy.orm import scoped_session, create_session
    
    engine = None
    
    db_session = scoped_session(lambda: create_session(bind=engine))


We then need a function to create the engine when needed:


::

    def init_engine(uri, **kwargs):
        global engine
        engine = create_engine(uri, **kwargs)
        return engine


Provided you call init_engine first you can then use db_session
thereafter, as the scoped_session will bind the session to the current
value of engine .

You can call init_engine in your application factory function:


::

    def create_app(config):
        
        app = Flask(__name__)
        app.config.from_pyfile(config)
        
        init_engine(app.config['DATABASE_URI'])
        
        return app


Now you can use the db_session anywhere in your application, as long
as you first call create_app .
.. _http://flask.pocoo.org/docs/shell/#firing-before-after-request: http://flask.pocoo.org/docs/shell/#firing-before-after-request

