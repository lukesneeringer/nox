Welcome to Nox
==============

.. toctree::
   :hidden:
   :maxdepth: 2

   tutorial
   config
   usage
   contrib

``nox`` is a command-line tool that automates testing in multiple Python environments, similar to `tox`_. Unlike tox, Nox uses a standard Python file for configuration.

Install nox via `pip`_::

    pip install nox-automation

Nox is configured via a ``nox.py`` file in your project's directory. Here's a simple noxfile that runs `py.test`_ with Python 2.7 and Python 3.4::

    def session_py27(session):
        session.interpreter = 'python2.7'
        session.install('-r', 'requirements.txt')
        session.run('py.test')

    def session_py34(session):
        session.interpreter = 'python3.4'
        session.install('-r', 'requirements.txt')
        session.run('py.test')

To run both of these sessions, execute::

    nox

For each session, Nox will automatically create `virtualenv`_ with the appropriate interpreter, install the specified dependencies, and run the commands in order.

To learn how to install and use Nox, see the :doc:`tutorial`. For documentation on configuring sessions, see :doc:`config`. For documentation on running ``nox``, see :doc:`usage`.

.. _tox: https://tox.readthedocs.org
.. _pip: https://pip.readthedocs.org
.. _py.test: http://pytest.org
.. _virtualenv: https://virtualenv.readthedocs.org
