Agnostic Database Migrations
============================

Overview
--------

Agnostic is a light-weight, easy-to-learn, and flexible database migration tool
in which migration scripts are written in pure SQL. It is agnostic towards
database, programming language, and object relational mapper (ORM).

Super Quick Start
-----------------

Here is an absurdly brief introduction to Agnostic:

.. code:: bash

    ~/myapp $ pip3 install agnostic # Notice: python3 only!
    <snip>

    ~/myapp $ mkdir migrations

    ~/myapp $ agnostic -t postgres -u myuser -s myapp bootstrap
    Migration table created.

    ~/myapp $ cat > migrations/add_cell_phone.sql
    ALTER TABLE customer ADD cell_phone VARCHAR(255);
    ^D

    ~/myapp $ cat > migrations/add_nickname.sql
    ALTER TABLE customer ADD nickname VARCHAR(255);
    ^D

    ~/myapp $ agnostic -t postgres -u myuser -s myapp migrate
    Backing up schema "myapp" to "/tmp/tmpm8glpgaa".
    About to run 2 migrations in schema "myapp":
     * Running migration add_cell_phone (1/2)
     * Running migration add_nickname (2/2)
    Migrations completed successfully.
    Removing backup "/tmp/tmpm8glpgaa".

For a not-quite-as-quick-but-still-pretty-quick start, please refer to the
`full documentation <http://agnostic.readthedocs.org/en/latest/index.html>`_.
