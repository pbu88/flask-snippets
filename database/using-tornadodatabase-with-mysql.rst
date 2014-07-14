Using tornado.database with MySQL
=================================

The Tornado framework has a handy lightweight wrapper around MySQLdb:
`tornado.database`_

If you are using MySQL and SQLAlchemy is too much overhead this is a
nicer way of working with SQL directly than using DB API, as it
includes a number of convenience methods and defaults, for example
hiding the underlying cursors and accessing columns through dict or
object syntax. The code is small enough that it should be easy to
modify to your requirements.

Example usage in Flask:


::

    from tornado.database import Connection
    from flask import Flask, g, render_template
    
    app = Flask(__name__)
    
    import config 
    
    @app.before_request
    def connect_db():
        g.db = Connection(config.DB_HOST,
                          config.DB_NAME,
                          config.DB_USER,
                          config.DB_PASSWD)
    
    @app.after_request
    def close_connection(response):
        g.db.close()
        return response
    
    @app.route("/")
    def index():
        newsitems = g.db.iter("select * from newsitems")
        return render_template("index.html", newsitems=newsitems)



::

    {% for item in newsitems %}
    <h3>{{ item.title }}</h3>
    {% endfor %}


You can get much of the same functionality in SQLAlchemy 0.6 using
NamedTuples, without using the ORM:


::

    from sqlalchemy import create_engine
    
    @app.before_request
    def connect_db():
        g.db = create_engine(config.DB_URI)
    
    @app.route("/")
    def index():
        newsitems = g.db.execute("select * from newsitems")
        # now you can do newsitem.title...
.. _https://github.com/bdarnell/torndb: https://github.com/bdarnell/torndb

