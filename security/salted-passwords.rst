Salted Passwords
================

When you have user accounts and you give them passwords you really
don't want to store them in the database unhashed. However just
hashing the passwords is barely more secure because of `Rainbow
table`_ attacks. What you want to do is to salt the passwords which
means that instead of just hashing the password you hash the password
+ a salt. And you also don't just want to concatenate them but use
HMAC. And because that's common and easy to make wrong, Werkzeug
provides a helper for that which also generates a hash for you.

Here is how it works. The following example assumes that you use some
class for your user object:


::

    from werkzeug.security import generate_password_hash, \
         check_password_hash
    
    class User(object):
    
        def __init__(self, username, password):
            self.username = username
            self.set_password(password)
    
        def set_password(self, password):
            self.pw_hash = generate_password_hash(password)
    
        def check_password(self, password):
            return check_password_hash(self.pw_hash, password)


And here is how it works:


::

    >>> me = User('John Doe', 'default')
    >>> me.pw_hash
    'sha1$Z9wtkQam$7e6e814998ab3de2b63401a58063c79d92865d79'
    >>> me.check_password('default')
    True
    >>> me.check_password('defaultx')
    False
.. _http://en.wikipedia.org/wiki/SHA-1: http://en.wikipedia.org/wiki/SHA-1
.. _http://www.unlimitednovelty.com/2012/03/dont-use-bcrypt.html: http://www.unlimitednovelty.com/2012/03/dont-use-bcrypt.html
.. _Rainbow table: http://en.wikipedia.org/wiki/Rainbow_table

