..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Add lock API for changing instance to lock/unlock
=================================================

https://blueprints.launchpad.net/mogan/+spec/lock-instances

This spec proposes to provide the lock REST API, then operators do not
have to worry about instances could be terminated by mistaken.

Problem description
===================

In Mogan, we provided REST API to delete baremetal servers when do not need
any more. However, there is no limit of delete REST API usage. So user can
delete baremetal server in any circumstances. This change proposes to add
the lock REST API to disable terminate baremetal server if necessary.

Use Cases
---------

when booting a baremetal server, a delete request from other users by
mistaken is unacceptable. To avoid this, lock your server is better.

For another use case, when maintaining, delete maintained baremetal
servers is also unacceptable. As an operator, if you want to maintain a
baremetal servers, You could lock baremetal servers to forbid deleting
operations. After maintaining, you need to unlock baremetal servers.
Then you can do other operations.

Proposed change
===============

* Modify the data model to record the lock state of instances.
* Add the lock REST API to lock/unlock instance.

Alternatives
------------

Maybe leveraging the instance state instead of adding a new locked field
is an alternative. While, if we want to lock an buiding instance, it`s
difficult to deal with the lock state and build state with only one state
column.

Data model impact
-----------------

The `mogan.objects.instance.Instance` object would have new `locked` and
`locked_by` field of type `mogan.objects.fields.ListOfStrings` that would
be populated on-demand(i.e. not eager-loaded).

A locked shall be defined as a tinyint no longer than 1 bytes in length,
and the locked_by shall be defined as an enum with owner and admin as its
valid values.

For the database schema, the following table changes would suffice ::

    ALTER TABLE `instances`
    ADD COLUMN `locked`  tinyint(1) NULL DEFAULT NULL,
    ADD COLUMN `locked_by`  enum('owner','admin') NULL DEFAULT NULL;


REST API impact
---------------


* Request method:
    * PUT

* URL:
    * /instances/{instance_uuid}/states/lock

*Lock an instance*

* Parameters for request ::

    {
        "target": true
    }

*Unlock an instance*

* Parameters for request ::

    {
        "target": false
    }

* Normal HTTP response code:
    * `202 ACCEPTED`

* Expected error http response codes
    * `400 BadRequest`
      The request params were invalied

    * `404 NotFound`
      The instance requested to be lock was not found

    * `403 Forbidden`
      The user has no access to request this API

    * `409 Conflict`
      The instance requested to be lock or unlock was not in valid status

* Policy changes:
    **Only Admin and owner is allowed to request these API.**
    * If `Admin` locked an instance, only `Admin` can unlock it.
    * If `Owner` locked an instance, both `Owner` and `Admin` can unlock it.
    * If an instance has been locked, **UNLOCK** was only allowed for `Owner`
      and `Admin`. And, other operations should be denied for non-admin.

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

As part of this effort we will also need to add the support to
python-moganclient.

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
  zhangjialong <zhangjl@awcloud.com>

Other contributors:
  jolie <guoshan@awcloud.com>

Work Items
----------

* Modify the database model of instances.
* Add lock REST API to lock and unlock instances.
* Valid an instance is locked before execute other operations.
* Support the new lock REST API in python-moganclient.


Dependencies
============

None.

Testing
=======

* Unit tests will be added to Mogan for testing the new
  REST API.

Documentation Impact
====================

The in-tree API reference will be updated for the mogan REST API
documentation.

References
==========

None
