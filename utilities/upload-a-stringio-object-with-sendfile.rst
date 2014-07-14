Upload a StringIO object with send_file
=======================================

Sometimes, you want to avoid the creation of a file to send this file
to the client.

So, in this case, you can use a StringIO object with the send_file
helper.

Here is an example:


::

    #!/usr/bin/env python
    
    # Thanks to Dan Jacob for a part of the code !
    
    from flask import Flask, send_file
    import StringIO
    
    app = Flask(__name__)
    
    @app.route('/')
    def index():
        strIO = StringIO.StringIO()
        strIO.write('Hello from Dan Jacob and Stephane Wirtel !')
        strIO.seek(0)
        return send_file(strIO,
                         attachment_filename="testing.txt",
                         as_attachment=True)
            
    app.run(debug=True)

