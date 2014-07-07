Simple Pagination
=================

Unless you are using JavaScript to dynamically load more contents pagination is a neat concept to structure many items of information into multiple pages. The idea is that if you have 100 items you show 20 per page and have 5 pages in total then.

Simple Pagination Class
-----------------------
If you are using `Flask-SQLAlchemy`_ you can use the integrated pagination class it provides. Here is a simple pagination class that does roughly the same without the support for slicing SQLAlchemy query objects:

.. code-block:: python

    from math import ceil


    class Pagination(object):

        def __init__(self, page, per_page, total_count):
            self.page = page
            self.per_page = per_page
            self.total_count = total_count

        @property
        def pages(self):
            return int(ceil(self.total_count / float(self.per_page)))

        @property
        def has_prev(self):
            return self.page > 1

        @property
        def has_next(self):
            return self.page < self.pages

        def iter_pages(self, left_edge=2, left_current=2,
                       right_current=5, right_edge=2):
            last = 0
            for num in xrange(1, self.pages + 1):
                if num <= left_edge or \
                   (num > self.page - left_current - 1 and \
                    num < self.page + right_current) or \
                   num > self.pages - right_edge:
                    if last + 1 != num:
                        yield None
                    yield num
                    last = num

URLs and Views
--------------
So how do you declare URLs and views when using Pagination? The Werkzeug routing system which Flask use supports this nicely with route level defaults. You specify a “default” for page 1 for the bare URL and provide an integer wildcard for other pages:

.. code-block:: python

    from flask import redirect

    PER_PAGE = 20

    @app.route('/users/', defaults={'page': 1})
    @app.route('/users/page/<int:page>')
    def show_users(page):
        count = count_all_users()
        users = get_users_for_page(page, PER_PAGE, count)
        if not users and page != 1:
            abort(404)
        pagination = Pagination(page, PER_PAGE, count)
        return render_template('users.html',
            pagination=pagination,
            users=users
        )

Note how this code is returning an 404 error for all pages besides the first page if no items were there to display. This is generally a good idea.

When a user heads to ``/users/page/1`` Flask will redirect him automatically to ``/users/`` to keep the URL unique.

URL Generation Helper
---------------------
Now how can a template generate a URL to a different page without much hassle? Because the only difference from one URL to the other is the page part in it we can provide a little helper function that wraps ``url_for`` to generate a new URL to the same endpoint with a different page:

.. code-block:: python

    def url_for_other_page(page):
        args = request.view_args.copy()
        args['page'] = page
        return url_for(request.endpoint, **args)
    app.jinja_env.globals['url_for_other_page'] = url_for_other_page

Rendering The Pagination
------------------------
So how do you render such a pagination? Here is a simple macro that uses the ``iter_pages`` method of the pagination class to show a simple pagination:

.. code-block:: html

    {% macro render_pagination(pagination) %}
      <div class=pagination>
      {%- for page in pagination.iter_pages() %}
        {% if page %}
          {% if page != pagination.page %}
            <a href="{{ url_for_other_page(page) }}">{{ page }}</a>
          {% else %}
            <strong>{{ page }}</strong>
          {% endif %}
        {% else %}
          <span class=ellipsis>…</span>
        {% endif %}
      {%- endfor %}
      {% if pagination.has_next %}
        <a href="{{ url_for_other_page(pagination.page + 1)
          }}">Next &raquo;</a>
      {% endif %}
      </div>
    {% endmacro %}

.. _Flask-SQLAlchemy: http://packages.python.org/Flask-SQLAlchemy/