Using TurboMail with Flask
==========================

`TurboMail`_ is an email package originally bundled with TurboGears
but now existing as a standalone library.

It provides convenient functionality for configuring and sending
emails, for example for testing, multipart messages, etc.

It's quite straightforward to set up TurboMail for your Flask
application. One thing you need to take care of is ensuring that the
interface is set up so that when the current process exits, any unsent
emails are cleanly dispatched. This is achieved using the standard
atexit module.


::

    import atexit
    
    from turbomail.control import interface
    from turbomail.message import Message
    
    from flask import Flask
    
    # pass in dict of config options
    
    interface.start({'mail.on' : True})
    
    # ensures interface cleanly shutdown when process exits
    
    atexit.register(interface.stop, force=True)
    
    app = Flask(__name__)
    
    @app.route("/")
    def index():
        # send an email
        msg = Message("from@example.com",
                      "to@example.com",
                      "a subject")
        msg.plain = "body of message"
        msg.send()
.. _TurboMail: http://www.python-turbomail.org/

