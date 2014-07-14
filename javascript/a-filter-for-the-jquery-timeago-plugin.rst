A Filter for the jQuery Timeago Plugin
======================================

`Timeago`_ is a jQuery plugin that makes it easy to support
automatically updating fuzzy timestamps (e.g. “4 minutes ago” or
“about 1 day ago”). It automatically keeps them updated and only needs
a very basic `span` tag or something similar with a certain class and
title attribute.

For instance


::

    <span class=timeago title="2008-07-17T09:24:17Z">...</span>


turns into something like this:


::

    <span class=timeago title="July 17, 2008">2 years ago</span>


Because generating the HTML can be quite annoying to do by hand, here
is a simple filter that does that for you:


::

    @app.template_filter()
    def datetimeformat(datetime, timeago=True):
        readable = datetime.strftime('%Y-%m-%d @ %H:%M')
        if not timeago:
            return readable
        iso_format = datetime.strftime('%Y-%m-%dT%H:%M:%SZ')
        return '<span class=timeago title="%s">%s</span>' % (
            iso_format,
            readable
        )


And here is how you use it:


::

    <p class=date>Date: {{ the_date|datetimeformat }}


Don't forget to include the Timeago plugin and to initialize it:


::

    $(function() {
      $('span.timeago').timeago();
    });
.. _Timeago: http://timeago.yarp.com/

