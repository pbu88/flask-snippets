Counting Online Users with Redis
================================

Sometimes you might want to show on the index page what users are
online. Sets in redis are perfect for this. You can take the current
time since 1970 in seconds, divide it by 60 and build a key based on
that, then add a user to that set. Make the set expire after the
maximum number of seconds you give a user in activity and when you
want to query all active users you just build a union of the keys of
the last N minutes.


Connecting to Redis
-------------------

In case you don't have a connection to redis yet, open with with those
two simple lines:


::

    from redis import Redis
    redis = Redis()


A redis instance is thread safe so you can just keep this on the
global level and use it directly. If you want to connect to a
different redis instance just pass the address to the constructor.
More in the pyredis docs.


Helpers
-------


::

    import time
    from datetime import datetime
    
    ONLINE_LAST_MINUTES = 5
    
    def mark_online(user_id):
        now = int(time.time())
        expires = now + (app.config['ONLINE_LAST_MINUTES'] * 60) + 10
        all_users_key = 'online-users/%d' % (now // 60)
        user_key = 'user-activity/%s' % user_id
        p = redis.pipeline()
        p.sadd(all_users_key, user_id)
        p.set(user_key, now)
        p.expireat(all_users_key, expires)
        p.expireat(user_key, expires)
        p.execute()
    
    def get_user_last_activity(user_id):
        last_active = redis.get('user-activity/%s' % user_id)
        if last_active is None:
            return None
        return datetime.utcfromtimestamp(int(last_active))
    
    def get_online_users():
        current = int(time.time()) // 60
        minutes = xrange(app.config['ONLINE_LAST_MINUTES'])
        return redis.sunion(['online-users/%d' % (current - x)
                             for x in minutes])




Marking a user as Online
------------------------

For testing purposes we can just use the remote address (IP) of the
connecting user as user id and mark that online. Normally you would
use your actual user id or usernames here.


::

    @app.before_request
    def mark_current_user_online():
        mark_online(request.remote_addr)




Displaying Online Users
-----------------------

And here a very simple view that shows all active users in the last
`ONLINE_LAST_MINUTES` minutes.


::

    from flask import Response
    
    @app.route('/online')
    def index():
        return Response('Online: %s' % ', '.join(get_online_users()),
                        mimetype='text/plain')


To check if a user is online or when he was online the last time you
can use the `get_user_last_activity` function which returns the time
the user was online the last time or `None` if it's too long in the
past.

