Generating Slugs
================

When you want to have nice readable URLs it's usually a good idea to
use the title or name of an object in the URL.

For example if you have a post entitled "Hello World!" the slug could
be "hello-world". The term slug probably originally came from
publishing but ever since Django used it on the core documentation
many web developers know what it means.

Because ASCII is the common subset nearly everyone around the world
can input with his keyboard it makes a lot of sense to limit the URL
to that. The following function tries its best to generate an ASCII
only and lowercase slug from a word:


::

    import re
    
    _punct_re = re.compile(r'[\t !"#$%&\'()*\-/<=>?@\[\\\]^_`{|},.]+')
    
    
    def slugify(text, delim=u'-'):
        """Generates an ASCII-only slug."""
        result = []
        for word in _punct_re.split(text.lower()):
            word = word.encode('translit/long')
            if word:
                result.append(word)
        return unicode(delim.join(result))


As you might have spotted, the code above is using Jason Kirkland's
translit codec: get `translitcodec from PyPI`_.

Wha translitcodec does is converting umlauts like "ü" to "ue" instead
of their improper "u" equivalent. If you don't need that you can also
take advantage of the builtin unicodedata module:


::

    import re
    from unicodedata import normalize
    
    _punct_re = re.compile(r'[\t !"#$%&\'()*\-/<=>?@\[\\\]^_`{|},.]+')
    
    def slugify(text, delim=u'-'):
        """Generates an slightly worse ASCII-only slug."""
        result = []
        for word in _punct_re.split(text.lower()):
            word = normalize('NFKD', word).encode('ascii', 'ignore')
            if word:
                result.append(word)
        return unicode(delim.join(result))


Please be advised that neither of the above functions will work
correctly on asian signs. In both cases the return value of that
function will most likely be an empty string. Prepare to have a backup
slug in that situation (like the post id or something). You also might
want to fall back to a non-ASCII slug because modern browsers will
display non-ASCII chars in the URL as long as the URL is UTF-8
encoded.

If you expect a lot of Asian characters or want to support them as
well you can instead use the `Unidecode`_ package that handles them as
well:


::

    import re
    from unidecode import unidecode
    
    _punct_re = re.compile(r'[\t !"#$%&\'()*\-/<=>?@\[\\\]^_`{|},.]+')
    
    
    def slugify(text, delim=u'-'):
        """Generates an ASCII-only slug."""
        result = []
        for word in _punct_re.split(text.lower()):
            result.extend(unidecode(word).split())
        return unicode(delim.join(result))
.. _Unidecode: http://pypi.python.org/pypi/Unidecode/0.04.1
.. _https://gist.github.com/1428479: https://gist.github.com/1428479
.. _translitcodec from PyPI: http://pypi.python.org/pypi/translitcodec

