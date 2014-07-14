Testing Issues with SQLAlchemy
==============================

You would like to perform some tests to ensure that you can insert and
query for objects.

Insertions work, but when you try to perform a query in your tests you
get a problem like this:


::

    Failed example:
        len(MyObject.query.all())
    Exception raised:
        Traceback (most recent call last):
          File ".../lib/python2.6/doctest.py", line 1248, in __run
            compileflags, 1) in test.globs
          File "<doctest myapp.MyObject[4]>", line 1, in <module>
            len(MyObject.query.all())
        AttributeError: 'NoneType' object has no attribute 'all'


What you need to do is ensure that you have initialized a request
context for your tests. This can be done by:


::

    app.test_request_context().push()


Now when you run your tests and query them they should work.

