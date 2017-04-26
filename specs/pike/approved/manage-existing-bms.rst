..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Manage Existing BMs
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
   managed by Mogan. Drivers need to implement an interface
   'list_adoptable_nodes'， that will return the list of adoptable nodes.
   Of course, there are some restrications to find which node could
   be managed. In Ironic, the node should be without instance_uuid, be in
   active state and have resource class set.
   Basically, it will depend on driver's implementation.

#. Introduce a new API, support to manage the existing nodes in Ironic.
   This API will reuse the most process of creation servers but do not need to
   make a scheduler since we will update the server.node_uuid according the
   request argument.

#. For managing the node's network, operators should create ports in Neutron
   first, and then specify those port_ids to manage this node, Mogan will check
   those ports existing and set this information to server networks.

#. Mogan will get the image from instance_info and resource class in
   manageable node and will check if the flavor in Mogan matches the resource
   class in node and also will check if the image is existing in Glance, if not
   there, Mogan will set the service's image field empty.

#. Update the status and power state of server, finish the adopt process.



Alternatives
------------

None

Data model impact
-----------------

None


REST API impact
---------------

#. Add a new custom action named 'manage' in ServerController, of course
   it is a admin only API::

    _custom_actions = {
         'manage': ['POST']
    }

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

#. Add a new API that will list manageable nodes which are Active in Ironic but
   without instance uuid. This API will include ironic node properties
   information in response body. It may look like this::

   {
    "manageable_nodes": [
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

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about adopt/manage servers will be added, including the preparation work
for operator.

References
==========

[1]: https://docs.openstack.org/developer/ironic/deploy/adoption.html
