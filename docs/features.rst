========
Features
========

The following features are available in the environment constructed by the included provisioning scripts.

Several features may only apply in a production or development environment. This is differentiated based on the :ref:`conf-var-debug` setting in the ``env.sh`` file.


.. _feat-dir-structure:

Well-defined project structure
==============================

The provisioning process creates a well-defined directory structure for all project-related files.

The root of this structure is ``/opt/app/``.

The most important subdirectory is ``/opt/app/src/``. This is the project root directory, and the target of the Vagrant synced folder. Subsequently, ``/opt/app/src/provision/`` contains all the provisioning scripts.

Some of the other useful directories in this structure are:

* ``/opt/app/logs/``: For storage of log files output by supervisor, etc.
* ``/opt/app/media/``: Target for Django's ``MEDIA_ROOT``.
* ``/opt/app/static/``: Target for Django's ``STATIC_ROOT`` (in production environments).

The final directory is ``/opt/app/ln/``. This directory is primarily used to simplify the process of configuring the server. It acts as a container for shortcut symlinks to various project-specific files and directories (i.e. those that contain the project name). It is designed to allow using known paths in config files, without forcing customisation in projects that would not otherwise need it. Using the shortcuts in the ``ln`` directory, default config files that work out-of-the-box can be provided (such as ``provision/conf/supervisor/programs/gunicorn.conf``). Otherwise, such files would require the modification of a series of paths to include the project name.

.. _feat-users:

Locked down user access
=======================

SSH access is locked down to the custom ``webmaster`` user created during provisioning. SSH is available via public key authentication only - no passwords. In a development environment, only the ``webmaster`` and ``vagrant`` users are allowed SSH access. In a production environment, only ``webmaster`` is granted access. No other users, including ``root``, can SSH into the machine.

The public key to use for the ``webmaster`` user must be provided via the :ref:`conf-var-public-key` variable in the ``env.sh`` file. This will be installed into ``/home/webmaster/.ssh/authorized_keys``.

The ``webmaster`` user is given sudo privileges. In development environments, for convenience, it does not require a password. In production environments, it does. A password is not configured as part of the provisioning process, one needs to be manually created afterwards. When logged in as the ``webmaster`` user, simply run the ``passwd`` command to set a password.

Most provisioned services, such as nginx and gunicorn, are designed to run under the default ``www-data`` user.

.. warning::

    Using the provisioning scripts in a production environment with :ref:`conf-var-debug` set to ``1`` will leave the ``webmaster`` user with open ``sudo`` access, unprotected by a password prompt. This is a Bad Idea.


.. _feat-deployments:

Multiple deployments
====================

In addition to differentiating between development and production environments by way of the :ref:`conf-var-debug` setting, different deployments can also be named using the :ref:`conf-var-deployment` setting. This deployment name is used when retrieving config files.

Many of the features described below can be configured by way of files located (by default) in ``provision/conf/``. Using a named deployment allows using an entirely different set of configuration files between environments. This could mean using different configs between development and production, between multiple different production deployments of the project, etc.

When a value is provided for the :ref:`conf-var-deployment` setting, it is appended to the directory in which the provisioning scripts look for configuration files. E.g. a ``DEPLOYMENT`` of ``'dev'`` would check the ``provision/conf-dev/`` directory.

To avoid the need to create copies of *every* config file per deployment, the files in deployment-specific config directories (e.g. ``provision/conf-dev/``) are not used in isolation, but rather they *override* those in the primary config directory (``provision/conf/``). If a file exists in a deployment-specific directory, it will take precedence over a matching file in the primary directory. If a file does *not* exist in a deployment-specific directory, the file from the primary directory will be used.

.. _feat-deployments-dev:

The ``dev`` deployment
----------------------

A default deployment named "dev" is available out of the box. The included ``env.sh`` file uses a :ref:`conf-var-deployment` setting of ``'dev'`` and a ``conf-dev/`` directory of config files is provided.

This deployment overrides some of the included default config files to make them compatible with development environments. Specifically, the following alterations are made during development:

* :ref:`Nginx <feat-nginx>` is configured to run, but in a more limited capacity than in production. The differences are explained further in the nginx :ref:`feature <feat-nginx>` and :ref:`configuration <conf-nginx>` documentation.
* The :ref:`supervisor <feat-supervisor>` command for :ref:`gunicorn <feat-gunicorn>` is overridden to clear the command. Gunicorn is not provisioned in development environments, so the supervisor command would only fail anyway.


.. _feat-time-zone:

Time zone
=========

The time zone can be set using the :ref:`conf-var-time-zone` setting in the ``env.sh`` file.


.. _feat-firewall:

Firewall
========

In production environments, and if a :ref:`firewall rules configuration file <conf-firewall>` is provided, a firewall is provisioned using `UncomplicatedFirewall <https://wiki.ubuntu.com/UncomplicatedFirewall>`_.


.. _feat-git:

Git
===

`Git <https://git-scm.com/>`_ is installed.

.. tip::
    A ``.gitconfig`` file can be placed in ``provision/conf/user/`` to enable configuration of the git environment for the ``webmaster`` user. This file should be ignored by source control.


.. _feat-ag:

Ag (silver searcher)
====================

The `"silver searcher" <https://github.com/ggreer/the_silver_searcher>`_ commandline utility, ``ag``, is installed in the guest machine. ``ag`` provides fast code search that is `better than ack <http://geoff.greer.fm/2011/12/27/the-silver-searcher-better-than-ack/>`_.

.. tip::
    An ``.agignore`` file can be placed in ``provision/conf/user/`` to add some additional user-specific "ignores" for the command. This file should be ignored by source control.


.. _feat-image-libs:

Image libraries
===============

Various system-level image libraries used by `Pillow <https://python-pillow.github.io/>`_ are installed in the guest machine.

To install Pillow itself, it should be included in ``requirements.txt`` along with other Python dependencies (see :ref:`feat-py-dependencies` below). But considering many of its features `require external libraries <http://pillow.readthedocs.io/en/3.0.x/installation.html#external-libraries>`_, and the high likelihood that a Django project will require Pillow, those libraries are installed in readiness.

The exact packages installed are taken from the Pillow `"depends" script for Ubuntu <https://github.com/python-pillow/Pillow/blob/master/depends/ubuntu_14.04.sh>`_, though not all are used.

Installed packages:

* libtiff5-dev
* libjpeg8-dev
* zlib1g-dev
* libfreetype6-dev
* liblcms2-dev


.. _feat-postgres:

PostgreSQL
==========

`PostgreSQL <https://www.postgresql.org/>`_ is installed.

In addition, a database user is created with a username equal to the :ref:`project name <conf-var-project-name>` and a password equal to :ref:`conf-var-db-pass`. A database is also created, also with a name equal to the :ref:`project name <conf-var-project-name>`, with the aforementioned user as the owner.

The Postgres installation is configured to listen on the default port (5432).


.. _feat-nginx:

Nginx
=====

`nginx <https://nginx.org/en/>`_ is installed.

The ``nginx.conf`` and site config files can be modified. Support for replacing placeholder variables within these config files is also available. See :ref:`conf-nginx` for details.

Nginx is controlled and monitored by :ref:`feat-supervisor`. A default supervisor program is provided, but can be modified. See :ref:`conf-supervisor-programs` for details.

Nginx is provisioned even in development environments, for situations where it is useful to have a production-level web server available. Its usage is optional, but it is available if necessary. The default site configuration of nginx differs between production and development, as detailed below. Further details on configuring nginx can be found in the :ref:`configuration documentation <conf-nginx>`.

In production, nginx is configured to serve static and media files, and to proxy all remaining requests through to :ref:`gunicorn <feat-gunicorn>`. It is also expected to be used with the built-in support for :ref:`provisioning Let's Encrypt <feat-letsencrypt>`. As such, an HTTPS-ready configuration is provided by default for production environments.

In development, nginx is configured to serve media files only and proxy all remaining requests through to a Django runserver on port 8460. Static files are not configured to be served by nginx in development, because Django handles automatically finding and serving them in order to avoid the need to run the ``collectstatic`` command after every modification.

.. _feat-letsencrypt:

Let's Encrypt
-------------

By default, the nginx site config for production assumes HTTPS-only communication, redirecting non-HTTPS traffic *to* HTTPS. This requires that a TLS certificate be installed on the server. The `Let's Encrypt <https://letsencrypt.org/>`_ service is used to do this. Let's Encrypt provides an automated service for obtaining and maintaining domain-validation TLS certificates for free.

Configuring Let's Encrypt requires a secondary process run *after* the main provisioning process. See the :ref:`configuration documentation <conf-letsencrypt>` for more information.


.. _feat-gunicorn:

Gunicorn
========

In production environments, `gunicorn <http://gunicorn.org/>`_ is installed.

The ``conf.py`` file used can be modified. See :ref:`conf-gunicorn` for details.

Gunicorn is controlled and monitored by :ref:`feat-supervisor`. A default supervisor program is provided, but can be modified. See :ref:`conf-supervisor-programs` for details.


.. _feat-supervisor:

Supervisor
==========

`Supervisor <http://supervisord.org/>`_ is installed.

The ``supervisord.conf`` file used can be modified. See :ref:`conf-supervisor` for details.

Default programs for :ref:`feat-nginx` and :ref:`feat-gunicorn` are provided, but any number of additional programs can be added. See :ref:`conf-supervisor-programs` for details.


.. _feat-virtualenv:

Virtualenv
==========

A virtualenv is created using `pyenv <https://github.com/pyenv/pyenv>`_ and its `pyenv-virtualenv <https://github.com/pyenv/pyenv-virtualenv>`_ plugin.

The version of Python used to build the virtualenv can be specified in :ref:`conf-settings-sh` using the :ref:`conf-var-base-python` setting. If not specified, the system version will be used.

The virtualenv is automatically activated when the ``webmaster`` user logs in via SSH.

.. _feat-py-dependencies:

Python dependency installation
------------------------------

If a ``requirements.txt`` file is found in the project root directory (``/opt/app/src/``), the included requirements will be installed into the virtualenv (via ``pip -r requirements.txt``).

In development environments, a ``dev_requirements.txt`` file can also be specified to install additional development-specific dependencies, e.g. debugging tools, documentation building packages, etc. This keeps these kinds of packages out of the project's primary ``requirements.txt``.


.. _feat-node:

Node.js/npm and nps
===================

If a ``package.json`` file is found in the project root directory (``/opt/app/src/``), `node.js <https://nodejs.org/en/>`_ and `npm <https://www.npmjs.com/>`_ are installed. The version of node.js installed is dictated by the :ref:`conf-var-node-version` setting in :ref:`conf-settings-sh`.

A ``node_modules`` directory is created at ``/opt/app/node_modules/`` and a symlink to this directory is created in the project root directory (``/opt/app/src/node_modules``). Keeping the ``node_modules`` directory out of the synced folder helps avoid potential issues with Windows host machines - path names generated by installing certain npm packages can exceed the maximum Windows allows.

.. note::
    In order to create the ``node_modules`` symlink when running a Windows host and using VirtualBox shared folders, ``vagrant up`` must be run with Administrator privileges to allow the creation of symlinks in the synced folder. See :ref:`limitations-windows` for details.

.. note::
    If a ``package.json`` file is added to the project at a later date, provisioning can be safely re-run to install node/npm (using the ``vagrant provision`` command).

.. _feat-node-dependencies:

Node.js dependency installation
-------------------------------

If a ``package-lock.json`` file is found in the project root directory (``/opt/app/src/``), ``npm ci`` will be run to install npm dependencies.

In production environments, ``npm ci --production`` will be used, limiting the installed dependencies to those listed in the ``dependencies`` section of ``package.json``. Otherwise, dependencies listed in ``dependencies`` and ``devDependencies`` will be installed. See the `documentation on npm install <https://docs.npmjs.com/cli/install>`_.

.. note::
    ``npm ci`` is used in place of ``npm install`` so that the provisioning process cannot cause any modification to ``package-lock.json``, which is possible depending on the configuration of ``package.json``. This requires a ``package-lock.json`` file, in addition to ``package.json``, be present and correct in order to install the appropriate dependencies.

nps
---

If node and npm were installed, and a ``package-scripts.js`` file is also found in the project root directory (``/opt/app/src/``), `nps <https://www.npmjs.com/package/nps>`_ is installed globally.

.. note::
    If a ``package-scripts.js`` file is added to the project at a later date, provisioning can be safely re-run to install nps (using the ``vagrant provision`` command).


.. _feat-python:

Multiple Python versions and tox support
========================================

The base Python version (used to create the virtualenv under which all relevant Python processes for the project will be run) and additional versions of Python can be specified in :ref:`conf-settings-sh`, via the :ref:`conf-var-base-python` and :ref:`conf-var-python-versions`, respectively.

All specified Python versions are installed with `pyenv <https://github.com/pyenv/pyenv>`_. The pyenv `global command <https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#pyenv-global>`_ is used to provide system-wide access to all installed versions, with the following priority:

 * :ref:`conf-var-python-versions`, in the order they are defined
 * The specified :ref:`conf-var-base-python`, if there is one and if it doesn't already appear in ``PYTHON_VERSIONS``
 * The system Python

For example:

.. code-block:: bash

    # The following settings...
    BASE_PYTHON_VERSION='3.6.4'
    PYTHON_VERSIONS=('2.7.14' '3.5.4')

    # ... yield the command:
    pyenv global 2.7.14 3.5.4 3.6.4 system

If you want the specified base version to appear somewhere specific among the list of versions, include it explicitly in ``PYTHON_VERSIONS``:

.. code-block:: bash

    # The following settings...
    BASE_PYTHON_VERSION='3.6.4'
    PYTHON_VERSIONS=('3.6.4' '2.7.14' '3.5.4')

    # ... yield the command:
    pyenv global 3.6.4 2.7.14 3.5.4 system

This support is most useful when using `tox <https://tox.readthedocs.io/en/latest/>`_ to test your code under multiple versions of Python.

.. _feat-env-py:

env.py
======

Several of the :ref:`conf-env-sh` settings are designed to eliminate hardcoding environment-specific and/or sensitive settings in Django's ``settings.py`` file. Things like the database password, the ``SECRET_KEY`` and the ``DEBUG`` flag should be configured per environment and not be committed to source control.

`12factor <http://12factor.net/>`_ recommends these types of settings `be loaded into environment variables <http://12factor.net/config>`_, with these variables subsequently used in ``settings.py``. But environment variables can be a kind of invisible magic, and it is not easy to simply view the entire set of environment variables that exist for a given project's use. To make this possible, an ``env.py`` file is written by the provisioning scripts.

This ordinary Python file simply defines a dictionary called ``environ``, containing settings defined as key/value pairs. It can then be imported by ``settings.py`` and used in a manner very similar to using environment variables.

.. code-block:: python

    # Using env.py
    from . import env
    env.environ.get('DEBUG')

    # Using environment variables
    import os
    os.environ.get('DEBUG')

The ``environ`` dictionary is used rather than simply providing a set of module-level constants primarily to allow simple definition of default values:

.. code-block:: python

    env.environ.get('DEBUG', False)

The default ``environ`` dictionary will contain the following key/values:

* ``DEBUG``: Will be True if :ref:`conf-var-debug` is set to ``1``, False if it is set to ``0``.
* ``DB_USER``: Set to the value of the :ref:`project name <conf-var-project-name>`.
* ``DB_PASSWORD``: Set to the value of :ref:`conf-var-db-pass`. Automatically generated by default.
* ``TIME_ZONE``: Set to the value of :ref:`conf-var-time-zone`.
* ``SECRET_KEY``: Set to the value of :ref:`conf-var-secret-key`. Automatically generated by default.

If a specific project has additional sensitive or environment-specific settings that are better not committed to source control, it is possible to modify the way ``env.py`` is written such that it can contain those settings as well, or at least placeholders for them. See :ref:`conf-env-py` for more details.

.. note::

    The ``env.py`` file should not be committed to source control. Doing so would defeat the purpose!


.. _feat-project-provisioning:

Project-specific provisioning
=============================

In addition to the above generic provisioning, any special steps required by individual projects can be included using the ``provision/project.sh`` file. If found, this shell script file will be executed during the provisioning process. This file can be used to install additional system libraries, create/edit configuration files, etc.

For more information, see the :doc:`project-provisioning` documentation.


.. _feat-commands:

Shortcut commands
=================

The following shell commands are made available on the system path for convenience:

* ``pull+``: For git users. A helper script for pulling in the latest changes from origin/master and performing several post-pull updates. It must be run from the project root directory (``/opt/app/src/``). Specifically, and in order of operation, the script:

    * Runs ``git pull origin master`` as the ``www-data`` user
    * Runs ``python manage.py collectstatic`` (production environments only), also as the ``www-data`` user
    * Checks for differences in requirements.txt\ :sup:`#`
    * Asks to install from requirements, if any differences were found
    * Runs ``pip install -r requirements.txt`` if installing was requested
    * Checks for unapplied migrations (using Django's ``showmigrations`` management command)
    * Asks to apply the migrations, if any were found
    * Runs ``python manage.py migrate`` if applying was requested
    * Runs ``python manage.py remove_stale_contenttypes`` if using Django 1.11+
    * Restarts gunicorn (production environments only)

#: When first run, ``pull+`` detects differences between the ``requirements.txt`` file as it existed *before* the pull vs *after* the pull. Even if no differences are found, the installed packages may still be out of date if an updated ``requirements.txt`` was pulled in prior to running the command. After the first run, it stores a temporary copy of ``requirements.txt`` any time updates are chosen to be installed. It can then compare the newly-pulled file to this temporary copy, enabling it to detect changes from any pulls that took place in the meantime as well. However, if the requirements are updated manually (outside of using this command), it will detect differences in the files even if the installed packages are up to date.
