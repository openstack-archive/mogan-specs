..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Add lock API for changing instance to maintenance
=================================================

https://blueprints.launchpad.net/mogan/+spec/lock-instances

This spec proposes to provide the lock REST API, then operators do not
have to worry about instances could be terminated when maintaining.

Problem description
===================

In Mogan, we provided REST API to delete baremetal servers when do not need
any more. However, there is no limit of delete REST API usage. So user can
delete baremetal nodes in any circumstances. This change proposes to add
the lock REST API to disable terminate baremetal nodes if necessary.

Use Cases
---------

When maintaining, delete maintained baremetal nodes is unacceptable.
As an operator, if you want to maintain a baremetal servers, You could
lock baremetal node to forbid deleting operations.

After maintaining, you need to unlock baremetal node. Then you can do
other operations.

Proposed change
===============

The proposed change is relatively simple, we just need to add the lock
REST API, and change the state of instance to locked.

Alternatives
------------

None

Data model impact
-----------------

None

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

* JSON schema definition for the request body::

    {
        "type": "object",
        "properties": {
            "target": {
                "type": "boolean"
                }
            },
        "required": [
            "target"
        ]
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
    **Only Admin is allowed to request these API.**

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

* Add lock REST API to lock and unlock instances.
* Modify the delete REST API to valid an instances is locked.
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
