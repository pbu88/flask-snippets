Server-side Sessions with Redis
===============================

If you need to store a lot of session data it makes sense to move the
data from the cookie to the server. In that case you might want to use
redis as the storage backend for the actual session data.

The following code implements a session backend using redis. It allows
you to either pass in a redis client or will connect to the redis
instance on localhost. All the keys are prefixed with a specified
prefix which defaults to `session:`.


::

    import pickle
    from datetime import timedelta
    from uuid import uuid4
    from redis import Redis
    from werkzeug.datastructures import CallbackDict
    from flask.sessions import SessionInterface, SessionMixin
    
    
    class RedisSession(CallbackDict, SessionMixin):
    
        def __init__(self, initial=None, sid=None, new=False):
            def on_update(self):
                self.modified = True
            CallbackDict.__init__(self, initial, on_update)
            self.sid = sid
            self.new = new
            self.modified = False
    
    
    class RedisSessionInterface(SessionInterface):
        serializer = pickle
        session_class = RedisSession
    
        def __init__(self, redis=None, prefix='session:'):
            if redis is None:
                redis = Redis()
            self.redis = redis
            self.prefix = prefix
    
        def generate_sid(self):
            return str(uuid4())
    
        def get_redis_expiration_time(self, app, session):
            if session.permanent:
                return app.permanent_session_lifetime
            return timedelta(days=1)
    
        def open_session(self, app, request):
            sid = request.cookies.get(app.session_cookie_name)
            if not sid:
                sid = self.generate_sid()
                return self.session_class(sid=sid, new=True)
            val = self.redis.get(self.prefix + sid)
            if val is not None:
                data = self.serializer.loads(val)
                return self.session_class(data, sid=sid)
            return self.session_class(sid=sid, new=True)
    
        def save_session(self, app, session, response):
            domain = self.get_cookie_domain(app)
            if not session:
                self.redis.delete(self.prefix + session.sid)
                if session.modified:
                    response.delete_cookie(app.session_cookie_name,
                                           domain=domain)
                return
            redis_exp = self.get_redis_expiration_time(app, session)
            cookie_exp = self.get_expiration_time(app, session)
            val = self.serializer.dumps(dict(session))
            self.redis.setex(self.prefix + session.sid, val,
                             int(redis_exp.total_seconds()))
            response.set_cookie(app.session_cookie_name, session.sid,
                                expires=cookie_exp, httponly=True,
                                domain=domain)


Here is how to enable it:


::

    app = Flask(__name__)
    app.session_interface = RedisSessionInterface()


If you get an attribute error that `total_seconds` is missing it means
you're using a version of Python older than 2.7. In this case you can
use this function as a replacement for the `total_seconds` method:


::

    def total_seconds(td):
        return td.days * 60 * 60 * 24 + td.seconds
.. _https://github.com/andymccurdy/redis-py/blob/master/redis/client.py: https://github.com/andymccurdy/redis-py/blob/master/redis/client.py

