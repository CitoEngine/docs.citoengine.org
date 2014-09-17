Getting Started
===============
.. _integerations tools: https://github.com/CitoEngine/integration_tools/

It is highly recommended that you glance over the :ref:`architecture` and :ref:`terminology` docs before
proceeding further.

Assuming you have the CitoEngine and CitoPluginServer setup, lets configure an end-to-end setup where we:

a. Setup the ``Event`` codes.

b. Setup the a ``Plugin``

c. Configure ``EventActions``


Setting up Event Codes
^^^^^^^^^^^^^^^^^^^^^^

With a fresh installation, you should first define some teams and categories before creating events. Head over to
``CitoEngine->Settings->Teams`` and add a few teams there (e.g. Ops, QA, DBA-Ops, etc.). Next, head over to
``CitoEngine->Settings->Categories`` and add a few event categories (e.g. Disk, CPU, Memory, Application, etc.).

.. note:: It is possible to change the names of teams and categories anytime after their creation.

To define an event, go to ``CitoEngine->Event Codes->Define New Event Code``. Fill in the summary as needed e.g.:

.. code-block:: guess

    Summary: /var full
    Description: Server's /var partition is full, it needs to be cleaned up.
    Severity: S3
    Team: Ops
    Category: Disk
    Status: <enabled>


As this is your first event definition, its event code would be **1**.
With this bare minimum setup, you are now ready to accept Incidents (alerts) for *EventCode: 1*. Lets test our newly
created event code:

``event_publisher.py -e 1 -H "foo.bar.com" -m "It Works!" --cito-server localhost --cito-port 8080``

.. note:: You can find ``event_publisher.py`` in `integerations tools`_ repository.

Alternately, you can do a JSON ``POST`` to the listener e.g.

``curl -X POST -d '{"eventid": 1, "element": "citoengine", "message": "healthcheck message"}' "http://my.citoengine.com:8080/addevent/"``

Setting up a Plugin
^^^^^^^^^^^^^^^^^^^

Login to the plugin server and create an API Key ``CitoPluginServer->API Keys->Add new key`` e.g. *Ops-Key*

Next, we define a Plugin. This can be done at ``CitoPluginServer->Plugins->Add new Plugin``. The ``Name`` here is what
gets displayed in CitoEngine, so make sure it is unique and non ambigious. Remember, ``Plugin path`` field is relative to
the ``PLUGIN_DIR`` in your settings file i.e. if you have /opt/citoplugins/clear_tmp.sh plugin and your settings is ``PLUGIN_DIR='/opt/citoplugins'``
then you just need to give ``clear_tmp.sh`` in ``Plugin path``. To summarize, for our example, a plugin definition would look like:

.. code-block:: guess

    Name: ClearTmp
    Description: Clears /var/tomcat/temp folder.
    Plugin path: clear_tmp.sh
    Status: <enabled>
    Accessible by: Cyrus, Ops-Key

Now lets go back to the API section and copy the URL listed under our previously defined API key e.g.
``http://192.168.77.77:9000/api/13429401-3e5b-46d4-9762-b40ce689386e``

Add this to ``CitoEngine->Plugins->Add a server``, once added click on the **Refresh** link in the listings page. This would query the plugin
server and fetch all active plugins.


Configuring an Event Action
^^^^^^^^^^^^^^^^^^^^^^^^^^^

With the newly created Plugin (ClearTmp) ready to be used, lets go back to our previously created event and add an
action against it. Go to ``CitoEngine->Events->View Event Codes`` and click on our example event. In the details page,
click on ``Add an action to this event``, this should show you the event action creation form. Select the plugin *ClearTmp*,
make sure *enabled* checkbox is ticked.

We need to configure when to invoke the plugin. This can be done by setting the ``Threshold count`` and ``Threshold timer`` values.
``Threshold count`` of **2** and ``Threshold timer`` of **60** indicates that execute the plugin if this event is called **2 times in 60 secs**

If you are using a self signed SSL certificate, you may want to uncheck the ``SSL Verify`` box on this page. Hit save and you are done.

Use the ``curl`` or ``event_publisher.py`` to send a few sample events making sure that your plugin is executed as intended.

