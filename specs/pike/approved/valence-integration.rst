..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Valence Integration
===================

https://blueprints.launchpad.net/mogan/+spec/rsd-integration

The current Mogan implementation only supports pre-set configuration servers.
For custom servers, Mogan should to be able to compose bare metal through
integration with Valence that leverages the Redfish API to compose nodes using
disaggregated resources.


Problem description
===================

Mogan currently can only provision pre-set configuration servers, but users may
want to request a custom server with specific configurations like CPU, RAM, and
DISK.

Use Cases
---------

* An enterprise user wants to manage the RSD and general servers in a
unified manner.

* An enterprise user wants to apply a custom server with CPU, RAM, and DISK
specified himself.


Proposed change
===============

First, we need to refactor our flavor to pass Valence required parameters when
composing a node, need to align with Valence team. But for non-rack servers
we can keep the current way of scheduling a node to provision.

When a request come with the Valence specific flavor, We can invoke Valence to
compose the node on the fly, then register the composed node into Ironic with
Redfish driver(not supported yet). When nodes are enrolled in Ironic, there's
no difference with non-rack nodes. And these works are all done before the
current instance create workflow, so we can create a new taskflow [1]_ for
Valence which includes compose and enroll tasks:

ComposeNodeTask:
* execute: Invoke Valence to compose a node according the specified flavor.
* revert: Release the composed node if there's something wrong when enrolling.

EnrollNodeTask:
* execute: Enroll the composed node to Ironic.
* revert: If some exception raised and the node has been enrolled, need to
remove it from Ironic.

For Valence node, we should skip the scheduling task in provison workflow.
Currently there are ScheduleCreateInstanceTask and OnFailureRescheduleTask,
we can get rid of these two tasks when initialize the task flow in Valence
scenario. Or maybe can handle this like select which node instances are
launched(not supported yet).

Also, if there's some exception raised when provisioning, we should release the
composed node to Valence pool and remove it from Ironic.

When deleting a node we should remove it from ironic first, then release the
resources to Valence pool. For this, we can add a new field to instance to
indicate whether it's a valence instance or not.


Alternatives
------------

It will automatically invoke valence to compose node if scheduling max attempts
exceeds instead of using a specific flavor to indicate it's a Valence instance.

Data model impact
-----------------

The proposed change will be add the following fields to the instance object
with their data type and default value for migrations.

+-----------------------+--------------+-----------------+
| Field Name            | Field Type   | Migration Value |
+=======================+==============+=================+
| composed              | bool         | None            |
+-----------------------+--------------+-----------------+


REST API impact
---------------

None

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

There's one potential performance impact on the instance creating process,
as we need to composing the node from Valence first.

Other deployer impact
---------------------

None

Developer impact
----------------

* As Mogan plans to support not only Ironic driver but also CloudBoot, need
to figure out whether CloudBoot has supported Redfish already or there's not
a plan to support it.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <niu-zglinux>

Work Items
----------

* Refactor flavor(instance type) to meet Valence's requirements.
* Add `composed` filed to instance object.
* Add a new taskflow for node composing and enrolling.
* Change delete instance process to handle composed node gracefully.
* Add Valence installation in Mogan devstack plugin as an option

Dependencies
============

* Need valence client to be ready to integrate.

* Redfish driver landed in ironic.

* Valence PodManager simulator need to be improved, maybe return a fake
node(VM) and maybe we can test it with ssh driver before Redfish driver
available.


Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about Valence integration will be added.

References
==========

.. [1] http://wiki.openstack.org/wiki/TaskFlow
