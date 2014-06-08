Integrating CitoEngine with 3rd party tools
===========================================

CitoEngine can be easily integrated with existing monitoring systems. All integration scripts mentioned here can be
found at `integerations tools`_ repository.

.. _integerations tools: https://github.com/CitoEngine/integration_tools/

Nagios
------

QuickStart
^^^^^^^^^^


1. Define an event in ``CitoEngine UI -> Events -> Define an event``

2. Add a custom Nagios variable called ``_CITOEVENTID`` in the service definition as shown below:

.. code-block:: guess
    :emphasize-lines: 3

    define service {
                service_description                   Total Processes
                _CITOEVENTID                          666
                check_command                         check_local_procs!250!400!RSZDT
                contact_groups                        +admins
                use                                   local-service
    }

3. Add the ``citoengine`` user to your notification group, in this example we will add the contact to the group ``admins``:

.. code-block:: guess
    :emphasize-lines: 4

    define contactgroup {
                contactgroup_name                     admins
                alias                                 Nagios Administrators
                members                               bofh,citoengine
    }

.. note:: Same logic applies for host definitions.

4. Copy ``citoengine.cfg`` to your nagios' directory and include it in ``nagios.cfg``.

5. Edit the ``citoengine.cfg`` file and replace the *server* and *port* to their actual values.

6. Copy ``event_publisher.py`` script to ``/usr/local/bin/`` and make it executable.


Bulk update of service definitions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you have a lot of service definitions then the above steps may prove very tedious. To help you around this we an use a helper script called ``cito_config_parser.py``.
This script runs in two modes, one where it parses an existing service definition file and other where it updates the service definition file with the relevant event_id's
exported from CitoEngine. Here is how you can do it:

1. Parse the existing service definition file:

.. code-block:: guess

    cito_config_parser.py --type nagios -c services.cfg --parse --out my-services.txt

2. Copy the output of ``my-services.txt`` into ``CitoEngine -> Tools -> Add events in bulk``

3. Select your *Team*, *Severity*, *Category*, etc and hit submit.

4. The next page shows you a list of forms for each service definition you pasted above. Go through it carefully, modify it and hit submit.

5. Go to ``CitoEngine UI -> Events``, select your *Team* check the *Export CSV* checkbox and hit search. The UI will give a CSV file of all your team's events.
Save this locally and have a quick look at it to confirm everything is in order.

6. Generate the new services config using the following command:

.. code-block:: guess

    cito_config_parser.py --type nagios -c services.cfg --events-file events.csv --generate --out new_services.cfg

.. note:: Do not run the ``--generate`` command on a previously configured services.cfg which already has _CITOEVENTID added. Always use the original service definition file.
.. note:: Sensu support will be released shortly.