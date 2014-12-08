Dumping post-mortem information on production
=============================================

Out of the box, when an exception is thrown on production (non debug mode), 
Flask will log the traceback and and execute the corresponding handler (either 
custom or default).

In some circumstances, just a traceback is not enough to figure out obscure 
bugs. Info like the session, the request arguments, cookies, etc can be very 
handy, and it would be nice to have them logged too.

There are two straightforward ways to do this. You can subclass Flask class and 
override **handle_exception()** method:

.. code-block:: python

    class MyFlask(Flask):
        
        def handle_exception(self, e):
            # add all necessary log info here
            log.info("dumping session: %s", session)
            log.info("dumping request: %s", request)
            log.info("dumping request args: %s", request.args)
            return super(MyFlask, self).handle_exception(e)

or, you can subscribe to the **got_request_exception()** signal, which is 
emitted inside Flask.handle_exception() method:

.. code-block:: python

    def dump_environment(e, **extra):
        # add all necessary log info here
        log.info("dumping session: %s", session)
        log.info("dumping request: %s", request)
        log.info("dumping request args: %s", request.args)


    from flask import got_request_exception
    got_request_exception.connect(dump_environment)

Keep in mind that the information you want to log depends on your needs, 
**session**, **request** and **request.args** are just examples, you should 
change it for whatever you think is useful to log in your own environment
