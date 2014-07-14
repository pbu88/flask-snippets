Nicer Errors with Descriptions from Werkzeug
============================================

Writing nice and shiny error messages and assigning them to the
application's error handlers can be a lot of work. If you're happy
with the default descriptions which are provided by werkzeug, you're
done with a few lines of code:


::

    from flask import Markup, render_template
    from werkzeug.exceptions import default_exceptions
    
    def show_errormessage(error):
        desc = error.get_description(flask.request.environ)
        return render_template('error.html',
            code=error.code,
            name=error.name,
            description=Markup(desc)
        ), error.code
    
    for _exc in default_exceptions:
        app.error_handlers[_exc] = show_errormessage
    del _exc


And here the example template:


::

    {% extends "base.html" %}
    {% block title %}Error {{ code }}: {{ name }}{% endblock %}
    {% block body %}
      {{ description}}
    {% endblock %}


Make sure to either wrap the description with `Markup()` or to use the
`|safe` filter in the template on the description as it contains HTML.
.. _https://github.com/mitsuhiko/flask/blob/92dbe3153a9e0c007ecd1987ccd5f3d796c901af/flask/app.py#L1196: https://github.com/mitsuhiko/flask/blob/92dbe3153a9e0c007ecd1987ccd5f3d796c901af/flask/app.py#L1196

