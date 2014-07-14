Getting an object from a SQLAlchemy model or abort
==================================================

A simple shortcut which let you returning an `abort code`_ when the
wanted object is not found.

This kind of trick is already integrated in the `Flask-SQLAlchemy
extension`_ (with the get_or_404() method ) so it's just usefull if
you're using SQLAlchemy natively...

Here's the code:


::

    def get_or_abort(model, object_id, code=404):
        """
        get an object with his given id or an abort error (404 is the default)
        """
        result = model.query.get(object_id)
        return result or abort(code)


And now how to use it in your app:


::

    def theme_detail(theme_id):
        # shows a theme
        theme = get_or_abort(Theme, theme_id)
        return render_template('theme_detail.html', theme=theme)
.. _Flask-SQLAlchemy extension: http://packages.python.org/Flask-SQLAlchemy/
.. _abort code: http://flask.pocoo.org/docs/patterns/errorpages/

