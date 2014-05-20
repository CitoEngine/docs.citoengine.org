Overview
========


**The problem:**

Configuring monitoring systems to alert properly is an art.
It's a fine art of configuring thresholds when your monitoring parameters vary widely or when the monitoring tools lack capability to monitor dynamic workloads.
It also takes discipline in working with monitoring systems during release process or outages.
Not all monitoring systems are configured or maintained properly. In the end you have alerts and lots of it!

**What is CitoEngine ?**

CitoEngine allows you to manage large volume of alerts and trigger actions.
These actions could notify or act on the alert by executing a script (a plugin).
It is ideal alert management service for teams who have multiple monitoring systems.

**What can it do?**

 * Accept alerts from *any* monitoring systems such as ``Nagios``, ``Sensu``, ``Cron-jobs``, etc. and aggregate alerts.
 * Lookup such alerts (called **Incidents**) to user-defined **Event** ID's and enable any action based on rules that meet a user-defined criteria
 * **Plugins** enable actions on **Incidents**. **Plugins** can be any script that run commands or make API calls.
 * **Dashboards** to give you an overview of all incoming alerts or grouped by **Teams**
 * It does *not* require any agents.
 * It plugins can be *any* executable script, no pesky DSL's.

**What it is not:**

CitoEngine is not a monitoring system.

**How do I use it?**

Now that you know what CitoEngine is, we will walk you through how you can use it.

CitoEngine is built on open source technologies and designed to run on Linux. It's built on the following components

* Python 2.7+ 
* Django 1.6.x / Twisted / other dependencies
* MariaDB / MySQL 5.5.x (PostgreSQL support coming soon)
* AWS SQS (RabbitMQ support coming soon)

Currently, we use AWS and it's SQS service for messaging support.  As soon as RabbitMQ support is integrated, AWS SQS service dependency is removed.

CitoEngine can be run on Ubuntu on baremetal hardware or on a Virtual Machine or in the cloud with the only the SQS service running on AWS. 

The next set of links on the side, will help you install and configure CitoEngine for you to manage your alerts!
