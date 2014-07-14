Securely Redirect Back
======================

A common pattern with form processing is to automatically redirect
back to the user. There are usually two ways this is done: by
inspecting a `next` URL parameter or by looking at the HTTP referrer.
Unfortunately you also have to make sure that users are not redirected
to malicious attacker's pages and just to the same host. If you are
using Flask-WTF there is a nicer way: `Redirects with Flask-WTF`_.

A function that ensures that a redirect target will lead to the same
server is here:


::

    from urlparse import urlparse, urljoin
    from flask import request, url_for
    
    def is_safe_url(target):
        ref_url = urlparse(request.host_url)
        test_url = urlparse(urljoin(request.host_url, target))
        return test_url.scheme in ('http', 'https') and \
               ref_url.netloc == test_url.netloc


A simple way to to use it is by writing a `get_redirect_target`
function that looks at various hints to find the redirect target:


::

    def get_redirect_target():
        for target in request.values.get('next'), request.referrer:
            if not target:
                continue
            if is_safe_url(target):
                return target


Since we don't want to redirect to the same page we have to make sure
that the actual back redirect is slightly different (only use the
submitted data, not the referrer). Also we can have a fallback there:


::

    def redirect_back(endpoint, **values):
        target = request.form['next']
        if not target or not is_safe_url(target):
            target = url_for(endpoint, **values)
        return redirect(target)


It will tried to use next and the referrer first and fall back to a
given endpoint. You can then use it like this in the views:


::

    @app.route('/login', methods=['GET', 'POST'])
    def login():
        next = get_redirect_target()
        if request.method == 'POST':
            # login code here
            return redirect_back('index')
        return render_template('index.html', next=next)


The `or` is important so that we have a redirect target if all hints
fail (in this case the index page).

In the template you have to make sure to relay the redirect target:


::

    <form action="" method=post>
      <dl>
        <dt>Username:
        <dd><input type=text name=username>
        <dt>Password:
        <dd><input type=password name=password>
      </dl>
      <p>
        <input type=submit value=Login>
        <input type=hidden value="{{ next or '' }}" name=next>
    </form>


The `or` here is just here to make `None` become an empty string.
.. _Redirects with Flask-WTF: /snippets/63/

