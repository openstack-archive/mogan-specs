..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Manage Existing BMs
===================

https://blueprints.launchpad.net/mogan/+spec/manage-existing-bms

This spec is intended to allow mogan to manage nodes that migrated to ironic
by operators.

Problem description
===================

At present the mogan API can only allow create new servers from nodes
in available state, there's no way to manage nodes in active which migrated
to ironic by operators.

For an operator of multiple infrastructures, it's reasonable to permit an
operator to migrate running baremtal nodes from one "system" to another
"system".

Use Cases
---------

* As an operator of hybrid infrastructures, I want to migrate running nodes
  to OpenStack cloud.

* As an operator of multiple distinct OpenStack infrastructures, I want to
  migrate running nodes from one OpenStack to another.


Proposed change
===============

*  Introduce a new admin only API, which supports to query nodes that could
   be managed by mogan. This API will pass down the request to drivers, which
   needs to add a new driver interface `get_manageable_nodes`, there will be
   driver specified criterias of which nodes are manageable. For ironic, it
   should be nodes in active state but without instance_uuid associated, and
   the resource class field should be well set.

*  Introduce a new API for managing running baremtal nodes listed by the above
   API. This needs to add a new workflow which will skip schduling comparing
   with server create workflow.

*  We will collect the image, network information from the adoptable nodes, and
   will check if the resource existing in glance and neutron. For images, it
   will be None if we can't find it, but for neutron port, it should be a must
   when determine wheter the node can be managed.


Alternatives
------------

None

Data model impact
-----------------

None


REST API impact
---------------

#. Add a new URI named 'manageable_servers' with ManageableServersController,
   of course it is a admin only API::

    POST v1/manageable_servers

#. The management API schema is like this::

    manage_server = {
     "type": "object",
     "properties": {
         'name': {'type': 'string', 'minLength': 1, 'maxLength': 255},
         'description': {'type': 'string', 'minLength': 1, 'maxLength': 255},
         'availability_zone': {'type': 'string', 'minLength': 1,
                               'maxLength': 255},
         'node_uuid': {'type': 'string', 'format': 'uuid'},
         'flavor_uuid': {'type': 'string', 'format': 'uuid'},
         'networks': {
             'type': 'array', 'minItems': 1,
             'items': {
                 'type': 'object',
                 'properties': {
                     'port_type': {'type': 'string', 'minLength': 1,
                                   'maxLength': 255},
                     'port_id': {'type': 'string', 'format': 'uuid'},
                 },
                 'required': ['port_id'],
                 'additionalProperties': False,
             },
         },
     },
     'required': ['name', 'node_uuid', 'flavor_uuid', 'networks'],
     'additionalProperties': False,
    }

#. Add a new API that will list manageable servers which will include all
   needed informations when calling manage API. It may look like this::

   GET v1/manageable_servers

   {
    "manageable_servers": [
        {
            "name": "test_server",
            "ports": [
                {
                    "uuid": "6d85703a-565d-469a-96ce-30b6de53079d",
                    "vif_port_id": "12345678-1234-1234-1234-123456789012",
                    "href": "http://127.0.0.1:6385/v1/nodes/6d85703a-565d-469a-96ce-30b6de53079d/ports",
                    "rel": "self"
                },
                {
                    "uuid": "6d85703a-565d-469a-96ce-30b6de53079d",
                    "vif_port_id": "12345678-1234-1234-1234-123456789013",
                    "href": "http://127.0.0.1:6385/nodes/6d85703a-565d-469a-96ce-30b6de53079d/ports",
                    "rel": "bookmark"
                }
                     ],
            "power_state": "power on",
            "provision_state": "active",
            "created_at": "2016-10-17T04:12:44+00:00",
            "uuid": "f978ef48-d4af-4dad-beec-e6174309bc71",
            "properties": {},
            "instance_info": {},
            "resource_class": 'gold',
        }
    ]
   }


Security impact
---------------

None

Notifications impact
--------------------

Notification about the adopt action will be added.

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

Other drivers will raise NotImplement exception if not add such interface.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  wanghao <sxmatch1986@gmail.com>

Work Items
----------

* Add new APIs.
* Add a new taskflow for server managing.

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about manage servers will be added, including the preparation work
for operator.

References
==========

None
