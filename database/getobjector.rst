get_object_or_404
=================


::

    from sqlalchemy.orm import exc
    from werkzeug.exceptions import abort
    
    def get_object_or_404(model, *criterion):
        try:
            return model.query.filter(*criterion).one()
        except exc.NoResultFound, exc.MultipleResultsFound:
            abort(404)


Example:


::

    board = get_object_or_404(Board, Board.slug == slug)


or


::

    user = get_object_or_404(User, User.id == id)

