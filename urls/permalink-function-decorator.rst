Permalink function decorator
============================

This is a function decorator that wraps a function that returns
arguments to flask.url_for. This is handy when modeling objects in an
ORM and you need to reference their absolute URL. Here is an example
of that in pseudocode:


::

    class User(Model):
        username = StringField(...)
        email = StringField(...)
        
        @permalink
        def absolute_url(self):
            return 'profiles', {'username':self.username}


Assuming that you have a 'profiles' endpoint setup, when
User.absolute_url is called it will return the URL to the 'profiles'
endpoint with the username values (producing something like
/profiles/username). If there is any error in building the URL, the
function will return None instead.

Here is the code:


::

    from flask import url_for
    from werkzeug.routing import BuildError
    
    def permalink(function):
        def inner(*args, **kwargs):
            endpoint, values = function(*args, **kwargs)
            try:
                return url_for(endpoint, **values)
            except BuildError:
                return
        return inner

