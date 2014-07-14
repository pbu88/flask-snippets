Triggering the debugger on purpose
==================================

The Flask (Werkzeug) debugger is really good, but it only triggers on
exceptions — sometimes it could be useful for debugging behavior that
isn't raising anything. In these situations you can simply raise an
exception intentionally.


::

    @app.route('/')
    def index():
       do_something_wrong()
       raise
       return 'Ohnoes'


This use of `raise` is actually wrong, which means it… raises an
exception. ☺

If you're afraid you might accidentally leave this in the code, here's
one that only enters the debugger in debug-mode:


::

    assert app.debug == False

