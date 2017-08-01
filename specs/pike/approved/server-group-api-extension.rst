..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============
Server Group
============

https://blueprints.launchpad.net/mogan/+spec/server-group-api-extension

This spec proposes adding server group support to mogan. It's quite like
nova's server groups, but nova only support *host* based affinity or
anti-affinity, which is not fit for bare metals.


Problem description
===================

Currently, it is not possible to schedule some servers to the same or
different location or group like what nova server group does for virtual
machines.

Use Cases
---------

* As a tenant, I would like to schedule servers on the same group of bare metal
  nodes.

* As a tenant, I would like to schedule servers on the different group of bare
  metal nodes.

Proposed change
===============

We proposes to add a special aggregate metadata key *affinity_zone*. You may
define it as failure domains(e.g., by power circuit, rack, room, etc), or
whatever you want to make affinity and anity-affinity happen in such groups.
All bare metals nodes are in a same affinity zone by default.

* A new mogan.objects.ServerGroup object would be added to the object model.

* Add a set of APIs to allow tenants manage the server groups, including create,
  delete, show and list APIs.

* Allow users to specify server group when claiming servers.

* Add logic to scheduler to check the server group plicy and members affinity
  relationship to filter nodes.

Alternatives
------------

None

Data model impact
-----------------

The proposed change will be adding the following fields to the server group
object with their data type and default value for migrations.

+-----------------------+--------------+-----------------+
| Field Name            | Field Type   | Default Value   |
+=======================+==============+=================+
| id                    | Integer      | None            |
+-----------------------+--------------+-----------------+
| uuid                  | UUID         | None            |
+-----------------------+--------------+-----------------+
| user_id               | UUID         | None            |
+-----------------------+--------------+-----------------+
| project_id            | UUID         | None            |
+-----------------------+--------------+-----------------+
| name                  | String       | None            |
+-----------------------+--------------+-----------------+
| policies              | ListOfString | None            |
+-----------------------+--------------+-----------------+
| members               | ListOfString | None            |
+-----------------------+--------------+-----------------+

REST API impact
---------------

REST API will be changed as part of this change.

- To create a new server group, a user will::

    POST /v1/server_groups

  With a body containing the JSON description of the server group.

  JSON Schema::

    {
        "type": "object",
        "properties": {
            'name': {'type': 'string', 'minLength': 1, 'maxLength': 255},
            'policies': {
                'type': 'array',
                'items': [{
                    'type': 'string',
                    'enum': ['anti-affinity', 'affinity']}],
                'uniqueItems': True,
                'additionalItems': False,
            },
        },
        'required': ['name', 'policies'],
        'additionalProperties': False,
    }

- To list server groups, a user will::

    GET /v1/server_groups

- To show server group details, a user will::

    GET /v1/server_groups/{servergroup_id}

- To delete a server group, a user will::

    DELETE /v1/server_groups/{servergroup_id}

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

None

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

* Add server group object with the proposed fields.
* Add REST API to support server group management.
* Change scheduler to consider the specified server group.
* Allow users to specify server group when claiming servers.
* Change CLI to support server group management.
* Add UT and docs.

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about server groups will be added.

References
==========

None
