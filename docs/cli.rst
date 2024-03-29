Command Line
============

Global Options
--------------

The command line interface is built around a set of subcommands. Each of these
subcommands supports a number of global options. To list the global options and
subcommands:

.. code:: bash

    ~ $ agnostic
    Usage: agnostic [OPTIONS] COMMAND [ARGS]...

      Agnostic database migrations: upgrade schemas, keep your sanity.

    Options:
      -t, --type <type>           Type of database.  [required]
      -h, --host <host>           Database hostname. (default: localhost)
                                  [required]
      -p, --port <port>           Database port #. If omitted, a default port will
                                  be selected based on <type>.
      -u, --user <user>           Database username.  [required]
      --password <pass>           Database password. If omitted, the password must
                                  be entered on stdin.  [required]
      -s, --schema <schema>       Name of database schema.  [required]
      -m, --migrations-dir <dir>  Path to migrations directory. (default:
                                  ./migrations)  [required]
      -d, --debug                 Display stack traces when exceptions occur.
      --version                   Show the version and exit.
      --help                      Show this message and exit.

    Commands:
      bootstrap  Bootstrap the migrations table.
      drop       Drop the migrations table.
      list       List migrations.
      migrate    Run pending migrations.
      snapshot   Take a snapshot of the current schema and...
      test       Test pending migrations.

The **options** have the following meanings.

type
    **(Required)** The type of database that Agnostic is connecting to. The only
    supported type in this alpha release is ``postgres``. May be specified as
    ``AGNOSTIC_TYPE`` environment variable instead.
host
    **(Optional)** Hostname or IP address of database server. It defaults to
    ``localhost``. Agnostic uses TCP/IP connections only, never file socket
    connections. May be specified as ``AGNOSTIC_HOST`` environment variable
    instead.
port
    **(Optional)** The TCP port number that the database server is listening on.
    If omitted, this is assumed to be the default port associated with this
    <type> of database. May be specified as ``AGNOSTIC_PORT`` environment
    variable instead.
user
    **(Required)** The username to connect to the database as. This user should
    have all privileges, including the right to run any DDL statement or any DDL
    statement. May be specified as ``AGNOSTIC_USER`` environment variable
    instead.
password
    **(Required)** The password associated with the user. If omitted, you will
    be prompted to type the password on ``stdin``. May be specified as
    ``AGNOSTIC_PASSWORD`` environment variable instead. (See warning below.)
schema
    **(Required)** The name of the schema that is being managed by Agnostic. May
    be specified as ``AGNOSTIC_SCHEMA`` environment variable instead.
migrations-dir
    **(Optional)** Path to the directory that contains migration scripts. If
    not specified, it defaults to ``migrations`` in the current working
    directory. May be specified as ``AGNOSTIC_MIGRATIONS_DIR`` instead.
debug
    Display stack traces when exceptions occur.

.. warning::

    **Be careful with passwords!**

    Passing a password in the ``--password`` argument is not safe in multi-user
    environments because the password will be visible in plaintext, both in your
    shell history and in the system-wide process list.

    We recommend that you type the password. Alternatively, export the
    ``AGNOSTIC_PASSWORD`` variable in your environment, but be wary of storing
    this password on disk in a world-readable ``.profile`` or ``.bashrc``.

(Lack Of) Configuration
-----------------------

Agnostic prefers convention over configuration, and in order to avoid the need
for configuration files, Agnostic takes all of its configurations as command
line arguments. This design makes Agnostic easier to learn, but it may result in
more typing, since the configuration needs to be passed into Agnostic each time
you run it.

If you prefer to use a configuration file, here are two ways to do that.

1. Set up an alias.
2. Set up an environment file.

The first idea is obvious to shell power users: set up an alias (e.g. in your
``~/.profile`` or ``~/.bash_aliases``) for Agnostic, like this:

.. code:: bash

    alias ag=agnostic -h myhost -t postgres -u myuser -s myapp -m /opt/myapp/migrations

Now you can run shorter commands like ``ag snapshot foo.sql`` or ``ag migrate``.
This approach may be a bit limiting if you have multiple projects and each
project has different database settings.

The second approach is a bit more flexible when dealing with multiple projects.
Create a file that contains Agnostic environment variables and put it in your
project's root directory. Let's call it ``.agnostic_env``.

.. code:: bash

    export AGNOSTIC_HOST=myhost
    export AGNOSTIC_USER=myuser
    export AGNOSTIC_TYPE=postgres
    export AGNOSTIC_SCHEMA=myapp
    export AGNOSTIC_MIGRATIONS_DIR=/opt/myapp/migrations

When you are working on a project, source these environment variables into your
shell:

.. code:: bash

    /opt/myapp $ source .agnostic_env

Now you can run commands like ``agnostic snapshot foo.sql`` and ``agnostic
migrate`` and Agnostic will read the parameters from your environment variables.
When you switch to work on another project, you just need to source that
project's ``.agnostic_env``.

bootstrap
---------

.. code:: bash

    ~ $ agnostic bootstrap --help
    Usage: agnostic bootstrap [OPTIONS]

      Bootstrap the migrations table.

      Agnostic stores migration metadata inside of the database that it is
      managing. The bootstrap process creates a table to store this tracking data
      and also (optionally) loads pre-existing migration metadata into it.

    Options:
      --load-existing / --no-load-existing
                                      Track existing migrations in the new
                                      migration table.  (default: --load-existing)
      --help                          Show this message and exit.


The ``bootstrap`` command creates a table inside the managed schema to track
migrations metadata.

load-existing
    By default, the bootstrap command loads existing migrations into the
    metadata table with the special status ``bootstrapped``. This option can be
    to control that behavior. See :ref:`build_vs_migrate` for more information.

drop
----

.. code:: bash

    ~ $ agnostic drop --help
    Usage: agnostic drop [OPTIONS]

      Drop the migrations table.

      BACK UP YOUR DATA BEFORE USING THIS COMMAND!

      This destroys all metadata about what migrations have and have not been
      applied. This is typically only useful when debugging.

    Options:
      -y, --yes  Do not display warning: assume "yes".
      --help     Show this message and exit.

The ``drop`` command has the opposite effect of ``bootstrap``: it deletes the
metadata table.

yes
    By default, Agnostic requires the user to type ``y`` on ``stdin`` to confirm
    that they want to delete this table. This prompt can be skipped by passing
    the ``--yes`` flag.

list
----

.. code:: bash

    ~ $ agnostic list --help
    Usage: agnostic list [OPTIONS]

      List migrations.

      This shows migration metadata: migrations that have been applied (and the
      result of that application) and migrations that are pending.

          * bootstrapped: a migration that was inserted during the bootstrap
            process.
          * failed: the migration did not apply cleanly; the migrations system
            will not be able to operate until this is rectified, typically by
            restoring from a backup.
          * pending: the migration has not been applied yet.
          * succeeded: the migration applied cleanly.

      Applied migrations are ordered by the "started_at" timestamp. Pending
      migrations follow applied migrations and are sorted in the same order that
      they would be applied.

    Options:
      --help  Show this message and exit.

List all known migrations, both applied and pending. See :ref:metadata for more
information.

migrate
-------

.. code:: bash

    ~ $ agnostic migrate --help
    Usage: agnostic migrate [OPTIONS]

      Run pending migrations.

    Options:
      --backup / --no-backup  Automatically backup the database before running
                              migrations, and in the event of a failure,
                              automatically restore from that backup. (default:
                              --backup).
      --help                  Show this message and exit.

Run all pending migrations in the pre-determined order. See
:ref:running_migrations for more details on this process.

backup

    By default, Agnostic backs up your schema. In the event of a migrations
    failure, Agnostic will try to restore from this backup. You can disable this
    behavior, if desired.

snapshot
--------

.. code:: bash

    ~ $ agnostic snapshot --help
    Usage: agnostic snapshot [OPTIONS] OUTFILE

      Take a snapshot of the current schema and write it to OUTFILE.

      Snapshots are used for testing that migrations will produce a schema that
      exactly matches the schema produced by your build system. See the online
      documentation for more details on how to use this feature.

    Options:
      --help  Show this message and exit.

A *snapshot* is a dump of the current schema, sans data. Snapshots are useful
for testing migrations, as detailed in :ref:`workflow`.

outfile
    The name of the file to write the snapshot to.

test
----

.. code:: bash

    ~ $ agnostic test --help
    Usage: agnostic test [OPTIONS] CURRENT TARGET

      Test pending migrations.

      Given two snapshots, one of your "current" state and one of your "target"
      state, this command verifies: current + migrations = target.

      If you have a schema build system, this command is useful for verifying
      that your new migrations will produce the exact same schema as the build
      system.

      Note: you may find it useful to set up a database/schema for testing
      separate from the one that you use for development; this allows you to test
      repeatedly without disrupting your development work.

    Options:
      -y, --yes  Do not display warning: assume "yes".
      --help     Show this message and exit.

The ``test`` command verifies that a set of migrations will run without error
and will also precisely produce the desired target schema. See
:ref:test_migrations for more details.
