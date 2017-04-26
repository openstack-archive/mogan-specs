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


Proposed change
===============

Before the process in Mogan, user need to make the node state to 'active'
by invoking Ironic adopt API.

#. Invoke Ironic node create API.
#. Creating ports in Ironic.
#. Invoke the Ironic node update API to update some instance_info in Ironic.
#. Invoke the node set provision state manage API to change node state to
   'manageable'.
#. Invoke the node set provision state adopt API to change node state to
   active.

There is more specifical illustration in Ironic doc: [1].


Then Mogan will do:

#. Introduce two new APIs, support to manage the existing nodes in Ironic,
   and also can unmanage them.
#. Reuse the most process of creation servers but do not need to make a
   scheduler since we will update the server.node_uuid according the request
   argument.
#. Create some vifs, plug those to node and save server nics.
#. When unmanaging servers, Mogan will unplug the vifs from Ironic, but do not
   destroy the node from Ironic.
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

#. Add a new API that will list manageable nodes which are Active in Ironic but
   without instance uuid in Mogan. This API will include ironic node properties
   information in response body.

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
