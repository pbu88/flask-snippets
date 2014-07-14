Show correct Tracebacks for some Errors triggered inside a Template on
===
---
By Joshua Bronson filed in `Template Tricks`_
This may only be an issue if you're using an environment like Google
App Engine.

If you've ever had an error triggered somewhere in a template and the
traceback displayed in the browser did not point to the line causing
the error (but instead to a line like `{% extends '_base.html' %}`),
this might cause a more helpful traceback to be displayed:


::

    def format_exception(tb):
        return tb.render_as_text()
    # undocumented feature
    app.jinja_env.exception_formatter = format_exception


Add this hack to get the browser to respect the plaintext whitespace:


::

    from flask import make_response
    def format_exception(tb):
        res = make_response(tb.render_as_text())
        res.content_type = 'text/plain'
        return res
    app.jinja_env.exception_formatter = format_exception


If jinja2.debugrenderer is implemented in the future,
`tb.render_as_html` would be even better.

