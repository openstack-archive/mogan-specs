..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Manage Existing BMS
===================

https://blueprints.launchpad.net/mogan/+spec/manage-existing-bms

Currently Mogan can provision bare metal server via ironic by using
creation API, that will install the operation system in bare metal node.
But we also want the function that can adopt the server after it has been
adopted by ironic with the existing bare metal node that has been used by
user. This feature will help user to manage their servers through the Mogan.

Problem description
===================

Mogan currently can only provision the 'new' servers, but users may
want to adopt the existing servers to Mogan, and then manage them by using
Mogan.

Use Cases
---------

* Some nodes have been used by user, and user want to mange them with
other new nodes through Mogan.

For different drivers, before managing those nodes via Mogan API,
there may be some works needed to be done. For example, user need to
make the node state to 'active' by invoking Ironic adopt API.
There is more specifical illustration in Ironic doc: [1].

Proposed change
===============

Mogan will do:

#  Introduce a new admin only API, support to query which nodes could be
   manageable by Mogan. Drivers need to implement an interface
   'list_managable_nodes'， that will return the list of manageable nodes.
   Of course, there are some restrications to find which node could
   be managed, like without instance_uuid in Mogan or must have node_type
   property in Ironic. It will depend on driver's implementation.

#. Introduce a new API, support to manage the existing nodes in Ironic.
   This API will reuse the most process of creation servers but do not need to
   make a scheduler since we will update the server.node_uuid according the
   request argument.

#. For managing the node's network, there are two cases which should be
   considered: 1. the node has already connected to some neutron ports,
   that user just need to get those ports from Ironic
   port.internal_info/tenant_port_id or other backends and specify the port
   ids to adopt this node. Mogan will check if those ports are existing in
   Neutron.
   2. the node didn't have any ports in neutron, then user need to decide
   which network will be used and pass them to mogan for adopting this node,
   Mogan will create new ports in those networks and invoke the port bond
   operations to make the switch port configured.

#. Mogan will also check the image and flavor that user specify in request
   to see if they're match with the information that get from the drivers.

#. Update the status and power state of server, finish the adopt process.



Alternatives
------------

None

Data model impact
-----------------

None


REST API impact
---------------

#. Add a new custom action named 'manage' in InstanceController, of course
   it is a admin only API::

    _custom_actions = {
         'management': ['POST']
    }

#. The management API schema is like this::

    management_server = {
     "type": "object",
     "properties": {
         'name': {'type': 'string', 'minLength': 1, 'maxLength': 255},
         'description': {'type': 'string', 'minLength': 1, 'maxLength': 255},
         'availability_zone': {'type': 'string', 'minLength': 1,
                               'maxLength': 255},
         'node_uuid': {'type': 'string', 'format': 'uuid'},
         'image_uuid': {'type': 'string', 'format': 'uuid'}
         'flavor_uuid': {'type': 'string', 'format': 'uuid'},
         'networks': {
             'type': 'array', 'minItems': 1,
             'items': {
                 'type': 'object',
                 'properties': {
                     'net_id': {'type': 'string', 'format': 'uuid'},
                     'port_type': {'type': 'string', 'minLength': 1,
                                   'maxLength': 255},
                     'port_id': {'type': 'string', 'format': 'uuid'},
                 },
                 'oneOf': [
                     {'required': ['net_id']},
                     {'required': ['port_id']}
                 ],
                 'additionalProperties': False,
             },
         },
     },
     'required': ['name', 'image_uuid', 'flavor_uuid', 'networks'],
     'additionalProperties': False,
    }

#. Add a new API that will list manageable nodes which are Active in Ironic but
   without instance uuid in Mogan. This API will include ironic node properties
   information in response body. It may look like this::

   {
    "manageable_nodes": [
        {
            "availability_zone": null,
            "name": "test_server",
            "ports": [
                {
                    "href": "http://127.0.0.1:6385/v1/nodes/6d85703a-565d-469a-96ce-30b6de53079d/ports",
                    "rel": "self"
                },
                {
                    "href": "http://127.0.0.1:6385/nodes/6d85703a-565d-469a-96ce-30b6de53079d/ports",
                    "rel": "bookmark"
                }
                     ],
            "power_state": "power on",
            "status": "building",
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

Other drivers except Ironic in Mogan will need the implementation to support
it.


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
* Add new process for server unmanaging.

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about adopt/manage servers will be added.

References
==========

[1]: https://docs.openstack.org/developer/ironic/deploy/adoption.html
