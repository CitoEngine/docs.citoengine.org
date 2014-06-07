Installing Cito Plugin Server
=============================

The following guide shows installation steps on ``Ubuntu 12.04 x86_64``. Theoretically application dependencies can be fulfilled on any
linux distribution viz. Redhat, ArchLinux, etc. In future, we will try to include installation steps for other distributions as well.
Python module dependencies are installed using `pip`_ rather than system installer, this gives us more control towards using modules of specific versions.
The following steps assume that you will be installing in ``/opt/cito_plugin_server`` directory.

.. _pip: http://www.pip-installer.org/

Installation
------------

**Install dependencies**

.. code-block:: bash


    # Installing MySQL and Python development packages
    sudo apt-get install libmysqlclient-dev python-dev python-pip
    sudo pip install virtualenv

**MySQL Installation and Configuration**


.. code-block:: bash

    # Install mysql server
    sudo apt-get install mysql-server mysql-client

    # Setup mysql root password
    sudo dpkg-reconfigure mysql-server-5.5
    # Create a new database 'cito_plugin_server'
    sudo mysqladmin -uroot -p create cito_plugin_server
    # Create a new mysql user
    /usr/bin/mysql -uroot -p -e "GRANT ALL PRIVILEGES ON cito_plugin_server.* TO 'cito_user'@'localhost' IDENTIFIED BY 'MINISTRYOFSILLYWALKS' with GRANT OPTION"

**Setup python virtualenv**

We recommend you use ``virtualenv`` for running cito engine, this will help you keep manage the dependencies better. Download the latest build

.. code-block:: bash

    cd /opt/
    git clone https://github.com/CitoEngine/cito_plugin_server /opt/cito_plugin_server


.. code-block:: bash

    sudo mkdir -p /opt/virtualenvs && sudo chown $USER /opt/virtualenvs/ && cd /opt/virtualenvs
    virtualenv --no-site-packages /opt/virtualenvs/citopluginvenv
    source /opt/virtualenvs/citopluginvenv/bin/activate
    pip install -q --upgrade setuptools
    pip install -r /opt/cito_plugin_server/requirements.txt


**Edit default settings:**  ``/opt/cito_plugin_server/cito_plugin_server/settings/production.py``

.. code-block:: python

    #Database config
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',   # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
            'NAME': 'cito_plugin_server',                         # Or path to database file if using sqlite3.
            'USER': '',                             # Not used with sqlite3.
            'PASSWORD': '',                         # Not used with sqlite3.
            'HOST': '',                             # Set to empty string for localhost. Not used with sqlite3.
            'PORT': '',                             # Set to empty string for default. Not used with sqlite3.
            'OPTIONS': {
                'init_command': 'SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED'
            }
        }
    }

    PLUGIN_RUNNER_CONFIG = {
        'timeout': 5
    }

    PLUGIN_DIR = '/opt/plugins/'

.. note:: Avoid editting ``/opt/cito_plugin_server/cito_plugin_server/settings/base.py`` unless you know what you are doing.

Initializing the tables and creating an admin account.

.. code-block:: bash

    cd /opt/cito_plugin_server

    # Populate the database
    python manage.py syncdb --noinput --migrate

    # Update django secret (for csrf)
    # If you are using the webapp on multiple nodes behind a load balancer,
    # make sure th secret_key.py file is same on all nodes.
    sudo sh -c '/opt/cito_plugin_server/bin/create-django-secret.py > /opt/cito_plugin_server/cito_plugin_server/settings/secret_key.py'

    # Create your first superuser!
    python manage.py createsuperuser

**That's it, you are done!**

.. note:: You can always validate your installation using the command ``python manage.py validate``


Starting the services
---------------------

You can either run the helper scripts in the ``/opt/cito_plugin_server/bin`` directory, or you can run the using ``manage.py <command>``

**Start CitoEngine Plugin Server**

We would recommended that you execute it with lower privileges. Have a look at ``bin/cito-webapp.sh``
for more information.

.. code-block:: bash

    /opt/cito_plugin_server/bin/cito-plugin-server.sh


Open your browser and access http://<host>:8000 to login to CitoEngine Plugin Server with the admin account you created earlier.
