Selectively Redirect Back
=========================

It is usually a good idea to redirect the user back to the page they
were viewing after they've logged in or edited their profile.

This is frequently done by either passing a `next` parameter in a form
or by looking at the referer in HTTP headers. Both of these approaches
cause either major headache or failure to redirect properly in case of
multistep login process or static-HTML form validation. Both of them
also require you to validate if the specified referer or form argument
actually belongs to your site.

If none of this scares you, there exists a wonderful snippet `Securely
Redirect Back`_ by Armin Ronacher for such validation.

However, quite often it's much easier and safer to white-list the
locations that we wish to allow our app to automatically return to,
and store the last white-listed location in a signed cookie.
Fortunately, `flask` already does the cookie signing for us, so the
rest is very straightforward.


::

    # This snippet is in public domain.
    # However, please retain this link in your sources:
    # http://flask.pocoo.org/snippets/120/
    # Danya Alexeyevsky
    
    from flask import session, redirect, current_app
    
    class back(object):
        """To be used in views.
    
        Use `anchor` decorator to mark a view as a possible point of return.
    
        `url()` is the last saved url.
    
        Use `redirect` to return to the last return point visited.
        """
    
        cfg = current_app.config.get
        cookie = cfg('REDIRECT_BACK_COOKIE', 'back')
        default_view = cfg('REDIRECT_BACK_DEFAULT', 'index')
    
        @staticmethod
        def anchor(func, cookie=cookie):
            @functools.wraps(func)
            def result(*args, **kwargs):
                session[cookie] = request.url
                return func(*args, **kwargs)
            return result
    
        @staticmethod
        def url(default=default_view, cookie=cookie):
            return session.get(cookie, url_for(default))
    
        @staticmethod
        def redirect(default=default_view, cookie=cookie):
            return redirect(back.url(default, cookie))
    back = back()


Please note, this code looks like class, but actually it's just
namespace, like module. In fact, you can just put it into module
`back.py`, unindent the code, remove `class`, `staticmethod`'s and the
last line, and it works the same way! (Also, you can cut the snippet
down to 8 lines of code, and it still does the same job).

To use it, we decorate our important views with `back.anchor`:


::

    @app.route("/")
    @back.anchor
    def index():
        ...
    
    @app.route("/view/<int:entrynum>")
    @back.anchor
    def view(entrynum):
        ...


It is very important that the `back.anchor` decorators come after all
the `app.route` decorators, since otherwise the latter `app.route`s
will bypass the anchor code.

And then within the views that we want to return from we call
`back.redirect()`:


::

    @app.route("/login")
    def login_stage1():
      ... # stuff that sends us to login_stage2, e.g. OpenID
    
    @app.route("/validate")
    def login_stage2():
      ...
      if everything is ok:
        return back.redirect()
      abort(401)


You can create templates with the link back:


::

    {% extends "base.html" %}
    
    {% block body %}
        <h1>Error!</h1>
        ...
        <a href="{{ back.url() }}">Back</a>
    {% endblock %}


Or else it makes it very easy to convert error pages into flash
messages:


::

    @app.errorhandler(404)
    def not_found():
      flash("Sorry, the requested page is not found", "error")
      return back.redirect()


Of course, in this case you have to add flash messages support to your
templates too.

The snippet is configured with two configuration options:


+ `REDIRECT_BACK_COOKIE` is the name of the cookie in which the last
  location is stored
+ `REDIRECT_BACK_DEFAULT` is the name of the default view if there is
  no cookie
.. _Securely Redirect Back: ../62

