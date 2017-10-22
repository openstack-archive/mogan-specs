..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
RSD Integration
===================

https://blueprints.launchpad.net/mogan/+spec/rsd-integration

The current Mogan implementation only supports pre-set configuration servers.
For custom servers, Mogan should to be able to compose bare metal through
integration with rsd-lib [1]_ that extends the existing Sushy [2]_ library
and leverages the Redfish API to compose nodes using disaggregated resources.


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
specified by himself.


Proposed change
===============

We also need to extend our server create API to pass rsd-lib required parameters
when composing a node, the parameters include the nodes hardware informations,
such as, processor, memory, remote drive, local drive and ethernet interface.
For non-rack servers we can keep the current way of scheduling a node to provision.

When a request come with the rsd-lib specific parameters, We can invoke the
rsd-lib directly to compose the node on the fly, then register the composed
node into Ironic with Redfish driver, but there maybe potential risk for
redfish cannot decide which one is the `correct` nic with multi-nics on a RSD
server. When nodes are enrolled in Ironic, there's no difference with non-rack
nodes. And these works are all done before the current server create workflow,
so we can create a new taskflow [3]_ for rsd-lib which includes compose and
enroll tasks:

ComposeNodeTask:
* execute: Invoke rsd-lib to compose a node according to the rsd specific
parameters.
* revert: Release the composed node if there's something wrong when enrolling.

EnrollNodeTask:
* execute: Enroll the composed node to Ironic.
* revert: If some exception raised and the node has been enrolled, need to
remove it from Ironic.

To distinguish RSD or no-rack server, we will add a system metadata key-value
'rsd=true' for it.

For rsd-lib node, we should skip the scheduling task in provison workflow.
Currently there are OnFailureRescheduleTask, we will check key/value pair
'rsd=true' in server's system_metadata to decide whether we need to trigger
rescheduling in rsd-lib scenario.

Also, if there's some exceptions raised when provisioning, we should release the
composed node to rsd-lib pool and remove it from Ironic.

When deleting a server we should remove it from ironic firstly, then release the
resources to RSD pool.


Alternatives
------------

It will automatically invoke to compose node if scheduling max attempts
exceeds instead of pass specific parameters to indicate it's a RSD server.

Data model impact
-----------------

None

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

There's one potential performance impact on the server creating process,
as we need to composing the node via rsd-lib first.

Other deployer impact
---------------------

None

Developer impact
----------------

* As Mogan plans to support not only Ironic driver but also other baremetal
drivers, need to figure out whether they has supported Redfish already or
there's no plan to support it.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <niu-zglinux>
  <shaohe_feng>
  <Xinran WANG>

Work Items
----------

* Refactor flavor(server type) to meet rsd-lib's requirements.
* Add `composed` filed to server object.
* Add a new taskflow for node composing and enrolling.
* Change delete server process to handle composed node gracefully.
* Add rsd-lib installation in Mogan devstack plugin as an option

Dependencies
============

* Redfish driver landed in ironic.

* PodManager simulator need to be improved, maybe return a fake
node(VM) and maybe we can test it with ssh driver before Redfish driver
available.


Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about rsd-lib integration will be added.

References
==========
.. [1] https://github.com/openstack/rsd-lib
.. [2] https://github.com/openstack/sushy
.. [3] https://wiki.openstack.org/wiki/TaskFlow

* https://github.com/openstack/rsd-lib/blob/master/doc/source/reference/usage.rst
