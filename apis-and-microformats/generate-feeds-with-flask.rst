Generating Feeds With Flask
===========================

Atom is an easy way to let other people subscribe to changes on your website. For example people can subscribe to the recent additions of content on your website.

Assuming you already have a way to access that information (database query for most recent blog posts for example) generating an Atom feed is easy as pie.

All you need is the AtomFeed class from the Werkzeug contrib package (this is there for you in any Flask application). Just create a view function like this:

.. code-block:: python

    from urlparse import urljoin
    from flask import request
    from werkzeug.contrib.atom import AtomFeed
    
    
    def make_external(url):
        return urljoin(request.url_root, url)
    
    
    @app.route('/recent.atom')
    def recent_feed():
        feed = AtomFeed('Recent Articles',
                        feed_url=request.url, url=request.url_root)
        articles = Article.query.order_by(Article.pub_date.desc()) \
                          .limit(15).all()
        for article in articles:
            feed.add(article.title, unicode(article.rendered_text),
                     content_type='html',
                     author=article.author.name,
                     url=make_external(article.url),
                     updated=article.last_update,
                     published=article.published)
        return feed.get_response()

Some things to keep in mind: published is optional, updated is not. A feed should not contain all items but only the most recent ones and it should be ordered by date. Also the URLs should be external, to ensure that you can use urlparse.urljoin with the current request's root URL as first argument and the one to externalize as second.

Extra attention: if the content (rendered_text) is marked as Markup you will have to convert it to standard unicode. The snippet above does that in any situation, because it cannot do any harm.

To tell browsers about a feed on a page, you can embed the following code in the <head> section of your site:

.. code-block:: html

    <link href="{{ url_for('recent_feed') }}"
          rel="alternate"
          title="Recent Changes" 
          type="application/atom+xml">

For more information, consult the Werkzeug documentation: `Atom Syndication`_.


*This snippet can be used freely for anything you like. Consider it public domain.*

.. _Atom Syndication: http://werkzeug.pocoo.org/docs/contrib/atom/