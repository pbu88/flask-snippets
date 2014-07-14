Handling URLs containing slash '/' character
============================================

Creating RESTful web services with Flask is a breeze, until you run
into an issue where you want to put an identifier containing slash
character ' / ' into the URI.

Let's say that you have an app that generates a list of urls for a
list of products:


::

    {% for p in products %}
      <a href="{{url_for('product', code=p.code)}}">{{ p.title }}</a><br>
    {% endfor %}


View function then takes the code from the URL, looks up the product
from database and renders template:


::

    @app.route('/product/<code>')
    def product(code):
        product = Product.query.filter_by(code=code).first()
        return render_template('product', product=product)


It all works fine until one of the product codes contains slash and
the URL */product/123/foo* ends up giving an error 404.

But all is not just lost. If your first idea is to write a clever
escape/unescape function that works automatically with both template
and view code, then STOP!

This can be fixed by just using a path: as converter keyword to the
URL argument:


::

    @app.route('/product/<path:code>')


Happy Slashing ! :)

