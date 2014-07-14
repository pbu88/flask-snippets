Admin Blueprint
===============

A common need of a website is to have an admin interface that is only
accessible to a subset of users (ex. users with an admin role).
Putting this code in the same modules (files) as the rest of the site
can clutter things quickly.

Here is some boilerplate that leverages the power of `Blueprints`_ to
separate the admin views/forms/etc from the rest of the site, as well
as conveniently handle the need to restrict all requests to admin
views.

*__init__.py*


::

    from flask import Flask
    import admin
    app = Flask(__name__)
    app.register_blueprint(admin.bp, url_prefix='/admin')


*admin/__init__.py*


::

    from flask import Blueprint
    from flask import redirect, request
    from google.appengine.api import users
    
    bp = Blueprint('admin', __name__)
    
    @bp.before_request
    def restrict_bp_to_admins():
        if not users.is_current_user_admin():
            return redirect(users.create_login_url(request.url))


This example makes use of Google App Engine's `User API`_ for
authentication/authorization, but could easily be modified to support
another authorization mechanism.

You may also prefer to abort the request with a HTTP 403 (
`abort(403)`) instead of returning the user back to the login page.
.. _Blueprints: http://flask.pocoo.org/docs/blueprints/
.. _User API: http://code.google.com/appengine/docs/python/users/

