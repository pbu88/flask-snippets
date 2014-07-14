Flask-Mail with google apps
===========================

A simple example on how to use flask-mail with google or google apps
email accounts. Other sources:

-Flask-mail docs: `http://packages.python.org/flask-mail/`_

-Google support:
`http://support.google.com/mail/bin/answer.py?hl=en&answer=78799`_

@kfk


::

    from flask import Flask
    from flaskext.mail import Mail, Message
    
    app =Flask(__name__)
    mail=Mail(app)
    
    app.config.update(
    	DEBUG=True,
    	#EMAIL SETTINGS
    	MAIL_SERVER='smtp.gmail.com',
    	MAIL_PORT=465,
    	MAIL_USE_SSL=True,
    	MAIL_USERNAME = 'you@google.com',
    	MAIL_PASSWORD = 'GooglePasswordHere'
    	)
    
    mail=Mail(app)
    
    @app.route("/")
    def index():
    	msg = Message(
                  'Hello',
    	       sender='you@dgoogle.com',
    	       recipients=
                   ['recipient@recipient_domain.com'])
    	msg.body = "This is the email body"
    	mail.send(msg)
    	return "Sent"
    
    if __name__ == "__main__":
        app.run()
.. _http://packages.python.org/flask-mail/: http://packages.python.org/flask-mail/
.. _answer=78799: http://support.google.com/mail/bin/answer.py?hl=en&answer=78799

