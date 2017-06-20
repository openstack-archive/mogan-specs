..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Track baremetal resources using Placement service
=================================================

https://blueprints.launchpad.net/mogan/+spec/track-resources-using-placement

This spec proposes using placement service for baremetal resources tracking.
The placement service is intended to enable more effective accounting of
resources and better scheduling of various entities in the cloud.


Problem description
===================

We'd like to be able to schedule not only based on baremetal node resources,
but also other resources in the cloud like shared storage. So we need to move
to this more generic modeling of quantitative resources.

Use Cases
---------

Cloud operators want to be able to add new classes of resources to the system
with a generic modle and do so without any downtime caused by database schema
migrations.


Proposed change
===============

We proposes to create a resource provider for each baremetal node like nova,
set provider's inventory to a single record of custom resource class like
CUSTOM_BAREMETAL_GOLD or whatever the ironic node's resource class attribute
is. All non-standard resource classes must begin with `CUSTOM_`, so only nodes
from ironic with resource classes begin with such prefix will be updated to
placement.

When the scheduler constructing the request to placement, it will extract
resources classes from flavor resources attribute to filter scheduling
candidates by the custom resource classes.

Alternatives
------------

None

Data model impact
-----------------

This will remove ComputeNodes and ComputePorts objects.

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

None

Other deployer impact
---------------------

This will break all mogan deployments as we migrate resources tracking to
another service, but as we don't have any release yet, so we suppose there's
no mogan deployment.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
    liusheng

Other contributors:
    zhenguo

Work Items
----------

* Copy placement client from nova.
* Update resource tracker to report resources to placement.
* Adjust scheuler filters for such refactoring.
* remove the compute_nodes and compute_ports resources objects.
* Add UT and docs.

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about new flavor will be added.

References
==========

None
