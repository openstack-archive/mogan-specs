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
   driver specified criteria of which nodes are manageable. For ironic, it
   should be nodes in active state but without instance_uuid associated, and
   the resource class field should be well set.

*  Introduce a new API for managing running baremtal nodes listed by the above
   API. This needs to add a new workflow which will skip schduling comparing
   with server create workflow.

*  We will collect the image, network information from the manageable nodes,
   and will check if the resource existing in glance and neutron.
   For images and neutron ports, it will be None if we can't find them.


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
         'metadata': {'type': 'object',
                      'patternProperties': {
                          '^[a-zA-Z0-9-_:. ]{1,255}$': {
                              'type': 'string', 'maxLength': 255
                              }
                         },
                      'additionalProperties': False
         }
         'node_uuid': {'type': 'string', 'format': 'uuid'},
     },
     'required': ['name', 'node_uuid'],
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
                     "address": "a4:dc:be:0e:82:a5",
                     "uuid": "1ec01153-685a-49b5-a6d3-45a4e7dddf53",
                     "neutron_port_id": "a9b94592-1d8e-46bb-836b-c7ba935b0136"
                 },
                 {
                     "address": "a4:dc:be:0e:82:a6",
                     "uuid": "1ec01153-685a-49b5-a6d3-45a4e7dddf54",
                     "neutron_port_id": "a9b94592-1d8e-46bb-836b-c7ba935b0137"
                 }
                      ],
             "portgroups": [],
             "power_state": "power on",
             "provision_state": "active",
             "uuid": "f978ef48-d4af-4dad-beec-e6174309bc71",
             "resource_class": 'gold',
             "image_source": "03239419-e588-42b6-a70f-94f23ed0c9e2"
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
