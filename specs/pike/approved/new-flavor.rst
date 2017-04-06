..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========
New Flavor
==========

https://blueprints.launchpad.net/mogan/+spec/new-flavor

Ironic added a "resource class" attribute to the primary node object represents
the hardware specs. And not like VMs, baremetal nodes are consumed atomically,
not piecemeal, so we would like to have the scheduler using a simplified search
for the baremetal nodes that only looks for node that has a particular resource
class.

Operators are the people who create the flavors and set the baremetal node
resource class attribute, so we need to add a 'resources' field to the flavor
which can include one or more resource classes to associate the baremetal node
resources. And a 'specs' field would be added for operators to fill the detail
hardware profile.


Problem description
===================

Currently, we just use the flavor name to match with the baremetal nodes,
which is not flexible, as we also want to support associating one flavor to
some different resource classes. And the detail hardware properties fields
make the flavor too complex, we need to keep the flavor simple.

Use Cases
---------

Operators and users wish to use a simple flavor/scheduler to search for
baremetal nodes.

Proposed change
===============

We proposes to add a resources filed which can associate one more more
resource class and a new specs field for hardware profile.

For properties like FPGA, we will leverage extra_specs to match the specific
class of machines, because capabilities can still be passed down to ironic.

The operator might define the flavors as such::

    - baremetal-gold
      resources: ironic-gold
      specs:
        cpus: 16
        memory_mb: 16384
      extra_specs:
        capabilities: boot_mode:bios

Alternatives
------------

None

Data model impact
-----------------

The proposed change will be adding the following fields to the flavor object
with their data type and default value for migrations.

+-----------------------+--------------+-----------------+
| Field Name            | Field Type   | Default Value   |
+=======================+==============+=================+
| uuid                  | UUID         | None            |
+-----------------------+--------------+-----------------+
| name                  | String       | None            |
+-----------------------+--------------+-----------------+
| resources             | ListOfString | None            |
+-----------------------+--------------+-----------------+
| specs                 | DictOfString | None            |
+-----------------------+--------------+-----------------+
| extra_specs           | DictOfString | None            |
+-----------------------+--------------+-----------------+
| is_public             | Bool         | True            |
+-----------------------+--------------+-----------------+
| disabled              | Bool         | False           |
+-----------------------+--------------+-----------------+


REST API impact
---------------

REST API will be changed as part of this change.

- To create a new flavor, a user will::

    POST /v1/flavors

  With a body containing the JSON description of the flavor.

  JSON Schema::

    {
        "type": "object",
        "properties": {
            'name': {'type': 'string', 'minLength': 1, 'maxLength': 255},
            'resources': {
                'type': 'array', 'minItems': 1,
                'items': {'type': 'string', 'minLength': 1, 'maxLength': 255},
            },
            'specs': {
                'type': 'object',
                'patternProperties': {
                    '^[a-zA-Z0-9-_:. ]{1,255}$': {
                        'type': 'string', 'maxLength': 255
                    }
                },
                'additionalProperties': False
            },
            'extra_specs': {
                'type': 'object',
                'patternProperties': {
                    '^[a-zA-Z0-9-_:. ]{1,255}$': {
                        'type': 'string', 'maxLength': 255
                    }
                },
                'additionalProperties': False
            },
            'is_public': {'type': 'boolean'},
        },
        'required': ['name', 'resources'],
        'additionalProperties': False,
    }

- To update a flavor, a user will::

    PATCH /v1/flavors/flavor_uuid

  We only allow to update below attributes::

    ['/name', '/is_public', '/disabled']

  Example of request BODY::

    {
        "op": "replace",
        "path": "/disabled",
        "value": true
    }

  Update other properties is not allowed, as it will make server properties
  not consistent with the real hardware. Users need to create a new flavor
  instead in this scenario. And when creating a server, we will check if
  the specified flavor is disabled.

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
  <niu-zglinux>

Work Items
----------

* Modify flavor object with the proposed fields.
* Change REST API to support new flavor properties.
* Change scheduler filters/weighters to match the new flavor.
* Change CLI to support flavor management.
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
