Installing CitoEngine
=====================

The following guide shows installation steps on ``Ubuntu 12.04 x86_64``. Theoretically application dependencies can be fulfilled on any
linux distribution i.e. Redhat, ArchLinux, etc. In future, we will try to include installation steps for other distributions as well.
Python module dependencies are installed using `pip`_ rather than system installer, this gives us more control towards using modules of specific versions.
The following steps assume that you will be installing in ``/opt/citoengine`` directory.

.. _pip: http://www.pip-installer.org/

Installation
------------

**Install dependencies**

.. code-block:: bash


    # Installing MySQL and Python development packages
    sudo apt-get install libmysqlclient-dev python-dev python-pip git
    sudo pip install virtualenv

.. note:: If you are going to use ldap authentication, then install the following as well  ``sudo apt-get install libldap2-dev libsasl2-dev libssl-dev``


**Downloading and installing the code**

We recommend you use ``virtualenv`` for running citoengine, this will help you manage dependencies better. Download the latest build

.. code-block:: bash

    cd /tmp
    git clone https://github.com/CitoEngine/cito_engine
    cd /tmp/cito_engine
    python setup.py install
    cd /opt/
    virtualenv /opt/citoengine
    source /opt/citoengine/bin/activate
    pip install -r /tmp/cito_engine/requirements.txt


**MySQL Installation and Configuration**


.. code-block:: bash

    # Install mysql server
    sudo apt-get install mysql-server mysql-client

    # Setup mysql root password
    sudo dpkg-reconfigure mysql-server-5.5

    # Create a new database 'citoengine'
    sudo mysqladmin -uroot -p create citoengine
    # Create a new mysql user
    /usr/bin/mysql -uroot -p -e "GRANT ALL PRIVILEGES ON citoengine.* TO 'citoengine_user'@'localhost' IDENTIFIED BY 'MINISTRYOFSILLYWALKS' with GRANT OPTION"


**Setting up RabbitMQ (Optional):**

If you are planning to use RabbitMQ, the following three lines should get you started.

.. code-block:: bash

    sudo rabbitmqctl add_user citoengine_user citoengine_pass
    sudo rabbitmqctl add_vhost /citoengine_event_listener
    sudo rabbitmqctl set_permissions -p /citoengine_event_listener citoengine_user ".*" ".*" ".*"



**Edit default settings:**  Copy the sample ``/opt/citoengine/conf/citoengine.conf-example`` to ``/opt/citoengine/conf/citoengine.conf``
and edit it accordingly.


**Message Queue Configuration:**

Edit the ``DATABASE`` configuration settings and change the settings. If you are running CitoEngine on AWS,
use AWS:SQS or if running onpremise, setup RabbitMQ as your message queue. Edit either of these configuration blocks and make sure you select ``QUEUE_TYPE`` to be either ``SQS`` or ``RABBITMQ``.


.. note:: Amazon SQS does not support message sequencing i.e. it does not guarantee first in, first out for message delivery. See http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/Welcome.html


**Initializing the tables and creating an admin account.**

.. code-block:: bash

    source /opt/citoengine/bin/activate
    cd /opt/citoengine/app

    # Populate the database
    python manage.py migrate

    # Update django secret (for csrf)
    # If you are using the webapp on multiple nodes behind a load balancer,
    # make sure th secret_key.py file is same on all nodes.
    sudo sh -c '/opt/citoengine/bin/create-django-secret.py > /opt/citoengine/app/settings/secret_key.py'

    # Create your first CitoEngine superuser!
    python manage.py createsuperuser

**That's it, you are done!**

.. note:: You can always validate your installation using the command ``python manage.py validate``


Starting the services
---------------------

CitoEngine is divided into two components, ``webapp`` and ``poller``. You can run these two components using the helper
scripts ``/opt/citoengine/bin/citoengine-poller.sh`` and ``/opt/citoengine/bin/citoengine-webapp.sh``. If you are on Ubuntu,
you can configure to run them as upstart services using ``/opt/citoengine/bin/upstart/configure-upstart.sh``.


**Start CitoEngine SQS Poller service**

.. code-block:: bash

    /opt/citoengine/bin/citoengine-poller.sh


**Start CitoEngine Engine**


.. code-block:: bash

    /opt/citoengine/bin/citoengine-webapp.sh


Open your browser and access http://<hostname or IP>:8000 to login to CitoEngine with the admin account you created earlier.
