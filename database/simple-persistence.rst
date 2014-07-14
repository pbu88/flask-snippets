Simple persistence
==================

People will burn me at the stake for suggesting this ☺ and I must warn
you that this is not for large datasets or high-traffic sites; but for
that tiny CMS you wrote for grandma's recipes it may do fine. It
probably does not work with multiple processes.

The upside is that you can seamlessly persist almost any object
without any dependencies or daemons and with very little code.


::

    from __future__ import with_statement
    
    import shelve
    from os import path
    from cPickle import HIGHEST_PROTOCOL
    from contextlib import closing
    
    from flask import Flask
    
    
    SHELVE_DB = 'shelve.db'
    
    
    app = Flask(__name__)
    app.config.from_object(__name__)
    
    db = shelve.open(path.join(app.root_path, app.config['SHELVE_DB']),
                     protocol=HIGHEST_PROTOCOL, writeback=True)
    
    
    @app.route('/<message>')
    def write_and_list(message):
        db.setdefault('messages', [])
        db['messages'].append(message)
        return app.response_class('\n'.join(db['messages']),
                                  mimetype='text/plain')
    
    
    with closing(db):
        app.run()


The db object works like a dict and as you see in our view we can add
a normal list to it and don't need to do anything special to persist
changes.

If you want something similar that scales better have a look at
`ZODB`_. I haven't actually used ZODB myself and there seem to be
controversy regarding its utility, but it is in use by most Zope sites
out there.
.. _ZODB: http://zodb.org/
.. _Flask-ZODB: http://packages.python.org/Flask-ZODB/

