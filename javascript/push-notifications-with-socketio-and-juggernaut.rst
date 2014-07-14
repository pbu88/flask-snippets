Push Notifications with socket.io and Juggernaut
================================================

Talking from the client to the server is easy. Just use
`XMLHttpRequest` to send an HTTP request to the server and you're
done. But what about talking from the server to the client in
realtime?

Turns out WSGI and traditional frameworks are not exactly suited for
that. However they don't have to since you can keep the web socket
connection around somewhere else.

This is how `Juggernaut`_ works. It's a server written with node that
uses redis to communicate with your server side application and
websockets, long polling or flash to talk to the client.

So how does it work? First of all you need to have Juggernaut
installed and running (also make sure to have redis running):

::

    $ npm install -g juggernaut
    $ juggernaut


For talking to juggernaut from Python you can use redis directly or
the `juggernaut` client library:

::

    pip install juggernaut


At that point you can use juggernaut to talk to your clients. On the
client side you need to include the juggernaut driver script and
subscribe to a channel:


::

    <script type=text/javascript
      src=http://localhost:8080/application.js></script>
    <script type=text/javascript>
      var jug = new Juggernaut();
      jug.subscribe('channel', function(data) {
        alert('Got message: ' + data);
      });
    </script>


In this case Juggernaut is running on `localhost:8080`. It makes sense
to make this configurable.

To send a message now with Python you can do it like this:


::

    >>> from juggernaut import Juggernaut
    >>> jug = Juggernaut()
    >>> jug.publish('channel', 'The message')


If redis is running somewhere else you will need to pass a redis
instance to `Juggernaut` and the `juggernaut` daemon.

For a full fledged example see `Flask-Pastebin`_.
.. _Flask-Pastebin: https://github.com/mitsuhiko/flask-pastebin
.. _http://blog.alexmaccaw.com/killing-a-library: http://blog.alexmaccaw.com/killing-a-library
.. _Juggernaut: https://github.com/maccman/juggernaut

