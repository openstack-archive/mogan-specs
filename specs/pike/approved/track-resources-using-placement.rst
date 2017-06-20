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
to this more generic modeling of quantitative and qualitative resources.

Use Cases
---------

Cloud operators want to be able to add new classes of resources to the system
with a generic model and do so without any downtime caused by database schema
migrations.


Proposed change
===============

* We will define new resource classes for each node accodring to what is in
  the ironic node's `resource_class` attribute.

* Generate the resource provider for each baremetal node to track the
  resources, set provider's inventory to a single record of custom resource
  class. The resource tracking model is as the following::

    resource classes:               CUSTOM_GOLD                    CUSTOM_SILVER
                                    /         \                          |
    resource providers: baremetal_node1     baremetal_node2       baremetal_node2
                               |                   |                     |
    inventories:       {'CUSTOM_GOLD': 1}   {'CUSTOM_GOLD': 1}  {'CUSTOM_SILVER': 1}

* As we consume baremetal nodes atomically not piecemeal, so the inventory data
  for such resource provider should be as::

    Inventory: {
        "total": 1,
        "reserved": 0,
        "min_unit": 1,
        "max_unit": 1,
        "step_size": 1,
        "allocation_ratio": 1.0
    }

* For **qualitative** aspects of the baremetal resources, the placement traits
  is targeted to support this, we can use custom traits to define what the
  resource provider's characteristics is.


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

* Add placement client and config options.
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
