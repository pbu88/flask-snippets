Catch-All URL
=============

A simple way to create a Catch-All function which serves every URL
including / is to chain two route filters. One for the root path '/'
and one including a *path* placeholder for the rest.

We can't just use one route filter including a *path* placeholder
because each placeholder must at least catch one character.


::

    from flask import Flask
    app = Flask(__name__)
    
    @app.route('/', defaults={'path': ''})
    @app.route('/<path:path>')
    def catch_all(path):
        return 'You want path: %s' % path
    
    if __name__ == '__main__':
        app.run()


A little demonstration..

::

    % curl 127.0.0.1:5000          # Matches the first rule
    You want path:  
    % curl 127.0.0.1:5000/foo/bar  # Matches the second rule
    You want path: foo/bar


Hope it helps!

