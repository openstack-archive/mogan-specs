..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Manage Existing BMS
===================

https://blueprints.launchpad.net/mogan/+spec/manage-existing-bms

Current Mogan can provision bare metal instances now by using creation API,
that will install the operation system in bare metal node. But we also want
the function that can adopt the existing bare metal node that has been used by
user. This feature will help user to manage their bms through the Mogan.

Problem description
===================

Mogan currently can only provision the 'new' servers, but users may
want to adopt the existing bms to Mogan, and then manage them by using Mogan.

Use Cases
---------

* Some nodes they have been used by user, and user want to mange them with
other new nodes through Mogan. 


Proposed change
===============

Before the process in Mogan, user need to make the node state to 'active'
by invoking Ironic API.

#. Invoke the Ironic creating node API with specific 'fake' driver to let the
   Ironic know this node. And pass the node_uuid to Mogan management API
   according the response of Ironic.
#. Creating ports in Ironic.
#. Invoke the Ironic setting provision state API to change the state
   from 'available' to 'active' but not redeploy the bm node.
#. Invoke the Ironic API to update the driver info of this node to let Mogan
   can manage the node states after managing done.

Then Mogan will do:

#. Introduce two new APIs, support to manage the existing bms, and also can
   unmanage them.
#. Reuse the most process of creation instances but do not need to make a
   scheduler since we will update the instance.node_uuid according the request
   argument.
#. Create some vifs, plug those to node and save instance nics.
#. When unmanaging nodes, Mogan will destroy the network from Neutron and
   unplug the vifs from Ironic, but do not destroy the node from Ironic.
   After this done, user need to delete the node in Ironic by calling node
   deleting API in Ironic.



Alternatives
------------

None

Data model impact
-----------------

None


REST API impact
---------------

#. Add a new custom action named 'manage' in InstanceController::

    _custom_actions = {
         'management': ['POST']
    }

#. The management API schema is like this::

    management_instance = {
     "type": "object",
     "properties": {
         'name': {'type': 'string', 'minLength': 1, 'maxLength': 255},
         'description': {'type': 'string', 'minLength': 1, 'maxLength': 255},
         'availability_zone': {'type': 'string', 'minLength': 1,
                               'maxLength': 255},
         'node_uuid': {'type': 'string', 'format': 'uuid'},
         'image_uuid': {'type': 'string', 'format': 'uuid'}
         'instance_type_uuid': {'type': 'string', 'format': 'uuid'},
         'networks': {
             'type': 'array', 'minItems': 1,
             'items': {
                 'type': 'object',
                 'properties': {
                     'net_id': {'type': 'string', 'format': 'uuid'},
                     'port_type': {'type': 'string', 'minLength': 1,
                                   'maxLength': 255},
                 },
                 'required': ['net_id'],
                 'additionalProperties': False,
             },
         },
     },
     'required': ['name', 'image_uuid', 'instance_type_uuid', 'networks'],
     'additionalProperties': False,
    }

#. Expose the InstanceUnmanageController in InstanceController::

   unmanage = InstanceUnmanageController()

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
  wanghao <sxmatch1986@gmail.com>

Work Items
----------

* Add new APIs.
* Add a new taskflow for node managing.
* Add new process for node unmanaging.

Dependencies
============

None

Testing
=======

Unit Testing will be added.

Documentation Impact
====================

Docs about adopt/manage bms will be added.

References
==========

None
