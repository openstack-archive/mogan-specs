..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================
Unmanage Existing BMs
=====================

https://blueprints.launchpad.net/mogan/+spec/unmanage-existing-bms

This spec is intended to allow mogan to unmanage nodes that have been managed
into Mogan by operators or created by users.

Problem description
===================

At present the mogan API can allow to manage nodes in active state which
migrated to ironic by operators. This function will help operator to migrate
running baremtal nodes from one "system" to another "system". But we also need
the 'unmanage' function to help operator to release those nodes from Mogan
under some cases, like be out of control from cloud system.

Use Cases
---------

* As an operator of hybrid infrastructures, after migrating running nodes
  to OpenStack cloud, I want to release those nodes from OpenStack cloud.

* As an operator of multiple distinct OpenStack infrastructures, after
  migrating running nodes from one OpenStack to another, I want to release
  those nodes from this OpenStack.


Proposed change
===============

*  Introduce a new API for unmanaging the managed server. Mogan will delete
   the server object, including port/volume server association information.
   But it will NOT delete the server from driver and keep the ports or volumes
   in Neutron or Cinder since the baremetal machine is still using by users.


Alternatives
------------

None

Data model impact
-----------------

None


REST API impact
---------------

#. Add a new API to unmanage a server from Mogan, it is an admin only API::

    DELETE v1/manageable_servers/{server_uuid}


Security impact
---------------

None

Notifications impact
--------------------

Notification about the unmanage action will be added.

Other end user impact
---------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

None

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  wanghao <sxmatch1986@gmail.com>

Work Items
----------

* Add new APIs.
* Add a new taskflow for server unmanaging.
* Add CLI support.

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about unmanaging servers will be added, including the API doc.

References
==========

None
