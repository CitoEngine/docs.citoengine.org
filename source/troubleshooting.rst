Troubleshooting
===============

Known issues
------------

.. _operational error: https://code.djangoproject.com/ticket/21597#comment:29

OperationalError 'MySQL server has gone away' in django1.6 when wait_timeout passed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default MySQL's ``wait_timeout`` is set at ``28800`` seconds (8 hrs). Although this is usually enough for most websites,
this may result in an `operational error`_ in poller for low traffic sites. In such cases, it would be better if you increase
the database ``wait_timeout`` to a higher number.
