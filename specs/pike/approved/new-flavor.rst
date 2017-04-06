..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========
New Flavor
==========

https://blueprints.launchpad.net/mogan/+spec/new-flavor

This spec proposes to add a flexible flavor for baremetal instances, besides
CPU, MEMORY_MB, we should also take into account of hard drives, nics, and
others like FPGA.

And the existing baremetal clouds are all able to select nodes based on not
only properties like virtual machines. You can choose cpu models, hard drives
types, nics types and so on [1]_, [2]_.


Problem description
===================

Currently, we just use a simple flavor to match node types. Which can't pass
all users' requests, and it's hard to management if we have indivisible bare
metal machines. Also, for Valence node, we need to be able to get detail
properties from the flavor.

Use Cases
---------

Users should be able to see what sort of CPU/RAM/DISK/NIC resources are
available on the baremetal flavors, and DISKs and NICs are also used for
deploy time raid and nics bond. Also, users can specify different type of
nics to different tenant networks and scheduler also considers the networks
amount and types specified.

NFV users want to select a flavor that targets their baremetal instance to
a machine with an FPGA that has particular software loaded.

Proposed change
===============

We proposes to add more properties based on nova flavor and change the property
type to dict to include more things like model, type instead of only size.

For properties like FPGA, we will leverage extra_specs to match the specific
class of machines.

REST API will be changed to POST/PUT the new properties, and new schema will be
added.

Some Examples:
  Example 1 (memory)::

    {
      "size_mb": "8192",
      "type": "DDR3"
    }

  Example 2 (cpus)::

    {
      "cores": "16",
      "model": "Dual Intel Xeon E5-2650 2.00GHz"
    }

  Example 3 (disks)::

    [
      {
        "size_gb": "600",
        "type": "SSD"
      },
      {
        "size_gb": "1024"
        "type": "HDD"
      }
    ]

  Example 4 (nics)::

    [
      {
        "speed": "10 Gbps",
        "type": "Ethernet"
      },
      {
        "speed": "40 Gbps"
        "type": "InfiniBand"
      }
    ]

Before reaching to scheduler, we need to add some checks for flavor properties
and the request parameters.

  * check whether nics type and amount satisfy the specified tenant networks.
  * check whether disks type and amount satisfy the specified partition and
    RAID configuration.

Scheduler need to be changed to match with the new flavor.

  * cpu_filter: model and cores
  * ram_filter: type and size
  * disk_filter: type, size, and amount
  * port_filter: type, speed, and amount
  * capabilities_filter: extra specs

Alternatives
------------

Use flavor extra specs to solve this.

Data model impact
-----------------

The proposed change will be add the following fields to the flavor object
with their data type and default value for migrations.

+-----------------------+--------------+-----------------+
| Field Name            | Field Type   | Default Value   |
+=======================+==============+=================+
| uuid                  | UUID         | None            |
+-----------------------+--------------+-----------------+
| name                  | String       | None            |
+-----------------------+--------------+-----------------+
| cpus                  | DictOfString | None            |
+-----------------------+--------------+-----------------+
| memory                | DictOfString | None            |
+-----------------------+--------------+-----------------+
| disks                 | ListOfDict   | None            |
+-----------------------+--------------+-----------------+
| nics                  | ListOfDict   | None            |
+-----------------------+--------------+-----------------+
| extra_specs           | DictOfString | None            |
+-----------------------+--------------+-----------------+
| is_public             | Bool         | True            |
+-----------------------+--------------+-----------------+
| disabled              | Bool         | False           |
+-----------------------+--------------+-----------------+
| projects              | ListOfString | None            |
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
            'cpus': {
                'type': 'object',
                'properties': {
                    'model': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                    'cores': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                }
                'required': ['model', 'cores'],
                'additionalProperties': False,
            }
            'memory': {
                'type': 'object',
                'properties': {
                    'size_mb': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                    'type': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                }
                'required': ['size_mb', 'type'],
                'additionalProperties': False,
            }
            'disks': {
                'type': 'array',
                'items': {
                    'type': 'object',
                    'properties': {
                        'size_gb': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                        'type': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                    },
                    'required': ['size_gb', 'type'],
                    'additionalProperties': False,
                },
            },
            'nics': {
                'type': 'array', 'minItems': 1,
                'items': {
                    'type': 'object',
                    'properties': {
                        'speed': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                        'type': {'type': 'string', 'minLength': 1, 'maxLength': 255},
                    },
                    'required': ['speed', 'type'],
                    'additionalProperties': False,
                },
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
            'projects': {
                'type': 'array',
                'items': {'type': 'string', 'minLength': 1, 'maxLength': 255},
            },
        },
        # disks is not a mandatory property, we need to support non disk machine
        'required': ['name', 'cpus', 'memory', 'nics'],
        'additionalProperties': False,
    }

  Example of request BODY::

    {
        "name": large,
        "cpus": {
            "cores": "16",
            "model": "Dual Intel Xeon E5-2650 2.00GHz"
        },
        "memory": {
            "size_mb": "8192",
            "type": "DDR3"
        },
        "disks": [
            {
                "size_gb": "600",
                "type": "SSD"
            },
            {
                "size_gb": "1024"
                "type": "HDD"
            }
        ],
        "nics": [
            {
                "speed": "10 Gbps",
                "type": "Ethernet"
            },
            {
                "speed": "40 Gbps"
                "type": "InfiniBand"
            }
        ],
        "extra_specs": {
            "FPGA": "true"
        },
        "is_public": false,
        "projects": [bf942f63-c284-4eb8-925b-c2fa1a89ed33]
    }


- To update a flavor, a user will::

    PATCH /v1/flavors/flavor_uuid

  We only allow to update below attributes::

    ['name', '/is_public', '/disabled', '/projects']

  Example of request BODY::

    {
        "op": "replace",
        "path": "/disabled",
        "value": true
    }

  Update other properties is not allowed, as it will make instance properties
  not consistent with the real hardware. Users need to create a new flavor
  instead in this scenario. And when creating a instance, we will check if
  the specified flavor is disabled.


- To list flavors::

    GET /v1/flavors
    GET /v1/flavors/detail


- To delete a flavor::

    DELETE /v1/flavors/flavor_uuid


- To show a flavor::

    GET /v1/flavors/flavor_uuid


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

* Change instance type DB model.
* Add new flavor object(rename from instance type).
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

.. [1] http://www.softlayer.com/bare-metal-servers
.. [2] https://www.rackspace.com/cloud/servers/onmetal

* https://wiki.openstack.org/wiki/Valence
