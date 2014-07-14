Simple insertion and row id
===========================

This function lets you abstract the insertion into the database and
get it's last id. This example uses sqlite3.


::

    def insert(table, fields=(), values=()):
        # g.db is the database connection
        cur = g.db.cursor()
        query = 'INSERT INTO %s (%s) VALUES (%s)' % (
            table,
            ', '.join(fields),
            ', '.join(['?'] * len(values))
        )
        cur.execute(query, values)
        g.db.commit()
        id = cur.lastrowid
        cur.close()
        return id

