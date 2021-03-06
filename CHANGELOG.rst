==========
Change Log
==========

0.7
===

This is a major, backwards-incompatible release.

With the support for separate configuration files for multiple deployments, this release changes the structure of ``provision/conf/supervisor/``. Instead of looking for separate ``production_programs`` and ``dev_programs`` subdirectories, the provisioning scripts only look for a directory called ``programs``. If needing different supervisor programs in development vs production, create a separate "conf" directory for the different deployment/s, along with a ``DEPLOYMENT`` entry in ``env.sh``, and add the programs in there. For more information, see the documentation on multiple deployment support: https://vagrant-django.readthedocs.io/en/latest/features.html#multiple-deployments.

The default nginx and gunicorn supervisor programs are moved from ``provision/conf/production_programs`` to ``provision/conf/programs``. In addition, config overrides for a default "dev" deployment are added in ``provision/conf-dev/``. If upgrading from a previous version, in addition to replacing the ``provision/scripts`` directory, you will also need to add the new ``conf-dev`` directory and mirror the renaming of the ``production_programs`` directory.

The default nginx site configs are significantly updated, not only to provide a separation of development and production configs, but also to provide secured (TLS-aware) and unsecured configs. Snippets are also now found in ``provision/conf/nginx/snippets/``. If upgrading from a previous version, you may need to update these nginx config files in both ``provision/conf/nginx/`` and ``provision/conf-dev/nginx/``.

The ``provision/templates/env.py.txt`` file is moved to ``provision/conf/env.py``, and the syntax for replaceable variables is altered. In addition, the ``ENV_PY_TEMPLATE`` environment setting can no longer be used to specify a different template for the ``env.py`` file, the provisioning scripts assume the presence of ``provision/conf/env.py``. Projects customising this template will need to move the file to the new location and update the syntax used to specify replaceable variables. Projects using the setting to specify an alternate template will need to do the same to the custom file. Projects not customising either will still need to move the ``provision/templates/env.py.txt`` file to ``provision/conf/env.py``.

This release also renames the ``provision/versions.sh`` file to ``provision/settings.sh``, and moves the project name from a provisioner argument in the ``Vagrantfile`` to the ``PROJECT_NAME`` setting in ``settings.sh``. If upgrading from a previous version, ``versions.sh`` will need to be renamed, and the ``PROJECT_NAME`` setting added to it.

* Updated default Vagrant box to Ubuntu 18.04.
* Separated installation of python from installation of the project's python dependencies.
* Separated installation of node/npm from installation of the project's npm dependencies.
* Added manually-invokable script for provisioning TLS via Let's Encrypt.
* Added support for copying nginx snippets from ``provision/conf/nginx/snippets/``.
* Added support for provisioning nginx in development environments.
* Added support for keeping separate configuration files for different deployments.
* Added config files for a default "dev" deployment.
* Added provisioning of nps if a ``package-scripts.js`` file is detected.
* Added support for ensuring the latest version of pip is installed in the Python virtualenv.
* Renamed ``provision/versions.sh`` to ``provision/settings.sh``.
* Moved project name from provisioner argument in ``Vagrantfile`` to ``PROJECT_NAME`` setting in ``provision/settings.sh``.
* Moved ``env.py`` template from ``provision/templates`` to ``provision/conf`` and removed ``ENV_PY_TEMPLATE`` environment setting.
* Moved copied nginx config files from ``/opt/app/conf/nginx/`` to ``/etc/nginx/``.
* Moved copied gunicorn config file from ``/opt/app/conf/gunicorn/`` to ``/etc/gunicorn/``.
* Fixed long-standing bug setting the timezone.
* Fixed issue of provisioning process altering ``package-lock.json`` files by using "npm ci" rather than "npm install" to install npm dependencies. Installation of npm dependencies is now predicated on the existence of ``package-lock.json`` rather than ``package.json``.

0.6.2
=====

This release again updates the default supervisor program for gunicorn, in ``provision/conf/supervisor/production_programs/gunicorn.conf``. If upgrading from a previous version, in addition to replacing the ``provision/scripts`` directory, you may want to copy this file into your project.

* Fixed bugs creating symlinks under ``/opt/app/ln/`` when re-provisioning.
* Added an ``environment`` setting to the included gunicorn supervisor program file to set the PYTHONPATH, making it consistent with Django management commands.

0.6.1
=====

This release updates the default supervisor program for gunicorn, in ``provision/conf/supervisor/production_programs/gunicorn.conf``. If upgrading from a previous version, in addition to replacing the ``provision/scripts`` directory, you may want to copy this file into your project.

* Fixed bug validating ``DEBUG`` flag.
* Fixed bug in rand_str when 'python' is not found and 'python3' is: actually use the 'python3' command.
* Added ``/opt/app/ln/`` directory as a container for shortcut symlinks to project-specific directories (i.e. that contain the project name). This allows using known paths in config files without forcing customisation per project. This restores a working supervisor program for gunicorn out of the box (it was broken in 0.5 with the move to pyenv/pyenv-virtualenv).

0.6
===

This release adds a ``provision/versions.sh`` file. If upgrading from a previous version, in addition to replacing the ``provision/scripts`` directory, be sure to copy this file into your project.

* Made ``DEBUG`` flag required.
* Added ``versions.sh`` to keep project version information outside ``env.sh`` (as it is not environment-specific).
* Moved base Python version definition from ``Vagrantfile`` to ``versions.sh``.
* Moved ``PYTHON_VERSIONS`` setting from ``env.sh`` to ``versions.sh``.
* Renamed ``scripts/database.sh`` to ``scripts/postgres.sh``.
* Removed installation of custom postgres apt repo in favour of using the OS's default.
* Added ``NODE_VERSION`` setting to ``version.sh``.
* Updated installation of custom node.js repo to use ``NODE_VERSION``.
* Removed ``apt.sh``.
* Added MIT license.

0.5.1
=====

* Ensured group read/write permissions are assigned to appropriate directories, specifically the synced folder (for webmaster user access).
* Updated to skip installing the system libraries required for installing additional Python versions if none are specified.

0.5
===

* Added support for pyenv and installing multiple versions of Python.
* Switched from using virtualenv directly to using pyenv-virtualenv.
* Increased robustness of postgres configuration (now looks in more places for config files).

0.4
===

* Removed the notion of "build modes".
* Updated ``provision/`` directory structure to support additional configuration files and templates.
* Moved synced folder to ``/opt/app/src``.
* Moved other important directories under ``/opt/app``. This is now the home of everything related to the project.
* Switched to using the "webmaster" user, created during the provisioning process, as the SSH user. The custom public key installed during provisioning is now installed for this user instead of "vagrant".
* Added provisioning for supervisor.
* Added provisioning for gunicorn as a production application server for Django, managed by supervisor.
* Added provisioning for nginx as a reverse proxy to gunicorn, managed by supervisor.
* Added provisioning for firewall rules via ufw.
* Added pull+ command.
* Removed shell+ and runserver+ commands.

0.3.2
=====

* Fixed bug creating the ``node_modules`` symlink in some Windows environments.

0.3.1
=====

* Fixed bug referencing ``DEBUG`` in ``provision/scripts/node-npm.sh``.

0.3
===

* Updated ``provision/`` directory structure.
* Added support for project-specific provisioning.
* Updated copy of specific configuration files in ``provision/config/`` to copy of all configuration files in ``provision/conf/``.
* Updated Node.js/npm to install when ``DEBUG`` is set or not. Will use ``npm install --production`` when not set.
* Updated Node.js/npm to install only if a package.json file is present.
* Added provisioning for several of the image libraries Pillow requires for some of its features.
* Updated "app" build mode to always set ``DEBUG``.

0.2.3
=====

* Fixed #3: No permission to create test databases.
* Made env.py file accessible only to the owner (vagrant), at least in certain situations.

0.2.2
=====

* Fixed #2: root ownership of node_modules/.bin.

0.2.1
=====

* Fixed #1: Installing psycopg2 via ``requirements.txt`` or ``dev_requirements.txt`` before Postgres was installed caused the ``pip install -r`` to fail.

0.2
===

* Added provisioning for node.js/npm, and detection of a ``package.json``, for development environments.
* Fixed bug writing shortcut scripts.
* Added provisioning for the silver searcher (ag).
* Renamed ``env.sh`` setting ``TIMEZONE`` to ``TIME_ZONE``, and added to ``env.py``.
