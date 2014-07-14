Share your Local Server with a Friend
=====================================

`Localtunnel`_ is a neat tool you can use to quickly share your local
Flask server with a friend.

To install Localtunnel, open a terminal and run the following command:

::

        sudo gem install localtunnel


Then, with Flask running at `http://localhost:5000`_, open a new
Terminal window and type

::

        localtunnel 5000
        Port 5000 is now publicly accessible from http://54xy.localtunnel.com ...


*(Get a* `gem: command not found` *error? Download RubyGems*`here`_
*.)*

If you load the URL given in the localtunnel output in your browser,
you should see your Flask app. It's actually being loaded from your
own computer!
.. _here: http://rubygems.org/pages/download
.. _Localtunnel: http://progrium.com/localtunnel/
.. _http://localhost:5000: http://localhost:5000

