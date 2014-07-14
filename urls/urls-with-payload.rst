URLs with Payload
=================

Sometimes it's useful if uses get generated links that trigger a
specific behavior. For instance you want to send users to a URL that
activates their account. Because that URL must only be used by the
receiver and only once, how could you do that?

A simple way is to use cryptographic signing with the help of the
`It's Dangerous`_ module. It allows you to put arbitrary JSON into a
URL and signing it.

Example:


::

    from flask import abort, redirect, flash
    from itsdangerous import URLSafeSerializer, BadSignature
    
    def get_serializer(secret_key=None):
        if secret_key is None:
            secret_key = app.secret_key
        return URLSafeSerializer(secret_key)
    
    @app.route('/users/activate/<payload>')
    def activate_user(payload):
        s = get_serializer()
        try:
            user_id = s.loads(payload)
        except BadSignature:
            abort(404)
    
        user = User.query.get_or_404(user_id)
        user.activate()
        flash('User activated')
        return redirect(url_for('index'))


So how do you generate that URL? Very similar:


::

    def get_activation_link(user):
        s = get_serializer()
        payload = s.dumps(user.id)
        return url_for('activate_user', payload=payload, _external=True)


The URL generated will look something like this:
`http://example.com/users/activate/NDI.qufkoGPGURs0UuFTludpcHLKa20`_

The link above can be used multiple times however. So how do you limit
it to being only useful once? Well for most actions that is not
necessary as it can only be triggered once anyways (for instance you
can only unsubscribe once, activate once etc.). However under certain
circumstances that is not okay (for example to redeem a voucher).

In that situation you cannot avoid having some information about
redeemed values on the server. With the signing however you can easily
make sure a voucher is only valid for the user it was provided for
without having to keep that information on the server:


::

    @app.route('/voucher/redeem/<payload>')
    def redeem_voucher(payload):
        s = get_serializer()
        try:
            user_id, voucher_id = s.loads(payload)
        except BadSignature:
            abort(404)
    
        user = User.query.get_or_404(user_id)
        voucher = Voucher.query.get_or_404(voucher_id)
        voucher.redeem_for(user)
        flash('Voucher redeemed')
        return redirect(url_for('index'))
    
    def get_redeem_link(user, voucher):
        s = get_serializer()
        payload = s.dumps([user.id, voucher.id])
        return url_for('redeem_voucher', payload=payload, 
                       _external=True)
.. _It's Dangerous: http://packages.python.org/itsdangerous/
.. _http://example.com/users/activate/NDI.qufkoGPGURs0UuFTludpcHLKa20: http://example.com/users/activate/NDI.qufkoGPGURs0UuFTludpcHLKa20

