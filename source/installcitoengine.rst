Installing CitoEngine
=====================

The following guide shows installation steps on ``Ubuntu 12.04 x86_64``. Theoretically application dependencies can be fulfilled on any
linux distribution viz. Redhat, ArchLinux, etc. In future, we will try to include installation steps for other distributions as well.
Python module dependencies are installed using `pip`_ rather than system installer, this gives us more control towards using modules of specific versions.
The following steps assume that you will be installing in ``/opt/cito`` directory.

.. _pip: http://www.pip-installer.org/

Installation
------------

**Install dependencies**

.. code-block:: bash


    # Installing MySQL and Python development packages
    sudo apt-get install libmysqlclient-dev python-dev python-pip
    sudo pip install virtualenv
    # If you are going to use ldap authentication, then install the following as well
    sudo apt-get intall libldap2-dev libsasl2-dev libssl-dev

**MySQL Installation and Configuration**


.. code-block:: bash

    # Install mysql server
    sudo apt-get install mysql-server mysql-client

    # Setup mysql root password
    sudo dpkg-reconfigure mysql-server-5.5
    # Create a new database 'cito'
    sudo mysqladmin -uroot -p create cito
    # Create a new mysql user
    /usr/bin/mysql -uroot -p -e "GRANT ALL PRIVILEGES ON cito.* TO 'cito_user'@'localhost' IDENTIFIED BY 'MINISTRYOFSILLYWALKS' with GRANT OPTION"

**Setup python virtualenv**

We recommend you use ``virtualenv`` for running citoengine, this will help you manage dependencies better. Download the latest build

.. code-block:: bash

    cd /opt/
    git clone https://github.com/CitoEngine/cito_engine /opt/cito


.. code-block:: bash

    sudo mkdir -p /opt/virtualenvs && sudo chown $USER /opt/virtualenvs/ && cd /opt/virtualenvs
    virtualenv --no-site-packages /opt/virtualenvs/citovenv
    source /opt/virtualenvs/citovenv/bin/activate
    pip install -q --upgrade setuptools
    pip install -r /opt/cito/requirements.txt


**Edit default settings:**  ``/opt/cito/cito/settings/production.py``

**Message Queue Configuration:**

CitoEngine can be run on Amazon Web Services (AWS) cloud or onpremise.  

If you are running CitoEngine on AWS, use AWS:SQS or if running onpremise, setup RabbitMQ as your message queue. Edit either of these configuration blocks and make sure
you select ``QUEUE_TYPE`` to be either ``SQS`` or ``RABBITMQ``

.. code-block:: python

    ##################################
    # AWS::SQS Configuration settings
    ##################################
    AWS_CONF = dict()
    AWS_CONF['region'] = 'us-east-1'
    AWS_CONF['awskey'] = ''
    AWS_CONF['awssecret'] = ''
    AWS_CONF['sqsqueue'] = 'citoq'


    ##################################
    # RabbitMQ Configuration settings
    ##################################
    RABBITMQ_CONF = dict()
    RABBITMQ_CONF['host'] = 'localhost'
    RABBITMQ_CONF['port'] = 5672
    RABBITMQ_CONF['username'] = 'cito_user'
    RABBITMQ_CONF['password'] = 'CHANGEME!'
    RABBITMQ_CONF['ssl'] = False
    RABBITMQ_CONF['exchange'] = ''
    RABBITMQ_CONF['vhost'] = '/cito_event_listener'
    RABBITMQ_CONF['queue'] = 'cito_commonq'

    ##############################
    # Queue type: SQS or RABBITMQ
    ##############################
    QUEUE_TYPE = 'RABBITMQ'

.. note:: Avoid editing ``/opt/cito/cito/settings/base.py`` unless you know what you are doing.

**Setting up RabbitMQ (Optional):**

If you are planning to use RabbitMQ, the following three lines should get you started.

.. code-block:: bash

    sudo rabbitmqctl add_user cito_user cito_pass
    sudo rabbitmqctl add_vhost /cito_event_listener
    sudo rabbitmqctl set_permissions -p /cito_event_listener cito_user ".*" ".*" ".*"

**Database Configuration:**

.. code-block:: python

    #Database config
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.mysql',   # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
            'NAME': 'cito',                         # Or path to database file if using sqlite3.
            'USER': '',                             # Not used with sqlite3.
            'PASSWORD': '',                         # Not used with sqlite3.
            'HOST': '',                             # Set to empty string for localhost. Not used with sqlite3.
            'PORT': '',                             # Set to empty string for default. Not used with sqlite3.
            'OPTIONS': {
                'init_command': 'SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED'
            }
        }
    }

**Initializing the tables and creating an admin account.**

.. code-block:: bash

    cd /opt/cito

    # Populate the database
    python manage.py syncdb --noinput --migrate

    # Update django secret (for csrf)
    # If you are using the webapp on multiple nodes behind a load balancer,
    # make sure th secret_key.py file is same on all nodes.
    sudo sh -c '/opt/cito/bin/create-django-secret.py > /opt/cito/cito/settings/secret_key.py'

    # Create your first CitoEngine superuser!
    python manage.py createsuperuser

**That's it, you are done!**

.. note:: You can always validate your installation using the command ``python manage.py validate``


Starting the services
---------------------

CitoEngine is divided into three components, ``poller``, ``listener`` and ``webapp``. You will have to start services of all three components.
You can either run the helper scripts in the ``/opt/cito/bin`` directory, or you can run the using ``manage.py <command>``


**Start CitoEngine SQS Poller service**

.. code-block:: bash

    /opt/cito/bin/cito-poller.sh

**Start CitoEngine Event Listener service**

.. code-block:: bash

    /opt/cito/bin/cito-listener.sh


**Start CitoEngine Webapp**

We would recommended that you execute above commands with lower privileges. Have a look at ``bin/cito-webapp.sh``
for more information.

.. code-block:: bash

    /opt/cito/bin/cito-webapp.sh


Open your browser and access http://<hostname or IP>:8000 to login to CitoEngine with the admin account you created earlier.
