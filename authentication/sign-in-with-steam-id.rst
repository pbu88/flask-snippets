Sign in with Steam ID
=====================

If you are planning on developing a website where the target audience
are gamers (mainly PC gamers) it might be useful to use `steam`_ for
authentication. If you want to see this demoed check out
`bf3.immersedcode.org`_.

Steam supports OpenID based authentication which is very easy to
implement in Flask with the help of the `Flask-OpenID`_ extension and
your ORM of choice (for instance SQLAlchemy via `Flask-SQLAlchemy`_).


0. Create the App
~~~~~~~~~~~~~~~~~

The application setup with all extensions set up will look like this:


::

    from flask import Flask, redirect, session, json, g
    from flaskext.sqlalchemy import SQLAlchemy
    from flaskext.openid import OpenID
    
    app = Flask(__name__)
    app.config.from_pyfile('settings.cfg')
    db = SQLAlchemy(app)
    oid = OpenID(app)




1. Register the API
~~~~~~~~~~~~~~~~~~~

In order to use the steam API you have to register for an API key.
That's easy. Just to go the `registration website`_, log in with your
personal steam account and enter your domain name. You will instantly
receive an API key which you can add into your config:


::

    STEAM_API_KEY = 'ABCDEFG-12345'




2. Use Flask-SQLAlchemy for the User Model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First we need to create a user model. Flask-SQLAlchemy makes this
pretty easy. We want to store the steam ID for sign-in and the
nickname. Because the nick name can be changed by the user we will
update it whenever the user signs in.

Furthermore we provide a method that looks up a user by steam ID and
will create a new one if a new ID came along.


::

    class User(db.Model):
        id = db.Column(db.Integer, primary_key=True)
        steam_id = db.Column(db.String(40))
        nickname = db.String(80)
    
        @staticmethod
        def get_or_create(steam_id):
            rv = User.query.filter_by(steam_id=steam_id).first()
            if rv is None:
                rv = User()
                rv.steam_id = steam_id
                db.session.add(rv)
            return rv




3. Bridge the Steam API
~~~~~~~~~~~~~~~~~~~~~~~

In order to get a user's nickname we have to write a small function
that asks the steam API for a user's public information:


::

    import urllib2
    
    def get_steam_userinfo(steam_id):
        options = {
            'key': app.config['STEAM_API_KEY'],
            'steamids': steam_id
        }
        url = 'http://api.steampowered.com/ISteamUser/' \
              'GetPlayerSummaries/v0001/?%s' % url_encode(options)
        rv = json.load(urllib2.urlopen(url))
        return rv['response']['players']['player'][0] or {}




4. Use Flask-OpenID for sign-in
-------------------------------

Login code is simple now. We just have to redirect the user to the
steam OpenID signin page and register an `after_login` handler that
refreshes the nickname and binds the user to the session:


::

    import re
    
    _steam_id_re = re.compile('steamcommunity.com/openid/id/(.*?)$')
    
    @app.route('/login')
    @oid.loginhandler
    def login():
        if g.user is not None:
            return redirect(oid.get_next_url())
        return oid.try_login('http://steamcommunity.com/openid')
    
    @oid.after_login
    def create_or_login(resp):
        match = _steam_id_re.search(resp.identity_url)
        g.user = User.get_or_create(match.group(1))
        steamdata = get_steam_userinfo(g.user.steam_id)
        g.user.nickname = steamdata['personaname']
        db.session.commit()
        session['user_id'] = g.user.id
        flash('You are logged in as %s' % g.user.nickname)
        return redirect(oid.get_next_url())




5. Logout and Session Handling
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to automatically pull in the current user each request we can
use a `before_request` handler. We will also need a logout link:


::

    @app.before_request
    def before_request():
        g.user = None
        if 'user_id' in session:
            g.user = User.query.get(session['user_id'])
    
    @app.route('/logout')
    def logout():
        session.pop('user_id', None)
        return redirect(oid.get_next_url())




6. The Legal Stuff
~~~~~~~~~~~~~~~~~~

Application using steam for signup have to do use certain logos for
the signup and put a link to steam into the footer. More information
can be found here: `steamcommunity.com/dev`_
.. _Flask-OpenID: http://packages.python.org/Flask-OpenID/
.. _Flask-SQLAlchemy: http://packages.python.org/Flask-SQLAlchemy/
.. _registration website: http://steamcommunity.com/dev/apikey
.. _bf3.immersedcode.org: http://bf3.immersedcode.org/
.. _steam: http://steampowered.com/
.. _steamcommunity.com/dev: http://steamcommunity.com/dev

