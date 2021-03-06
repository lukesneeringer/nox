Tutorial
========

This tutorial will walk you through installing, configuring, and running Nox.


Installation
------------

Nox can be easily installed via `pip`_::

    pip install nox-automation

Usually you install this globally, similar to ``tox``, ``pip``, and other similar tools.


Running Nox
-----------

The simplest way of running Nox will run all sessions defined in `nox.py`::

    nox

However, if you wish to run a single session or subset of sessions you can use the ``-s`` argument::

    nox --sessions lint py27
    nox -s lint py27

You can read more about invoking Nox in :doc:`usage`.


Creating a noxfile
------------------

When you run ``nox``, it looks for a file named `nox.py` in the current directory. This file contains all of the session definitions. A *session* is an environment and a set of commands to run in that environment. Sessions are analogous to *environments* in tox. Sessions are defined via functions in `nox.py` that start with ``session_``. A simple session for running `flake8`_ would look like this::

    def session_lint(session):
        session.install('flake8')
        session.run('flake8')

If you run this via `nox` you should see output similar to this::

    nox > Running session lint
    nox > virtualenv /tmp/example/.nox/lint
    nox > pip install --upgrade flake8
    nox > flake8
    nox > Session lint successful. :)


Setting up virtualenvs and installing dependencies
--------------------------------------------------

Nox automatically creates a separate `virtualenv`_ for every session. You can choose which Python interpreter to use when creating the session. When you install dependencies or run commands within a session, they all use the session's virtualenv. Here's an example of a session that uses Python 2.7, installs dependencies in various ways, and runs a command::


    def session_py27(session):
        # Use Python 2.7.
        session.interpreter = 'python2.7'
        # Install py.test
        session.install('pytest')
        # Install everything in requirements-dev.txt
        session.install('-r', 'requirements-dev.txt')
        # Install the current package in editable mode.
        session.install('-e', '.')
        # Run py.test. This uses the py.test executable in the virtualenv.
        session.run('py.test')


You can create as many session as you want. Typically, you will create one session per interpreter version you want to test on.

You can read more about configuring sessions in :doc:`config`.

Sharing configuration between sessions
--------------------------------------

If you want a bunch of sessions to do the same thing but use different interpreters, you can just define a function that sets up all of the common stuff::

    def common(session):
        session.install('-r', 'requirements-dev.txt')
        session.run('py.test')

    def session_py27(session):
        session.interpreter = 'python2.7'
        common(session)

    def session_py34(session):
        session.interpreter = 'python3.4'
        common(session)

Remember, Nox only recognizes functions that start with `session_` as sessions.


Passing arguments into sessions
-------------------------------

Often it's useful to pass arguments into your test session. Here's a quick example that demonstrates how to use arguments to run tests against a particular file::

    def session_test(session):
        session.install('py.test')

        if session.posargs:
            test_files = session.posargs
        else:
            test_files = ['test_a.py', 'test_b.py']

        session.run('pytest', *test_files)

Now you if you run::

    nox

Then nox will run::

    pytest test_a.py test_b.py

But if you run::

    nox -- test_c.py

Then nox will run::

    pytest test_c.py


.. _parametrized:

Parametrizing sessions
----------------------

Session arguments can be parametrized with the :func:`nox.parametrize` decorator. Here's a typical example of parametrizing Python intepreter versions::

    @nox.parametrize('python_version', ['2.7', '3.4', '3.5'])
    def session_tests(session, python_version):
        session.interpreter = 'python' + python_version
        session.install('pytest')
        session.run('py.test')

When you run ``nox``, it will create a three distinct sessions::

    $ nox
    nox > Running session tests(python_version='2.7')
    nox > virtualenv ./.nox/tests -p python2.7
    ...
    nox > Running session tests(python_version='3.4')
    nox > virtualenv ./.nox/tests -p python3.4
    ...
    nox > Running session tests(python_version='3.5')
    nox > virtualenv ./.nox/tests -p python3.5


:func:`nox.parametrize` has the same interface and usage as `py.test's parametrize <https://pytest.org/latest/parametrize.html#_pytest.python.Metafunc.parametrize>`_. You can also stack the decorator to produce sessions that are a combination of the arguments, for example::


    @nox.parametrize('python_version', ['2.7', '3.4'])
    @nox.parametrize('django_version', ['1.8', '1.9'])
    def session_tests(session, python_version):
        session.interpreter = 'python' + python_version
        session.install('pytest', 'django==' + django_version)
        session.run('py.test')


If you run ``nox --list-sessions``, you'll see that this generates the following set of sessions::

    * tests(django_version='1.8', python_version='2.7')
    * tests(django_version='1.9', python_version='2.7')
    * tests(django_version='1.8', python_version='3.4')
    * tests(django_version='1.9', python_version='3.4')


If you only want to run one of the parametrized sessions, see :ref:`running_paramed_sessions`.

.. _pip: https://pip.readthedocs.org
.. _flake8: https://flake8.readthedocs.org
.. _virtualenv: https://virtualenv.readthedocs.org
