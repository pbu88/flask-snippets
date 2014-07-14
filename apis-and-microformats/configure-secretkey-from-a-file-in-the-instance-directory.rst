Configure SECRET_KEY from a file in the instance directory.
===========================================================


::

    import sys
    import os.path
    
    
    def install_secret_key(app, filename='secret_key'):
        """Configure the SECRET_KEY from a file
        in the instance directory.
    
        If the file does not exist, print instructions
        to create it from a shell with a random key,
        then exit.
    
        """
        filename = os.path.join(app.instance_path, filename)
        try:
            app.config['SECRET_KEY'] = open(filename, 'rb').read()
        except IOError:
            print 'Error: No secret key. Create it with:'
            if not os.path.isdir(os.path.dirname(filename)):
                print 'mkdir -p', os.path.dirname(filename)
            print 'head -c 24 /dev/urandom >', filename
            sys.exit(1)


Usage example, after deploying to a new machine:


::

    $ ./run.py
    Error: No secret key. Create it with:
    mkdir -p /home/simon/exampleapp/instance
    head -c 24 /dev/urandom > /home/simon/exampleapp/instance/secret_key
    # Copy-and-paste the two commands
    $ ./run.py 
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

